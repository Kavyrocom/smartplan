---
name: smartplan
description: "Use when asked to create, plan, or deploy a structured non-code process: business workflow, automation setup, data migration, product launch, operational rollout, or multi-agent execution plan with GO gates and rollback."
version: 1.4.0
author: David Schkiwisk (kavyro.com)
license: MIT
metadata:
  hermes:
    tags: [planning, process, deployment, operations, workflow, rollback, go-gates]
    related_skills: [software-delivery-workflow]
---

# Smartplan

## Overview

Produire un plan d'exécution structuré pour un process non-code : déploiement métier, migration, automatisation, lancement, rollout opérationnel.

**Principe : un bon plan rend l'exécution évidente. Si quelqu'un doit deviner, le plan est incomplet.**

Ce skill n'est pas pour le code (utiliser `software-delivery-workflow` ou `superpowers:writing-plans`). Il couvre les plans de process métier, opérationnel et multi-agent.

**Note** : `superpowers:writing-plans` et `superpowers:executing-plans` sont 100% code-only (TDD, file paths, git commits, pytest). Ne pas les charger pour un plan de process non-code.

## When to Use

- Déployer un process métier (onboarding, support, funnel, reporting)
- Mettre en place une automatisation (cron, webhook, n8n, Make, Zapier, etc.)
- Migrer des données (CMS, CRM, DB, spreadsheet)
- Lancer un produit, un service, une offre
- Coordonner plusieurs agents ou acteurs sur une mission partagée
- Déployer une nouvelle procédure opérationnelle

**Ne pas utiliser pour :**
- Plan d'implémentation code → `software-delivery-workflow`
- Plan d'implémentation code TDD → `superpowers:writing-plans`
- L'automatisation elle-même (le cron, le webhook, le job) → le skill produit le **plan de déploiement**, pas l'automatisation. Le cronjob est un livrable du plan, pas une raison d'exclure le plan.
- Process simple tenant en 2-3 actions → action directe, pas de plan. Un plan n'apporte de valeur que si la séquence, les responsabilités ou les GO gates réduisent un risque réel.

## Core behavior

Pour ce tour, tu produis un plan. Tu ne l'exécutes pas.

- Ne pas exécuter le process pendant la planification
- Ne pas modifier de système externe (CMS, CRM, DB, DNS, production)
- Tu peux inspecter le contexte avec des commandes read-only
- Le livrable est un plan markdown structuré

## Plan document structure

Le plan doit contenir ces sections, dans cet ordre. Chaque section est obligatoire sauf si explicitement marquée "optionnel".

### Header

```markdown
# Plan de déploiement : [Nom du process]

**Objectif :** [Une phrase — ce que ce plan accomplit]
**Périmètre :** [Ce qui est inclus / ce qui est explicitement exclu]
**Propriétaire :** [Qui pilote l'exécution — agent ou humain]
**Date :** [Date de création]
**Statut :** Brouillon | Validé | En exécution | Terminé | Annulé
```

### 1. Contexte et prérequis

```markdown
## 1. Contexte et prérequis

### État actuel
[Ce qui existe aujourd'hui — décrire la situation de départ]

### Prérequis
- [ ] [Ce qui doit être en place avant de commencer — systèmes, accès, données, permissions]
- [ ] [Chaque prérequis est énoncé sous forme vérifiable : "X est accessible via Y". En mode plan-only, "vérifié" = énoncé en forme vérifiable, pas testé pour de réel.]

### Hypothèses
- [Si une information manque (timezone, propriétaire, métriques, fournisseur), ne pas deviner en silence. Faire une hypothèse explicite ici et la marquer comme risquée. L'exécuteur ou le valideur pourra la corriger au GO-1.]
```

### 2. Étapes d'exécution

Chaque étape est une action concrète avec un responsable, un livrable et une vérification.

```markdown
## 2. Étapes d'exécution

### Étape 1 : [Nom de l'étape]
**Responsable :** [Agent / humain — nommer explicitement]
**Action :** [Ce qui doit être fait — concret, pas vague]
**Livrable :** [Ce que produit cette étape — artefact, fichier, statut]
**Vérification :** [Comment confirmer que l'étape est OK — ce qu'il faut vérifier, pas nécessairement une commande. La commande ou le script peut être un livrable de l'étape précédente.]
**Risque :** [Ce qui peut mal tourner — et comment le détecter]

### Étape 2 : ...
```

