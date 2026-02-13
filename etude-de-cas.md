# Étude de cas : Refonte de Maprocuration.gouv.fr

## Introduction

L'objectif de cette étude est d'évaluer à la fois vos connaissances techniques ainsi que vos capacités d'analyse et de présentation.

Comme tout cas d'étude, il n'y a pas de bonne ou de mauvaise réponse... uniquement une belle opportunité pour vous de nous présenter vos convictions et votre cheminement.

**Livrables attendus :**
- Présentation : 30 mn environ, entre 5 et 10 slides
- POC technique pour démontrer la faisabilité de votre approche (détails en section 4)

> Si vous avez des questions sur l'étude de cas, n'hésitez pas à les envoyer par mail au consultant qui est en copie... Good Luck ! ;)

---

## 1. Maprocuration.gouv.fr

### Contexte

Le site **maprocuration.gouv.fr** est une plateforme gouvernementale mise en ligne en 2017 permettant aux citoyens français de :
- Faire une demande de procuration électorale en ligne
- Être mandataire pour un autre électeur
- Gérer et suivre leurs procurations

Le service est utilisé massivement lors des périodes électorales (présidentielles, législatives, européennes) avec des pics de trafic importants.

### Situation actuelle

#### Architecture technique

- **Frontend :** Application web développée avec un framework JavaScript des années 2015 (Angular.js 1.x)
- **Backend :** API REST développée en Java 8 avec Spring Boot 1.5, déployée sur des serveurs Tomcat 8
- **Base de données :** PostgreSQL 9.6 sur une architecture maître-esclave
- **Authentification :** FranceConnect (SSO gouvernemental) + authentification locale
- **Infrastructure :** hébergement on-premise dans un datacenter ministériel
- **Intégration :** Webservices SOAP synchrones avec le Répertoire Electoral Unique (REU) du Ministère de l'Intérieur

### Pain points identifiés

#### Technique

- Stack vieillissante (Angular.js 1.x en EOL, Java 8 bientôt non supporté)
- Performance dégradée en période de pic (timeouts fréquents sur les appels SOAP vers le REU)
- Couplage fort entre frontend et backend (pas de réutilisation possible pour une app mobile)
- Déploiement monolithique : chaque release nécessite un arrêt complet du service
- Tests automatisés limités (couverture < 40%)
- Logs non structurés, difficultés de debugging en production
- Pas de monitoring temps réel des parcours utilisateurs
- Scalabilité verticale uniquement (serveurs surdimensionnés qui tournent à 10% en temps normal)

#### Organisationnel

- Équipe de 8 personnes (5 devs, 2 ops, 1 PO)
- Release tous les 4-6 mois (vs 2 mois initialement)
- Blocage récurrent sur les environnements de test (conflits de déploiement)
- Délai important pour les demandes marketing/communication (ex : ajout d'une bannière d'information)
- Pas de culture DevOps (silos dev/ops marqués)

#### Métier

