# 📘 **Cahier des Charges – ENT CÉSAR**

## **Version 1.0 — Document fonctionnel & technique**

---

## 1. **Contexte du projet**

L’ENT **CÉSAR** est un projet de création d’un nouvel Espace Numérique de Travail visant à remplacer et moderniser un ENT existant devenu obsolète.
Ce nouvel outil doit améliorer la communication, centraliser les outils pédagogiques, simplifier les usages numériques et répondre aux besoins actuels de l’ensemble de la communauté éducative.

Objectifs principaux :

* Moderniser l’interface et l’ergonomie.
* Améliorer la rapidité et la fiabilité.
* Centraliser les communications, documents et outils pédagogiques.
* Favoriser le travail collaboratif.
* Renforcer la sécurité et la conformité règlementaire.
* Permettre une évolution future plus simple (modularité).

---

## 2. **Publics concernés**

L’ENT CÉSAR doit s’adresser à un ensemble d’utilisateurs variés, chacun avec des droits et interfaces adaptées :

* **Élèves**
* **Enseignants**
* **Personnel administratif et direction**
* **Parents / responsables légaux**
* **Partenaires institutionnels**
* **Intervenants extérieurs** (AESH, associations, conférenciers, etc.)

Chaque profil disposera de permissions spécifiques et d’un tableau de bord personnalisé.

---

## 3. **Fonctionnalités attendues**

### 3.1. **Messagerie interne**

* Envoi / réception de messages entre utilisateurs.
* Groupes de communication (classe, équipe pédagogique…).
* Notifications (web, mail, mobile).

### 3.2. **Gestion des notes et évaluations**

* Consultation des notes par élèves/parents.
* Publication des évaluations par les enseignants.
* Statistiques simplifiées.

### 3.3. **Cahier de texte / agenda**

* Planning par classe et par utilisateur.
* Travail à faire, documents joints, échéances.
* Synchronisation possible avec calendriers externes (optionnel).

### 3.4. **Visioconférence intégrée**

* Salle virtuelle pour cours en ligne.
* Possibilité d’enregistrer les sessions (si autorisé).
* Intégration avec un service existant (ex : BigBlueButton, Jitsi, Teams…).

### 3.5. **Applications pédagogiques**

* Bibliothèque d’applications internes et externes.
* Connexion unifiée (SSO) vers les outils autorisés.
* Possibilité d’ajouter des modules tiers dans le futur.

### 3.6. **Gestion documentaire**

* Espace personnel de stockage.
* Partage de documents (classe, équipe, projets).
* Versioning, aperçu des fichiers, droits fins.

### 3.7. **Tableau de bord personnalisé**

* Widgets selon le profil (notes, messages, agenda, alertes…)
* Interface moderne, responsive et accessible.

---

## 4. **Interopérabilité & Intégrations**

L’ENT CÉSAR doit être interopérable avec :

* Solutions de gestion existantes (ex : bases élèves, systèmes académiques).
* Annuaire LDAP / Active Directory (si existant).
* Solutions d’authentification : **SSO/SAML/OAuth2**.
* Applications pédagogiques tierces (bibliothèques numériques, manuels en ligne, outils collaboratifs…).

Formats supportés :

* Import/export CSV, XML, Excel.
* API REST pour interactions avec services externes.

---

## 5. **Sécurité & conformité**

Les exigences minimales incluent :

* **Conformité RGPD renforcée.**
* Stockage des données sur des serveurs **localisés en France ou en Europe**.
* Chiffrement des données en transit (HTTPS) et au repos.
* Journalisation des actions sensibles.
* Protection contre les accès non autorisés (authentification forte possible).
* Sauvegardes quotidiennes + plan de reprise d’activité (PRA).

---

## 6. **Conception UX/UI**

L’ENT CÉSAR doit proposer :

* Une interface moderne, épurée et responsive.
* Un design adapté aux ordinateurs, tablettes et smartphones.
* Accessibilité conforme au RGAA au maximum possible.
* Navigation simplifiée en trois clics maximum pour les actions principales.
* Charte graphique moderne (couleurs sobres, icônes claires, lisibilité prioritaire).

---

## 7. **Architecture technique (proposition)**

* Architecture **modulaire** permettant l’ajout de nouveaux services.
* Infrastructure **cloud** (souverain de préférence) ou hébergement interne selon décision ultérieure.
* Backend basé sur un framework robuste (ex : Java Spring, Node.js, .NET, selon choix du MOE).
* Base de données sécurisée (PostgreSQL ou équivalent).
* Frontend responsive basé sur un framework moderne (Vue.js, React, Angular).

---

## 8. **Planning prévisionnel**

Proposition indicative :

| Phase                                | Durée estimée | Description                              |
| ------------------------------------ | ------------- | ---------------------------------------- |
| Analyse & cadrage                    | 1–2 mois      | Ateliers, validation des besoins         |
| Conception fonctionnelle & technique | 1 mois        | Maquettes UX, architecture               |
| Développement phase 1                | 3–4 mois      | Fonctions essentielles                   |
| Tests & retours utilisateurs         | 1 mois        | Corrections                              |
| Déploiement pilote                   | 1 mois        | Mise en place dans un établissement test |
| Améliorations                        | 1–2 mois      | Ajustements                              |
| Déploiement général                  | —             | Après validation                         |

---

## 9. **Budget & ressources**

Le budget est à définir selon :

* Niveau de personnalisation,
* Choix de l’hébergement,
* Modules supplémentaires,
* Maintenance annuelle.

Ressources internes potentielles :

* Administrateurs système pour déploiement / gestion,
* Référents pédagogiques pour les tests,
* Assistance utilisateurs.

---

## 10. **Livrables attendus**

* Cahier des charges complet (présent document).
* Dossier de conception fonctionnelle (maquettes + workflows).
* Architecture technique détaillée.
* Manuel utilisateur (profils variés).
* Procédure d’administration.
* Rapport de tests.
* Version finale prête à déployer.

---

## 11. **Critères de réussite**

* Accessibilité améliorée par rapport à l’ancien ENT.
* Rapidité, stabilité et simplicité d’usage.
* Satisfaction des enseignants, élèves et parents.
* Adoption forte durant la première année.
* Pérennité et évolutivité technique.