**Règles d'étape :**
- Une étape = une action vérifiable, pas un groupe flou
- Le responsable est nommé (agent, profil, ou humain)
- La vérification est exécutable : commande, URL, readback, checkbox
- Si l'étape modifie un système externe, elle est marquée `[MUTATION]`
- Si plusieurs étapes sont indépendantes et sans shared state, les marquer `[PARALLÈLE]` ; sinon elles sont séquentielles par défaut

### 3. GO gates

Points où l'exécution s'arrête et attend une validation explicite.

```markdown
## 3. GO gates

| Gate | Après étape | Condition | Qui valide |
|------|-------------|-----------|------------|
| GO-1 | Étape 2 | Prérequis vérifiés + plan relu | [Humain] |
| GO-2 | Étape 5 | Migration testée en dry-run | [Humain] |
| GO-3 | Étape 7 | Readback prod OK | [Humain] |
```

**Règles GO :**
- Un GO est obligatoire avant toute mutation externe, publication, envoi, achat ou changement de permissions
- Un GO n'est pas requis pour lecture, diagnostic, brouillon ou préparation
- Un GO peut autoriser une séquence bornée de mutations si celles-ci sont listées explicitement dans la condition du GO. Sinon, une mutation = un GO.
- Les test sends vers des systèmes externes (email, Slack, API) sont des mutations et nécessitent un GO
- Le validateur est toujours une personne réelle (pas un agent). Si aucun nom n'est disponible, utiliser un rôle identifié (ex: "Responsable Ops", "Chef de projet") et le marquer comme hypothèse risquée.
- Sans GO, l'exécution s'arrête et attend

### 4. Rollback

Comment revenir en arrière si ça casse.

```markdown
## 4. Rollback

### Niveau 1 : Annulation immédiate
[Action pour revenir en arrière — commande, restauration, switch]

### Niveau 2 : Restauration
[Backup nécessaire avant l'étape risquée — où, quand, comment]

### Niveau 3 : Rollback complet
[Revenir à l'état initial — étapes inverses]

### Déclencheurs de rollback
- [Condition qui déclenche un rollback — error rate, readback KO, données corrompues]
- [Pas de "si ça se passe mal" vague — des conditions observables]
```

### 5. Vérification finale

```markdown
## 5. Vérification finale

### Critères de fin
- [ ] [Chaque critère est vérifiable indépendamment]
- [ ] [Le process fonctionne dans les conditions réelles]
- [ ] [Readback effectué sur la source servie]

### Preuve de succès
- [Artefact ou commande qui prouve que le process est déployé et fonctionne]
- [URL, fichier, statut, log — pas une affirmation]
```

### 6. Risques et points d'attention (optionnel)

```markdown
## 6. Risques et points d'attention

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| [Risque] | Faible/Moyen/Élevé | [Impact] | [Action préventive] |
```

### 7. Calendrier et jalons (obligatoire si une deadline est donnée)

Quand la demande contient une échéance explicite (ex. "go live dans 2 semaines", date de lancement, migration à réaliser avant une fenêtre de maintenance), ajouter une section calendrier avant les notes. Le calendrier doit transformer les étapes en jalons datés ou J-n, identifier le chemin critique et montrer ce qui peut être parallélisé.

```markdown
## 7. Calendrier et jalons

| Quand | Jalons | Étapes couvertes | Responsable | Condition de sortie |
|-------|--------|------------------|-------------|---------------------|
| J-14 à J-12 | [Jalon] | Étapes 1-3 | [Humain/Agent] | [Preuve vérifiable] |
```

Règles :
- Ne pas inventer une date absolue si seule une durée relative est donnée ; utiliser J-14, Semaine 1, etc.
- Si la deadline est serrée, mettre en évidence les travaux `[PARALLÈLE]` et les décisions qui bloquent le chemin critique.
- Le calendrier ne remplace pas les étapes : il résume l'ordre d'exécution et les points de validation.

### 8. Notes (optionnel)

```markdown
## 7. Notes

[Décisions prises pendant la planification, alternatives envisagées, contexte utile pour l'exécuteur]
```

## Transitions de statut

Le header contient un champ `Statut` qui doit refléter l'état réel du plan :