- Pics de charge non anticipés : lors de la présidentielle 2022, le site a connu plusieurs interruptions de service
- Parcours utilisateur non optimisé (taux d'abandon de 35% sur la procédure complète)
- Pas d'accessibilité RGAA complète (non-conformité aux normes gouvernementales)
- Impossibilité de faire de l'A/B testing pour améliorer les parcours
- Pas d'app mobile native (uniquement site responsive)
- Expérience utilisateur datée, nombreux retours négatifs

### Impacts attendus de la refonte

#### Court terme (6-12 mois)

- Moderniser la stack technique (frameworks à jour, sécurité renforcée)
- Améliorer la performance et la disponibilité (objectif 99.9% de SLA en période électorale)
- Mettre en conformité RGAA (accessibilité niveau AA minimum)
- Réduire le time-to-market (releases mensuelles)

#### Moyen terme (12-24 mois)

- Lancer une application mobile iOS/Android
- Permettre l'intégration d'autres services citoyens (changement d'adresse, carte électorale)
- Ouvrir des APIs aux collectivités territoriales (mairies, préfectures) pour consultation
- Implémenter des notifications push/email pour le suivi des procurations
- Améliorer l'expérience utilisateur (parcours simplifié, pré-remplissage intelligent)

#### Long terme (24+ mois)

- Intégrer l'IA pour :
  - Chatbot d'assistance citoyen (FAQ, aide au remplissage)
  - Détection de fraude (analyse de patterns suspects)
  - Recommandations de mandataires proches géographiquement
- Internationalisation (citoyens français à l'étranger)
- Analytics avancés pour piloter les améliorations continues

### Contraintes

#### Réglementaires

- Conformité RGPD stricte (données personnelles sensibles)
- Hébergement obligatoire sur le territoire français (données souveraines)
- Certifications SecNumCloud ou équivalent pour l'hébergement
- Accessibilité RGAA niveau AA obligatoire
- Conservation des logs d'audit pendant 5 ans

#### Volumétrie

- Trafic normal : 500 utilisateurs simultanés
- Pic électoral : 15 000 utilisateurs simultanés attendus (élection présidentielle)
- 2 millions de procurations/an en moyenne
- Pics saisonniers prévisibles (calendrier électoral connu à l'avance)

#### Budget/Délai

- Prochaine échéance électorale majeure : 18 mois
- Budget migration : non communiqué, mais contraintes budgétaires fortes (service public)
- Pas de possibilité d'interruption prolongée du service (max 4h de maintenance)

---

## 2. Architecture physique détaillée

L'architecture physique actuelle mise en œuvre est la suivante :

```
                    Internet
                       |
                 [Firewall WAF]
                       |
                [Load Balancer F5]
                       |
    +------------------+------------------+
    |                  |                  |
[Apache 2.4]      [Apache 2.4]      [Apache 2.4]
Contenu statique  Contenu statique  Contenu statique
Angular.js        Angular.js        Angular.js
    |                  |                  |
    |      [AJP]       |      [AJP]       |
    +------------------+------------------+
                       |
    +------------------+------------------+
    |                  |                  |
[Tomcat 8.5]      [Tomcat 8.5]      [Tomcat 8.5]
Spring Boot       Spring Boot       Spring Boot
API REST          API REST          API REST
    |                  |                  |
    +------------------+------------------+
                       |
                  [JDBC Pool]
                       |
         +-------------+-------------+
         |                           |
  [PostgreSQL 9.6]          [PostgreSQL 9.6]
     Master (RW)               Slave (RO)
       RHEL 7                   RHEL 7
         |
         |
    [Webservices]
         |
   [SOAP/HTTPS]
         |
         v
  [REU - Répertoire Electoral Unique]
  [Ministère de l'Intérieur]
  [Mainframe AS/400]
         |
         v
  [Autres systèmes ministériels]
  - Fichier des électeurs
  - Système de gestion des cartes électorales
```

### Détails techniques complémentaires

- **Frontend :** Angular.js 1.6, build Gulp, pas de SSR
- **Backend :** Java 8, Spring Boot 1.5, Spring Security, Hibernate 5
- **Cache :** Ehcache local (pas de cache distribué)
- **Sessions :** Stockées en base PostgreSQL (sticky sessions sur le load balancer)
- **Logs :** Fichiers texte sur chaque serveur, pas d'agrégation centralisée
- **Monitoring :** Nagios basique (ping, CPU, RAM) - pas de monitoring applicatif
- **Backup :** Dump PostgreSQL quotidien, rétention 30 jours

---

## 3. Organisation

### Équipe actuelle

#### Développement (5 personnes)

- 2 développeurs fullstack Java/Angular (seniors)
- 2 développeurs Java backend (intermédiaires)
- 1 développeur frontend Angular (junior)

#### Exploitation (2 personnes)

- 1 administrateur système Linux/PostgreSQL
- 1 ingénieur réseau/sécurité

#### Produit (1 personne)

- 1 Product Owner (à mi-temps sur le projet)

### Processus actuel

#### Développement

- **Méthodologie utilisée :** Scrum sprints de 3 semaines
- **Gestion du code :** GitLab on-premise, feature branches, merge requests
- **Revue de code :** obligatoire mais souvent superficielle (manque de temps)
- **Tests :** unitaires partiels (40% de couverture), les tests E2E sont manuels
- **Documentation :** wiki interne peu maintenu

#### Déploiement

- **Environnements :** DEV, RECETTE, PRE-PROD, PROD
- **Pipeline CI/CD :** GitLab CI basique (build + tests unitaires)
- **Déploiement :** Manuel via scripts Ansible (exécutés par les ops)
- **Fréquence :** 1 release tous les 4-6 mois
- **Maintenance :** Fenêtre de 4h le dimanche matin (arrêt complet)

### Pain points organisationnels

- Silos marqués entre dev et ops (pas de culture DevOps)
- Environnements de recette souvent instables (conflits de déploiement)
- Délai important pour déployer un hotfix critique (minimum 48h)
- Pas de feature flags (impossible de désactiver une fonctionnalité sans redéployer)
- Documentation technique obsolète
- Onboarding des nouveaux développeurs difficile (plusieurs semaines)
- Pas de tests de charge réguliers (découverte des problèmes en production)

---

## 4. Demande

Un projet de refonte de la plateforme Maprocuration est à l'étude. Le DSI du ministère vous sollicite pour l'aider à atteindre l'impact à 12 mois et 24 mois.

### 4.1 Présentation (5-10 slides, 30 minutes)

Votre présentation devra couvrir les aspects suivants :

#### 1. Diagnostic de l'existant

- Quels sont les enjeux d'architecture auxquels devra répondre la nouvelle plateforme pour permettre de lever les difficultés actuelles et répondre aux enjeux métier envisagés ?
- Parmi les solutions techniques ou l'architecture mises en place, est-ce que des choses vous surprennent ou vous semblent problématiques ?

#### 2. Architecture cible

- Quelle architecture logicielle mettriez-vous en place si vous deviez refaire la plateforme from scratch ? (big picture d'architecture, découpage en domaines métier, bounded contexts potentiels)
- Quelle architecture physique proposez-vous ? (déploiement, scalabilité, haute disponibilité)
- Quels choix de frameworks, outils, couches logicielles mettriez-vous en œuvre ?
- Quelle stratégie de migration préconisez-vous ? (big bang, migration incrémentale, strangler pattern, etc.)

#### 3. Gestion des pics de charge

- Comment gérez-vous la montée en charge de 500 à 15 000 utilisateurs simultanés ?
- Quelle stratégie de cache proposez-vous ?
- Comment assurez-vous la disponibilité pendant les périodes critiques ?

#### 4. Intégrations

- Comment gérez-vous l'intégration avec le REU (système legacy en SOAP synchrone) ?
- Quelle stratégie pour exposer des APIs aux partenaires (collectivités territoriales) ?
- Comment gérez-vous l'authentification via FranceConnect ?

#### 5. Organisation et processus

- D'un point de vue processus / organisation, quels aménagements proposeriez-vous pour régler les problèmes présentés ci-dessus ?
- Comment accélérer les releases tout en maintenant la qualité ?
- Comment améliorer la collaboration dev/ops ?
- Quelle organisation d'équipe proposez-vous (stream-aligned teams, platform team, etc.) ?

#### 6. Risques et mitigation

- Quels sont les principaux risques techniques et comment les mitigez-vous ?
- Quelle stratégie de rollback en production ?
- Comment assurez-vous la continuité de service pendant la migration ?

### 4.2 POC Technique

En complément de votre présentation, vous devrez réaliser un POC technique démontrant la faisabilité d'un élément critique de votre architecture.

Nous n'attendons pas une application complète, mais une démonstration de votre compréhension des enjeux et de votre capacité à les adresser techniquement.

#### Scénarios au choix (choisir 1 parmi 7)

**Option 1 : Gérer la charge**
- Contexte : Démontrer votre approche pour gérer les pics de charge et l'agrégation de données pour différents clients (web, mobile).

**Option 2 : Architecture Événementielle - Notifications citoyens**
- Contexte : Démontrer une architecture asynchrone pour découpler les traitements et améliorer la scalabilité.

**Option 3 : Migration Progressive**
- Contexte : Démontrer votre approche pour migrer progressivement le système legacy vers une nouvelle architecture sans interruption de service.

**Option 4 : Intégration Legacy**
- Contexte : Démontrer comment intégrer proprement avec un système legacy (REU en SOAP) sans polluer votre nouvelle architecture.

**Option 5 : Détecter avant les clients les éventuels problèmes en production**
- Contexte : Identifier rapidement les problèmes en production.

**Option 6 : Developer Experience**
- Contexte : Démontrer comment améliorer l'efficacité des développeurs.

**Option 7 : Tout élément critique à votre convenance**
- Contexte : Un élément que vous jugez critique et qui n'est pas dans les 6 options précédemment proposées.

### 4.3 Format de restitution

L'entretien de restitution durera **1h30** et sera organisé comme suit :

**Timing :**
- **30 minutes :** Présentation de votre stratégie et de vos analyses (slides)
- **30 minutes :** Démo du POC et revue de code en live
- **30 minutes :** Discussions approfondies (tech, orga, humain)

---

## 5. Notes importantes

### Liberté technologique

L'énoncé mentionne des technologies Java/Spring, mais vous êtes **totalement libre** de choisir la stack technologique que vous préférez (Java, Go, Python, Node.js, Rust, etc.) afin de répondre à l'étude de cas.

Nous évaluons votre capacité à étayer et défendre vos choix en fonction du contexte et des enjeux de l'entreprise, pas de votre connaissance d'une technologie particulière.

### Ressources autorisées

Vous êtes libre d'utiliser tous les outils à votre disposition.

**IMPORTANT :** Les choix d'architecture doivent être les vôtres.

---

**Bonne chance ! Nous avons hâte de découvrir votre approche.**
