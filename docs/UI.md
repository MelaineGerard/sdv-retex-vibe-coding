# 🎨 **Cahier des Charges UI/UX — ENT CÉSAR**

## Version 1.0 – Spécifications ergonomiques, interfaces et expérience utilisateur

---

## 1. **Objectifs UX du projet**

L’interface de l’ENT CÉSAR doit viser :

* **Simplicité** : les utilisateurs doivent accéder rapidement aux informations prioritaires.
* **Efficacité** : réduction du nombre de clics nécessaires pour accomplir les tâches courantes.
* **Cohérence** : un design systémique, homogène sur toutes les pages et tous les modules.
* **Accessibilité** : conformité RGAA au maximum possible.
* **Modernité** : esthétique contemporaine, fluide et professionnelle.
* **Adaptabilité** : compatible mobile, tablette, desktop.
* **Personnalisation** : tableau de bord modulable selon les profils.

---

## 2. **Personas & profils utilisateurs**

### 2.1 Personas principaux

1. **Élève (collège/lycée)**

   * Utilisations prioritaires : cahier de texte, notes, messagerie.
   * Attentes : rapidité, clarté, intuitivité, lisibilité.

2. **Enseignant**

   * Utilisations prioritaires : messagerie, dépôt de documents, saisie de contenus pédagogiques.
   * Attentes : peu de clics, interface claire pour publier.

3. **Parent / Responsable légal**

   * Utilisations : consultation des notes, communications, suivi du travail.
   * Attentes : simplicité, lisibilité, vue globale.

4. **Personnel administratif / direction**

   * Utilisations : communication institutionnelle, accès à des modules administratifs.
   * Attentes : efficacité, fiabilité, vision globale.

---

## 3. **Principes UX fondamentaux**

### 3.1 Design centré utilisateur

* Toutes les fonctionnalités doivent être atteignables en **3 clics maximum**.
* Navigation par **scénarios d’usage** et non par organigramme administratif.

### 3.2 Lisibilité et hiérarchie

* Priorité au **contenu pédagogique** sur le décoratif.
* Utilisation cohérente des espacements, tailles de titres, contrastes.

### 3.3 Accessibilité

* Contrastes conformes à WCAG AA.
* Navigation clavier possible.
* Éléments cliquables larges et espacés.
* Font-size minimum : **16px**.

### 3.4 Responsivité

L’ENT doit être conçu en **mobile-first**, puis étendu à tablette et desktop.

---

## 4. **Charte graphique (directives)**

### 4.1 Palette de couleurs

Objectif : sobriété, professionnalisme et modernité.
Palette recommandée :

* **Couleurs principales :** nuances de bleu ou vert (thème éducatif).
* **Neutres :** blanc, gris clair, gris foncé pour le texte.
* **Accent :** couleur secondaire pour boutons importants (orange ou violet léger).

Critères :

* Contraste suffisant avec le texte.
* Cohérence sur l’ensemble des modules.

### 4.2 Typographie

* **Typo principale :** Sans serif moderne (ex. Inter, Roboto, Source Sans).
* Grades prévus :

  * Titre H1 : 28–32px
  * H2 : 22–24px
  * H3 : 18–20px
  * Texte : 16px minimum
  * Légendes : 14px maximum

### 4.3 Iconographie

* Pack d’icônes unique (Material Icons, Lucide, ou équivalent).
* Style : linéaire, minimaliste.

### 4.4 Composants UI standardisés

* Boutons primaires / secondaires strictement définis.
* Champs de texte avec labels explicites au-dessus.
* Cartes (cards) pour l’organisation des contenus du tableau de bord.
* Modal minimaliste.
* Système de tags et badges (travail urgent, devoir à rendre, nouveau message…).

---

## 5. **Structure générale de l’interface**

### 5.1 Page d’accueil / Tableau de bord

Un tableau de bord personnalisable selon les rôles.
Widgets possibles :

* Messages
* Travail à faire
* Derniers documents reçus
* Notes récentes
* Cours du jour
* Notifications importantes

Objectifs :

* Visibilité immédiate des tâches urgentes.
* Aucune surcharge visuelle.

### 5.2 Barre de navigation

Navigation latérale ou haute, contenant :

* Accueil
* Messagerie
* Cahier de texte
* Notes
* Ressources / Documents
* Visioconférence
* Applications
* Administration (selon droits)

Elle doit être **repliable** sur mobile et tablette.

### 5.3 Pages internes

#### Messagerie

* Interface à 3 colonnes (contacts / liste des messages / contenu).
* Système de tags (non lu, important).

#### Cahier de texte

* Vue calendrier / liste.
* Filtrage par matière, enseignant, date.
* Couleurs adaptées pour les matières.

#### Notes

* Graphiques simples (courbes, barres).
* Mise en évidence des moyennes.
* Lisibilité prioritaire pour les parents.

#### Visioconférence

* Bouton “Rejoindre” très visible.
* Indication du créneau, professeur, classe.

#### Documents

* Organisation par dossiers.
* Drag & drop pour upload.
* Aperçu rapide.

---

## 6. **Interactions & micro-interactions**

* Feedback instantané (chargement, notifications, validations).
* Animations discrètes (fade, slide léger).
* Affichage clair en cas d’erreur, avec texte explicatif.
* Notification non intrusive en cas de message ou de nouveau document.

---

## 7. **Règles de navigation**

* Toujours afficher **où l’utilisateur se trouve** (breadcrumbs).
* Boutons d’action toujours en bas à droite ou en haut à droite.
* Retour arrière toujours possible.
* Pas de “pages mortes”.

---

## 8. **Tests utilisateurs (UX Research)**

### 8.1 Tests prévus

* Tests de maquettes (Figma / XD).
* Tests sur prototypes interactifs avec :

  * 3 élèves
  * 3 enseignants
  * 2 parents
  * 1 membre administratif

### 8.2 Indicateurs de réussite UX

* Temps pour effectuer une tâche < 30 secondes.
* 90 % de compréhension des icônes.
* 0 point bloquant remonté lors des tests.

---

## 9. **Livrables UI/UX attendus**

1. **Guide de conception (Design System complet)**

   * Composants
   * Icônes
   * Couleurs
   * Styles typographiques
   * Règles d’espacement
   * Modèles de pages

2. **Maquettes haute fidélité (desktop + mobile)**

3. **Prototype interactif**

4. **Spécifications précises pour les développeurs**

   * États des composants (normal, hover, désactivé, erreur…).
   * Marges, espacements, tailles précises.

5. **Tests utilisateurs documentés**

---

## 10. **Critères de validation UI/UX**

L’interface sera considérée conforme si :

* L’utilisateur accède aux fonctions principales en ≤ 3 clics.
* Tous les écrans sont responsives et accessibles.
* L’interface est cohérente dans l’ensemble de l’ENT.
* Les tests utilisateurs montrent une satisfaction ≥ 80 %.
* Le design respecte la charte et le design system.
