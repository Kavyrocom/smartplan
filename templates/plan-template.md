# Plan de déploiement : [Nom du process]

**Objectif :** [Une phrase — ce que ce plan accomplit]
**Périmètre :** [Ce qui est inclus / ce qui est explicitement exclu]
**Propriétaire :** [Agent ou humain]
**Date :** [Date]
**Statut :** Brouillon

---

## 1. Contexte et prérequis

### État actuel
[Ce qui existe aujourd'hui]

### Prérequis
- [ ] [Système X accessible via Y]
- [ ] [Accès Z vérifié]

### Hypothèses
- [Hypothèse — risquée]

---

## 2. Étapes d'exécution

### Étape 1 : [Nom]
**Responsable :** [Agent/humain]
**Action :** [Action concrète]
**Livrable :** [Artefact produit]
**Vérification :** [Ce qu'il faut vérifier — la commande peut être un livrable d'une étape précédente]
**Risque :** [Ce qui peut mal tourner]

### Étape 2 : [Nom] `[MUTATION]`
**Responsable :** [Agent/humain]
**Action :** [Action concrète]
**Livrable :** [Artefact produit]
**Vérification :** [Ce qu'il faut vérifier — la commande peut être un livrable d'une étape précédente]
**Risque :** [Ce qui peut mal tourner]

### Étape 3 : [Nom] `[PARALLÈLE]`
**Responsable :** [Agent/humain]
**Action :** [Action concrète]
**Livrable :** [Artefact produit]
**Vérification :** [Ce qu'il faut vérifier — la commande peut être un livrable d'une étape précédente]
**Risque :** [Ce qui peut mal tourner]

---

## 3. GO gates

| Gate | Après étape | Condition | Qui valide |
|------|-------------|-----------|------------|
| GO-1 | Étape 1 | Prérequis vérifiés + plan relu | [Humain] |
| GO-2 | Étape 2 | Dry-run OK | [Humain] |

---

## 4. Rollback

### Niveau 1 : Annulation immédiate
[Action]

### Niveau 2 : Restauration
[Backup — où, quand, comment]

### Niveau 3 : Rollback complet
[Étapes inverses]

### Déclencheurs de rollback
- [Condition observable]
- [Condition observable]

---

## 5. Vérification finale

### Critères de fin
- [ ] [Critère vérifiable]
- [ ] [Readback source servie]

### Preuve de succès
- [Commande/URL/statut]

---

## 6. Risques et points d'attention (optionnel)

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| [Risque] | Faible/Moyen/Élevé | [Impact] | [Action] |

---

## 7. Notes (optionnel)

[Décisions, alternatives, contexte]