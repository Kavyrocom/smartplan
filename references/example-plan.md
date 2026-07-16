# Plan de déploiement : Pipeline de notification API → Slack

**Objectif :** Mettre en place un pipeline qui écoute les webhooks d'une API, stocke les événements en base, et notifie un canal Slack en temps réel.
**Périmètre :** Configuration de l'endpoint, création de la table, mise en place du webhook entrant, notification Slack. Exclut l'analyse des événements et le dashboarding.
**Propriétaire :** Agent orchestrateur
**Date :** 2026-07-16
**Statut :** Brouillon

---

## 1. Contexte et prérequis

### État actuel
Aucun pipeline de notification n'existe. Les événements de l'API ne sont ni stockés ni notifiés.

### Prérequis
- [ ] API accessible (vérifier : `curl -s https://api.example.com/health` retourne 200)
- [ ] Base de données accessible (vérifier : `sqlite3 /path/to/db.sqlite ".tables"` ne lève pas d'erreur)
- [ ] Token webhook Slack récupéré (vérifier : présent dans les variables d'environnement)
- [ ] Endpoint récepteur déployé (vérifier : `curl -s https://receiver.example.com/ping` retourne 200)

### Hypothèses
- L'API supporte les webhooks sortants (à vérifier étape 1) — risquée si non documenté.
- Le canal Slack de destination est #api-alerts (non confirmé par l'utilisateur — à valider au GO-1).
- Le fuseau horaire des timestamps est UTC (non spécifié — à confirmer si l'équipe est hors UTC).

---

## 2. Étapes d'exécution

### Étape 1 : Vérifier les capacités webhook de l'API
**Responsable :** Agent
**Action :** Lister les événements webhook disponibles dans la documentation API
**Livrable :** Liste des événments disponibles (order.created, payment.received, etc.)
**Vérification :** `curl -s https://api.example.com/webhooks/events | jq '.[].name'` retourne ≥ 1 événement
**Risque :** API ne supporte pas les webhooks → fallback vers polling

### Étape 2 : Créer la table de stockage `[MUTATION]`
**Responsable :** Agent
**Action :** `CREATE TABLE api_events (id INTEGER PRIMARY KEY, event_type TEXT, payload TEXT, received_at DATETIME DEFAULT CURRENT_TIMESTAMP, notified INTEGER DEFAULT 0)`
**Livrable :** Table créée
**Vérification :** `sqlite3 /path/to/db.sqlite ".tables" | grep api_events` retourne la table
**Risque :** Migration lock → vérifier qu'aucune autre écriture tourne

### Étape 3 : Configurer le webhook API → endpoint récepteur `[MUTATION]`
**Responsable :** Agent
**Action :** Enregistrer l'endpoint récepteur auprès de l'API pour les événements sélectionnés
**Livrable :** Webhook ID côté API
**Vérification :** `curl -s https://api.example.com/webhooks | jq '.[].id'` contient l'ID
**Risque :** Endpoint injoignable depuis l'API → vérifier DNS + firewall + HTTPS

### Étape 4 : Configurer la notification Slack `[MUTATION]`
**Responsable :** Agent
**Action :** Créer un webhook entrant Slack et le brancher sur le récepteur pour qu'un événement stocké déclenche un message
**Livrable :** Règle de notification active
**Vérification :** `curl -s -X POST https://hooks.slack.com/services/T... -H 'Content-Type: application/json' -d '{"text":"test"}'` retourne 200 + message visible dans Slack
**Risque :** Token invalide → vérifier les permissions Slack

### Étape 5 : Test end-to-end (dry-run) `[MUTATION]`
**Responsable :** Agent
**Action :** Déclencher un événement test via l'API, vérifier stockage en base + notification Slack
**Livrable :** Événement test en base + message Slack reçu
**Vérification :** `sqlite3 /path/to/db.sqlite "SELECT COUNT(*) FROM api_events WHERE event_type LIKE '%test%'"` retourne ≥ 1 ET message Slack reçu dans les 60s
**Risque :** Webhook silent fail → vérifier les logs du récepteur

### Étape 6 : Activer en production `[MUTATION]`
**Responsable :** Agent
**Action :** Passer le webhook API en mode live (production)
**Livrable :** Webhook en production
**Vérification :** `curl -s https://api.example.com/webhooks/<id> | jq '.status'` retourne `active`
**Risque :** Flux d'événements réel trop volumineux → rate-limit + queue

---

## 3. GO gates

| Gate | Après étape | Condition | Qui valide |
|------|-------------|-----------|------------|
| GO-1 | Étape 1 | Webhooks API disponibles + plan confirmé | Humain |
| GO-2 | Étape 5 | Dry-run end-to-end OK + message Slack reçu | Humain |
| GO-3 | Étape 6 | Readback production OK | Humain |

---

## 4. Rollback

### Niveau 1 : Annulation immédiate
- Désactiver le webhook API : `curl -X DELETE https://api.example.com/webhooks/<id>`
- Désactiver la notification Slack : supprimer la règle dans le récepteur

### Niveau 2 : Restauration
- Backup de la base : `cp /path/to/db.sqlite /backups/db-$(date +%Y%m%d).sqlite` avant l'étape 2
- Backup de la config webhook : export JSON avant l'étape 3 (`curl -s https://api.example.com/webhooks > /backups/webhooks-$(date +%s).json`)

### Niveau 3 : Rollback complet
- Supprimer le webhook API
- Supprimer la règle Slack
- `DROP TABLE api_events`
- Restaurer la base depuis le backup

### Déclencheurs de rollback
- Webhook API retourne erreur 5xx sur 3 événements consécutifs
- Notification Slack en spam (>20 messages/min) indiquant une boucle
- Données corrompues en base (payload vide ou malformé sur >10% des événements)
- Latence de notification > 5 minutes

---

## 5. Vérification finale

### Critères de fin
- [ ] Webhook API actif en production
- [ ] Événements stockés en base avec payload complet
- [ ] Notification Slack reçue dans les 60 secondes après l'événement
- [ ] Test end-to-end réussi avec un événement réel

### Preuve de succès
- `curl -s https://api.example.com/webhooks/<id> | jq '.status'` retourne `active`
- `sqlite3 /path/to/db.sqlite "SELECT COUNT(*) FROM api_events"` ≥ 1 après test
- Message Slack avec timestamp correspondant à l'événement test

---

## 6. Risques et points d'attention

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| API ne supporte pas les webhooks | Faible | Élevé | Vérifier étape 1 avant toute création |
| Flux d'événements trop volumineux | Moyen | Moyen | Rate-limit + queue côté récepteur |
| Token Slack expiré | Faible | Faible | Alerting sur échec de notification |
| Endpoint récepteur en downtime | Moyen | Élevé | Health check + retry 3x avec backoff |

---

## 8. Notes

- Le récepteur doit être idempotent : un même événement reçu 2x ne doit pas créer 2 entrées ni 2 notifications Slack.
- Le payload est stocké en TEXT brut pour permettre un reprocessing ultérieur sans dépendance API.
- La colonne `notified` permet de distinguer les événements notifiés de ceux en attente (utile pour un retry batch).