| Transition | Quand | Qui |
|------------|-------|-----|
| Brouillon → Validé | Plan relu + self-review passé + GO-1 reçu | Humain (GO-1) |
| Validé → En exécution | Exécution démarrée | Agent ou humain |
| En exécution → Terminé | Tous critères de fin vérifiés + preuve de succès | Agent |
| En exécution → Annulé | Rollback déclenché ou décision d'arrêt | Humain |
| Terminé/Annulé → Brouillon | Révision nécessaire après gap détecté | Agent ou humain |

## Révision pendant exécution

Si l'exécution révèle un gap dans le plan (étape manquante, prérequis invalide, vérification impossible) :

1. **Arrêter l'exécution** — ne pas improviser
2. **Repasser le statut en Brouillon**
3. **Réviser le plan** — ajouter/modifier les étapes concernées
4. **Re-faire le self-review**
5. **Re-demander GO** si une mutation était en cours
6. **Reprendre l'exécution** depuis l'étape révisée

Ne jamais continuer l'exécution avec un plan incomplet ou incorrect.

## Save location

Sauvegarder le plan avec `write_file` sous :
- `.hermes/plans/YYYY-MM-DD_HHMMSS-<slug>.md`

Relatif au working directory actif. Si le runtime fournit un chemin spécifique, l'utiliser.

## Post-sauvegarde

Après avoir sauvegardé le plan :

1. Répondre brièvement avec le chemin du plan et un résumé des GO gates
2. Offrir l'exécution : "Le plan est prêt. Tu veux que je l'exécute étape par étape avec les GO gates, ou que je délègue au profil propriétaire ?"
3. Ne pas commencer l'exécution sans réponse explicite

## Validation par stress test (optionnel)

Pour valider un nouveau skill ou une révision majeure, lancer 3-5 subagents en parallèle avec des scénarios variés (migration, lancement, automatisation, multi-agent, rollback). Chaque subagent :
1. Charge le skill via `skill_view`
2. Suit les instructions pour produire un plan réel
3. Reporte : chemin du plan, efficacité du guidage, gaps trouvés, self-review checklist

Consolider les gaps, corriger uniquement les problèmes généralistes (pas les gaps scenario-specific), bumper la version, resync les profils, et pousser une nouvelle release.

## Writing process

### Step 1 : Comprendre la demande
- Lire la demande et identifier le type de process
- Déterminer le périmètre (inclus / exclu)
- Identifier les systèmes concernés

