# Exercice technique : API Maprocuration - Gestion des mandats électoraux

## Objet : Exercice technique - Développement API Maprocuration

Bonjour,

Nous sommes ravis de vous proposer cet exercice technique dans le cadre de votre candidature. Cet exercice reflète les problématiques réelles auxquelles nous sommes confrontés sur le projet **maprocuration.gouv.fr**.

### Contexte du projet

Le service **maprocuration.gouv.fr** permet aux citoyens français de gérer leurs procurations électorales en ligne. Ce service gouvernemental connaît des pics d'activité importants lors des périodes électorales et doit garantir fiabilité, sécurité et conformité légale.

Votre mission consiste à développer une API REST permettant de gérer le cycle de vie complet d'une procuration électorale, de sa création à son expiration, en passant par la validation administrative.

---

## Ce que nous attendons de vous

### Objectifs de l'exercice

Cet exercice nous permettra d'évaluer :

- Votre capacité à comprendre et formaliser des besoins métier
- Votre capacité à traduire ces besoins en solution technique cohérente
- Votre maîtrise des fondamentaux backend (API REST, persistance, validation, tests)
- Votre approche de la qualité logicielle (architecture, lisibilité, robustesse)
- Votre autonomie dans les choix techniques et votre capacité à les justifier et les défendre
- Votre vision sur les aspects production (observabilité, résilience, gestion d'erreurs)
- Votre capacité à concevoir une interface utilisateur simple et fonctionnelle

### Format et timing

- **Durée :** Vous disposez de **2 semaines** à compter de la réception de cet email
- **Charge estimée :** Environ **8 à 12 heures** de développement effectif
- **Date limite :** [Date de rendu à préciser]

Nous sommes conscients que vous avez probablement une activité professionnelle et des contraintes personnelles. L'objectif n'est pas de monopoliser votre temps libre, mais d'évaluer votre capacité à produire un travail de qualité dans un contexte réaliste.

---

## Besoins fonctionnels

### User Stories principales

Nous vous fournissons les besoins exprimés sous forme de User Stories. À vous de les formaliser en spécifications détaillées (critères d'acceptance, règles de validation, formats d'API, gestion d'erreurs, modèle de données, etc.).

#### US1 : Faire une demande de procuration

**En tant que** citoyen absent le jour d'une élection  
**Je veux** faire une demande de procuration en ligne  
**Afin que** quelqu'un d'autre puisse voter à ma place

**Informations de contexte :**

Le mandant (personne qui donne procuration) doit fournir ses informations personnelles :
- Numéro électeur
- Nom, prénom
- Date de naissance
- Commune d'inscription
- Email

Le mandataire (personne qui votera à sa place) doit être désigné avec ses informations

Les élections doivent être sélectionnées (une ou plusieurs)

La raison d'absence doit être indiquée (vacances, obligations professionnelles, santé, handicap, autre)

Un justificatif peut optionnellement être joint

**Règles métier :**

- Un citoyen ne peut avoir qu'une seule procuration active par élection
- Un mandataire ne peut recevoir que 2 procurations maximum par élection (limite légale française)
- Le mandant et le mandataire doivent être inscrits dans la même commune
- Les élections sélectionnées doivent être futures (pas de procuration rétroactive)
- La raison d'absence est obligatoire

#### US2 : Consulter mes procurations

**En tant que** citoyen  
**Je veux** consulter la liste de mes procurations (données ou reçues)  
**Afin de** suivre leur état et vérifier qu'elles sont bien enregistrées

**Fonctionnalités attendues :**

- Voir les procurations où je suis mandant (j'ai donné procuration)
- Voir les procurations où je suis mandataire (je vote pour quelqu'un)
- Filtrage par statut, élection
- Pagination
- Tri par date

#### US3 : Consulter le détail d'une procuration

**En tant que** citoyen  
**Je veux** voir le détail complet d'une de mes procurations  
**Afin de** connaître son statut précis et son historique

**Informations attendues :**

- Toutes les données de la procuration
- Statut actuel et dates importantes
- Si validée : qui a validé et quand
- Si rejetée : motif du rejet
- Historique des changements de statut
- Actions disponibles selon le statut

#### US4 : Révoquer une procuration

**En tant que** citoyen mandant  
**Je veux** pouvoir annuler une procuration que j'ai donnée  
**Afin de** voter moi-même ou désigner un autre mandataire

**Règles métier :**

- Délai minimum de 48 heures avant l'élection (délai légal)
- Seul le mandant peut révoquer (pas le mandataire)
- Une procuration révoquée ne peut plus être réactivée
- Le mandataire doit être notifié (simulation acceptable pour l'exercice)

#### US5 : Valider ou rejeter une procuration (agent administratif)

**En tant qu'** agent du service électoral de ma commune  
**Je veux** valider ou rejeter les demandes de procuration  
**Afin de** m'assurer de leur conformité légale avant l'élection

**Règles métier :**

- Seule une procuration "en attente de validation" peut être traitée
- Le motif est obligatoire en cas de rejet
- La validation/rejet est définitive (pas de retour en arrière)
- Le citoyen doit être notifié (simulation acceptable)
- L'agent peut indiquer quels documents ont été vérifiés

#### US6 : Vérifier la disponibilité d'un mandataire

**En tant que** citoyen  
**Je veux** vérifier si je peux accepter une nouvelle procuration  
**Afin de** savoir si j'ai atteint la limite légale ou si j'ai déjà des engagements

**Informations attendues :**

- Nombre de procurations actuelles pour une élection donnée
- Limite autorisée (2)
- Disponibilité restante
- Liste des procurations en cours

#### US7 : Consulter les statistiques (service électoral)

**En tant que** responsable du service électoral  
**Je veux** consulter des statistiques sur les procurations  
**Afin de** piloter l'activité et anticiper la charge de travail

**Métriques attendues :**

- Nombre total de procurations (avec filtres par élection, commune, période)
- Répartition par statut
- Répartition par raison d'absence
- Répartition par commune
- Évolution temporelle (créations/validations par jour)
- Taux de validation
- Délai moyen de validation

#### US8 : Être notifié des événements importants

**En tant que** citoyen (mandant ou mandataire)  
**Je veux** recevoir des notifications sur l'évolution de mes procurations  
**Afin de** rester informé sans avoir à consulter régulièrement la plateforme

**Événements notifiables :**

- Procuration validée (mandant)
- Procuration rejetée avec motif (mandant)
- Nouvelle procuration reçue (mandataire)
- Procuration révoquée (mandataire)
- Rappels avant élection

> **Note :** Pour cet exercice, vous pouvez simuler les notifications (logs structurés, système d'événements en mémoire, message broker local). Nous n'attendons pas d'envoi réel d'emails ou SMS.

#### US9 : Gérer le cycle de vie automatique des procurations

**En tant que** système  
**Je veux** faire évoluer automatiquement le statut des procurations  
**Afin que** les données restent cohérentes avec la réalité

**Règles :**

- Une procuration dont l'élection est passée passe automatiquement en statut "expirée"
- Une procuration expirée n'est plus modifiable
- Les procurations pour plusieurs tours d'une même élection sont gérées indépendamment

---

### Contexte métier additionnel

**Système électoral français** (informations pour vous aider) :

- Le numéro d'électeur est composé de 9 chiffres
- Les communes sont identifiées par un code INSEE (5 chiffres)
- Les élections majeures : présidentielles (2 tours), législatives (2 tours), européennes (1 tour), municipales, etc.
- Un électeur vote dans la commune où il est inscrit sur les listes électorales
- Les bureaux de vote ferment généralement à 20h le jour de l'élection

**Statuts possibles d'une procuration :**

- `EN_ATTENTE_VALIDATION` : Créée, en attente de traitement par un agent
- `VALIDEE` : Validée par un agent, active pour l'élection
- `REJETEE` : Rejetée par un agent (non conforme)
- `REVOQUEE` : Annulée par le mandant
- `EXPIREE` : Élection passée

**À vous de définir :**

- Les autres données de référence dont vous avez besoin (raisons d'absence, élections de test, etc.)
- Le modèle de données complet
- Les formats d'API détaillés
- Les règles de validation précises
- Les messages d'erreur

---

## Ce que nous attendons dans vos livrables

### 1. Spécifications techniques (document séparé attendu)

Nous attendons que vous produisiez un document de spécifications comprenant :

#### Modèle de données

- Entités principales avec leurs attributs
- Relations entre entités
- Contraintes d'intégrité

#### Format d'API détaillé pour chaque endpoint

- Méthode HTTP
- URL avec paramètres
- Headers requis (le cas échéant)
- Format de requête (body) avec exemples
- Format de réponse (success) avec exemples
- Codes d'erreur HTTP et messages associés (avec exemples)

#### Règles de validation pour chaque champ

- Type de données
- Contraintes (longueur, format, plage de valeurs)
- Règles métier associées

#### Cas limites identifiés et leur gestion

- Accès concurrent (deux personnes créant la même procuration)
- Contraintes légales (limites, communes, délais)
- Données incohérentes ou invalides
- Performances (statistiques sur grands volumes)
- Indisponibilité de la base de données

#### Machine à états des procurations

- Diagramme ou description des transitions de statuts
- Conditions de transition
- Actions associées à chaque transition

#### Données de référence

- Liste des élections que vous utilisez pour les tests (avec dates)
- Liste des raisons d'absence possibles
- Exemples de communes
- Toute autre donnée de référence nécessaire

**Format attendu :** Document Markdown, PDF, ou directement dans votre README si bien structuré.

**Objectif :** Nous voulons voir votre capacité à transformer des besoins métier flous en spécifications techniques précises et exploitables. Nous évaluons votre capacité d'analyse et de formalisation.

### 2. Code source

#### API REST

- API fonctionnelle implémentant toutes les User Stories
- Architecture en couches claire
- Tests automatisés (couverture minimum 70% sur la logique métier)
- Gestion robuste des erreurs
- Base de données avec migrations versionnées

#### Interface graphique

Interface web simple permettant de tester l'API

**Pages minimales attendues :**

- Formulaire de création de procuration
- Liste des procurations (avec filtres)
- Détail d'une procuration
- Actions de révocation/validation
- Tableau de bord statistiques (page basique suffisante)

**Design sobre et fonctionnel :** pas besoin de design élaboré, privilégiez la simplicité et l'utilisabilité

**Responsive :** adapté desktop et mobile (basique acceptable)

**Gestion des erreurs** côté client avec messages clairs

**Précisions importantes sur le frontend :**

- Nous n'attendons pas un design pixel-perfect
- L'utilisation d'une bibliothèque CSS (Bootstrap, Tailwind, Material-UI, etc.) est encouragée
- L'objectif est de démontrer votre capacité à connecter une interface avec votre API
- Si vous n'êtes pas à l'aise en frontend, privilégiez la simplicité : un formulaire HTML bien structuré vaut mieux qu'une SPA complexe mais bancale

### 3. Environnement

- Docker Compose permettant de lancer l'ensemble (backend + frontend + BDD)
- Un seul `docker-compose up` doit suffire pour tout démarrer
- Données de test (seed) pour faciliter la démonstration

### 4. Documentation

**README complet avec :**

#### Démarche de spécification

- Comment avez-vous formalisé les besoins ?
- Quelles questions vous êtes-vous posées ?
- Quelles hypothèses avez-vous faites et pourquoi ?
- Quelles ambiguïtés avez-vous identifiées et comment les avez-vous résolues ?

#### Choix techniques justifiés (backend ET frontend)

#### Instructions de démarrage

- Prérequis
- Commandes

#### Comment accéder à l'interface web

#### Comment tester l'API directement si besoin

- Exemples de requêtes curl ou collection Postman/Bruno

#### Checklist des fonctionnalités implémentées

#### Ce que vous feriez avec plus de temps

#### Difficultés rencontrées (optionnel mais apprécié)

---

## Stack technique

Vous êtes totalement libre dans le choix de votre stack technique.

- **Backend :** Java/Spring, Node.js, Python/FastAPI, Go, Rust, .NET, etc.
- **Frontend :** React, Vue, Angular, Svelte, ou HTML/CSS/JS vanilla
- **Base de données :** PostgreSQL, MySQL, MongoDB, SQLite, etc.

**Ce qui importe :**

- Cohérence des choix avec le contexte
- Justification de ces choix
- Capacité à défendre vos décisions

---

## Éléments appréciés (non obligatoires)

- Documentation OpenAPI/Swagger générée
- Collection Postman/Bruno pour tester l'API
- Diagrammes d'architecture (C4, UML, ou autre)
- ADRs (Architecture Decision Records)
- Validation côté client (en plus de côté serveur)
- Logs structurés avec correlation ID
- Healthchecks et métriques
- Gestion de la concurrence (optimistic/pessimistic locking)
- Pipeline CI/CD
- Tests de charge

---

## Ce qui n'est PAS attendu

- Authentification/autorisation réelle (simulation via headers ou sélection utilisateur suffisante)
- Design élaboré ou charte graphique gouvernementale
- Animations complexes ou micro-interactions
- Déploiement en production (cloud, Kubernetes, etc.)
- Envoi réel d'emails ou SMS (logs/événements suffisants)
- Accessibilité RGAA complète (basique suffisant)
- Intégration avec de vraies APIs externes (REU, FranceConnect, etc.)

---

## Utilisation de l'IA générative

Vous êtes libre d'utiliser des outils d'IA générative (ChatGPT, Claude, GitHub Copilot, Cursor, v0, etc.) pour vous assister.

**Cependant :**

- Vous devez comprendre et pouvoir expliquer chaque ligne de code générée
- La formalisation des besoins doit être votre réflexion propre (pas un copier-coller de prompt ChatGPT)
- Les choix d'architecture doivent être justifiés par vous
- Lors de la revue, nous creuserons les détails pour nous assurer de votre maîtrise

> L'IA est un outil, la réflexion et les choix restent humains. Nous détectons rapidement les candidats qui auraient généré du contenu sans le comprendre.

---

## Restitution et évaluation

### Format de l'entretien (1h15)

#### 20 minutes - Démo live

Présentation de votre application en action :
- Parcours complet côté citoyen (création, consultation, révocation)
- Parcours côté agent (validation/rejet)
- Consultation des statistiques
- Gestion d'erreurs et cas limites

#### 25 minutes - Revue des spécifications et du code

- Comment avez-vous formalisé les besoins ?
- Quelles questions vous êtes-vous posées ?
- Quelles hypothèses avez-vous faites et pourquoi ?
- Quelles ambiguïtés avez-vous identifiées ?
- Revue du code backend et frontend
- Questions sur vos choix techniques

#### 30 minutes - Discussion technique

- Que manque-t-il pour passer en production ?
- Comment débogueriez-vous un incident signalé par un citoyen ?
- Comment feriez-vous évoluer l'API sans casser les clients existants ?
- Quelle serait votre stratégie de montée en charge ?
- Comment amélioreriez-vous l'UX avec plus de temps ?

---

### Critères d'évaluation

#### Spécifications et formalisation (25%)

- Clarté et exhaustivité des spécifications
- Pertinence des choix (modèle de données, API design, gestion d'erreurs)
- Identification et traitement des cas limites
- Qualité des hypothèses et justifications
- Capacité à identifier les ambiguïtés

#### Qualité du code backend (20%)

- Lisibilité et maintenabilité
- Architecture cohérente (séparation des responsabilités)
- Gestion robuste des erreurs
- Absence de code mort ou sur-complexité

#### Implémentation métier (15%)

- Complétude des US implémentées
- Exactitude des règles métier
- Gestion appropriée des cas limites
- Validation des données en profondeur

#### Design d'API (15%)

- Cohérence REST (verbes HTTP, codes de statut)
- Structure de réponses uniforme et prévisible
- Documentation (Swagger/OpenAPI ou équivalent)
- Gestion des erreurs avec messages clairs

#### Interface utilisateur (15%)

- Fonctionnalité (toutes les actions possibles)
- Utilisabilité (simplicité, clarté)
- Gestion d'erreurs côté client
- Connexion propre avec l'API

#### Documentation et tests (10%)

- README complet et clair
- Choix justifiés
- Tests pertinents avec bonne couverture
- Transparence sur les limites

---

## Questions et support

Si des points ne sont pas clairs ou si vous avez besoin de précisions, contactez-nous. Nous préférons clarifier en amont plutôt que vous laisser dans l'incertitude.

### Questions courantes anticipées

**Q : Dois-je spécifier tous les détails de l'API avant de coder ?**  
R : Nous attendons une formalisation raisonnable. Vous pouvez faire évoluer vos specs en cours de route, mais documentez vos choix et hypothèses.

**Q : Que faire si un besoin est ambigu ?**  
R : C'est exactement ce que nous voulons évaluer. Faites une hypothèse raisonnable, documentez-la clairement, et expliquez votre raisonnement. Nous discuterons de ces choix lors de l'entretien.

**Q : Dois-je créer des données de référence réalistes ?**  
R : Oui, créez les données dont vous avez besoin (élections, communes, etc.). Pas besoin d'avoir toutes les communes de France, quelques exemples suffisent.

**Q : Je ne suis pas expert en frontend, est-ce éliminatoire ?**  
R : Non. Un formulaire HTML bien structuré avec gestion d'erreurs est suffisant. L'essentiel est de montrer que vous savez faire communiquer un frontend et un backend.

**Q : Quelle priorité entre spécifications et code ?**  
R : Les deux sont importants. Un bon équilibre est attendu. Mieux vaut des spécifications simples mais complètes qu'un document de 50 pages illisible.

**Q : Combien de temps consacrer aux spécifications vs au code ?**  
R : Environ 25-30% du temps sur les spécifications, 70-75% sur le code et les tests. Les specs doivent guider votre développement.

Vous pouvez nous poser vos questions par email à **[adresse email]**. Nous nous engageons à répondre sous 24h ouvrées.

---

## Prochaines étapes

1. **Accusé de réception :** Merci de confirmer la bonne réception de cet email et votre engagement à réaliser l'exercice

2. **Questions éventuelles :** N'hésitez pas à revenir vers nous pour toute clarification

3. **Soumission :** Envoyez-nous le lien vers votre repository Git (ou archive .zip) avant la date limite. Le repository doit contenir :
   - Document de spécifications
   - Code source (backend + frontend)
   - Docker Compose
   - README complet

4. **Planification de la restitution :** Nous vous proposerons plusieurs créneaux pour l'entretien d'1h15

---

Nous avons hâte de découvrir votre approche, vos choix de formalisation et d'échanger avec vous sur votre démarche.