### Step 2 : Cartographier le contexte
- Inspecter le contexte avec read-only si nécessaire
- Identifier les prérequis et dépendances
- Lister les acteurs (agents, humains, systèmes)
- Si la mission implique plusieurs agents Hermes ou un routage entre profils, charger le skill de routage disponible sur l'installation avant d'assigner les responsables. Ne pas confondre avec le routing métier (ex: auto-routing de tickets, routing d'emails) qui est un livrable du plan, pas un routage d'agents.
- Si une information essentielle manque (timezone, propriétaire, métriques, fournisseur), faire une hypothèse explicite plutôt que deviner en silence. L'inscrire dans la section Hypothèses du plan.

### Step 3 : Séquencer les étapes
- Décomposer en étapes vérifiables
- Marquer les mutations externes `[MUTATION]`
- Marquer les étapes parallélisables `[PARALLÈLE]`
- Insérer les GO gates aux points de rupture

### Step 4 : Définir le rollback
- Identifier l'étape la plus risquée
- Définir le backup préalable
- Définir les déclencheurs de rollback

### Step 4 bis : Définir le calendrier si une échéance existe
- Si l'utilisateur donne une deadline ou une date de go-live, ajouter `## 7. Calendrier et jalons`
- Répartir les étapes en jalons datés ou relatifs (J-14, J-7, J-1, lancement, post-lancement)
- Identifier le chemin critique, les GO gates qui peuvent bloquer l'échéance et les travaux parallélisables
- Pour les lancements commerciaux, inclure aussi les jalons de mesure : objectif de revenu/capacité, suivi des conversions, seuil sold-out ou liste d'attente

### Step 5 : Écrire le plan
- Remplir chaque section
- Pas de placeholder ("TBD", "à définir", "TODO")
- Chaque étape a un responsable, un livrable et une vérification

### Step 6 : Self-review
Vérifier avant de déclarer le plan terminé :

- [ ] Header complet (objectif, périmètre, propriétaire, date, statut)
- [ ] Contexte et prérequis vérifiés
- [ ] Chaque étape est une action concrète, pas un groupe flou
- [ ] Chaque étape a un responsable nommé
- [ ] Chaque étape a une vérification exécutable
- [ ] Les GO gates sont placés avant chaque mutation externe
- [ ] Le rollback est défini avec des déclencheurs observables
- [ ] Les critères de fin sont vérifiables indépendamment
- [ ] Preuve de succès identifiée
- [ ] Si une deadline/date de lancement est donnée, un calendrier de jalons est inclus avec chemin critique et travaux parallélisables
- [ ] Aucun placeholder
- [ ] Plan sauvegardé sous `.hermes/plans/`

## Anti-patterns

### ❌ Étape vague
"Étape 1 : Configurer le système. Étape 2 : Tester."
**Pourquoi c'est mauvais :** Pas d'action concrète, pas de responsable, pas de vérification, pas de GO.
**Correct :** "Étape 1 : Créer le webhook form→endpoint. Responsable : agent. Livrable : webhook ID. Vérification : `curl -s https://example.com/api/webhooks | jq '.[].id'` contient l'ID. GO-1 requis avant activation."

### ❌ Vérification non exécutable
"Vérification : ça devrait marcher."
**Pourquoi c'est mauvais :** Pas de commande, pas d'URL, pas de readback — rien à vérifier.
**Correct :** "Vérification : `curl -s https://example.com/health` retourne 200 + `jq '.status'` retourne `active`."

### ❌ GO gate manquant
Mutation externe sans GO entre la préparation et la production.
**Pourquoi c'est mauvais :** Un changement externe sans validation peut casser la production.
**Correct :** "GO-2 après étape 5 : dry-run OK, validé par [Humain] avant l'étape 6 [MUTATION]."

### ❌ Rollback implicite
"En cas de problème, revenir en arrière."
**Pourquoi c'est mauvais :** Pas d'action concrète, pas de déclencheur, pas de condition.
**Correct :** "Déclencheur : readback KO ou erreur 500 sur l'endpoint. Action : désactiver le webhook via l'API. Backup : export JSON de la configuration avant l'étape 1."

### ❌ Placeholder dans le plan
"Étape 3 : TBD. Risque : à définir."
**Pourquoi c'est mauvais :** Le plan n'est pas exécutable — l'exécuteur doit deviner.
**Correct :** Toutes les sections remplies avec du contenu réel. Si une info manque, la collecter avant d'écrire le plan.

### ❌ Plan pour 2-3 actions simples
Créer un plan complet pour "envoyer un email à X".
**Pourquoi c'est mauvais :** Sur-engineering — le plan coûte plus cher que l'action.
**Correct :** Action directe. Un plan n'apporte de valeur que si la séquence, les responsabilités ou les GO gates réduisent un risque réel.

### ❌ Plan pour du code
Utiliser ce skill pour un plan d'implémentation Python/JS.
**Pourquoi c'est mauvais :** Ce skill n'a pas de TDD, pas de file paths, pas de git workflow.
**Correct :** Rediriger vers `software-delivery-workflow` ou `superpowers:writing-plans`.

### ❌ Plan sans étapes parallèles
Marquer toutes les étapes comme séquentielles quand certaines sont indépendantes.
**Pourquoi c'est mauvais :** L'exécution prend plus de temps que nécessaire.
**Correct :** Marquer les étapes indépendantes `[PARALLÈLE]` pour permettre une exécution concurrente.

### ❌ Confondre routing métier et routing d'agents
Charger un skill de routing Hermes pour un plan qui contient du routing métier (auto-routing de tickets, routing d'emails).
**Pourquoi c'est mauvais :** Le routing métier est un livrable du plan, pas un routage entre profils Hermes.
**Correct :** Charger le skill de routage seulement si plusieurs agents Hermes doivent se coordonner. Le routing métier est une étape du plan.

### ❌ Deviner en silence quand une info manque
Utiliser une timezone, un propriétaire ou un fournisseur par défaut sans le dire.
**Pourquoi c'est mauvais :** L'exécuteur ou le valideur ne sait pas que c'est une hypothèse, pas un fait.
**Correct :** Faire une hypothèse explicite dans la section Hypothèses, marquée comme risquée. L'exécuteur peut la corriger au GO-1.