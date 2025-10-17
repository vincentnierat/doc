# Spécifications Techniques : Cours de Certification IA

**Version : 2.1 (Draft)**
**Date : 16/10/2025**

## 1. Objectif de la fonctionnalité

L'objectif est de créer un **cours de certification sur l'IA** interactif pour les candidats. Le cours est présenté sous la forme d'un tableau de bord, structuré en **chapitres** et **sections**. Chaque section contient une série d'exercices (questions) avec un feedback immédiat pour favoriser l'apprentissage. La progression est suivie au niveau global, par chapitre et par section.

## 2. Parcours Utilisateur

### 2.1. Écran Principal / Tableau de Bord du Cours

1.  **Accès :** L'utilisateur accède à l'écran principal du cours depuis son profil.
2.  **En-tête :** L'écran affiche un titre de bienvenue et des statistiques globales :
    *   **Progression totale :** Un pourcentage représentant l'avancement global dans le cours (ex: 46%).
    *   **Exercices complétés :** Un ratio du nombre total d'exercices terminés (ex: 20/25).
3.  **Grille des Chapitres :**
    *   L'écran présente une grille de cartes, chaque carte représentant un **chapitre**.
    *   Chaque carte de chapitre contient : une illustration unique, le titre du chapitre (ex: "Chapitre 1 : Qu'est-ce que l'IA ?").
    *   À l'intérieur de chaque carte, une liste de **sections** est affichée, avec pour chacune le nombre d'exercices réussis (ex: "I. Comment définir l'IA ? - 1/1").
4.  **Navigation :** Un clic sur une section spécifique d'un chapitre navigue l'utilisateur vers l'écran d'exercices correspondant.

### 2.2. Déroulement d'un Exercice (Questionnaire de Section)

1.  **Affichage :** L'écran affiche les questions/exercices pour la section sélectionnée.
2.  **Interaction :** L'utilisateur répond aux questions. Les types de réponses peuvent être : choix unique (radio), choix multiples (checkbox), texte libre (input) ou nombre (input number).
3.  **Validation :** L'utilisateur clique sur "Valider".
4.  **Correction :** Pour chaque question, la correction s'affiche (réponse de l'utilisateur, bonne réponse si erreur, explication).
5.  **Progression :** Une fois l'exercice terminé, l'utilisateur est redirigé vers le tableau de bord principal, qui affiche maintenant sa progression mise à jour.

## 3. Architecture Technique

### 3.1. Modèles de Données (Typescript)

La nouvelle structure hiérarchique est la suivante :

```typescript
// types/aiCourse.ts

// --- Modèles pour le Tableau de Bord ---
export interface CourseStats {
  completionPercentage: number; // Pourcentage de complétion global
  completedExercises: number;
  totalExercises: number;
}

export interface SectionProgress {
  sectionId: string;
  titleKey: string;
  completedExercises: number;
  totalExercises: number;
}

export interface Chapter {
  chapterId: string;
  titleKey: string;
  illustrationUrl: string;
  sections: SectionProgress[];
}

export interface CourseDashboard {
  stats: CourseStats;
  chapters: Chapter[];
}

// --- Modèles pour un Exercice de Section ---
export type QuestionType = 'radio' | 'checkbox' | 'text' | 'number';

export interface QuestionOption { /* ... comme avant ... */ }

export interface Question { /* ... comme avant ... */ }

export interface SectionExercise {
  sectionId: string;
  titleKey: string;
  questions: Question[];
}
```

### 3.2. Contrat d'API (Endpoints)

*   **Récupérer le tableau de bord :**
    *   **Endpoint :** `GET /api/ai-course/dashboard`
    *   **Description :** Récupère toutes les données nécessaires pour afficher l'écran principal du cours.
    *   **Réponse (200 OK) :** Un objet de type `CourseDashboard`.

*   **Récupérer les exercices d'une section :**
    *   **Endpoint :** `GET /api/ai-course/section/{sectionId}/exercises`
    *   **Description :** Récupère la liste des questions pour une section donnée.
    *   **Réponse (200 OK) :** Un objet de type `SectionExercise`.

*   **Soumettre les réponses d'une section :**
    *   **Endpoint :** `POST /api/ai-course/section/{sectionId}/submit`
    *   **Description :** Envoie les réponses, sauvegarde la progression et renvoie la correction.
    *   **Réponse (200 OK) :** Un objet `feedback` comme défini précédemment.

### 3.3. Architecture Front-end

*   **Composants React :**
    *   `CourseDashboardScreen.tsx` : Nouvel écran principal affichant le tableau de bord.
    *   `ChapterCard.tsx` : Affiche un chapitre et la liste de ses sections.
    *   `ExerciseScreen.tsx` : Écran qui gère le déroulement d'un questionnaire pour une section (ancien `QuizContainer`).
    *   `QuestionRenderer.tsx` : Inchangé, affiche dynamiquement une question.
*   **Gestion de l'État (Redux) :**
    *   Un slice `courseDashboard.slice.ts` pour les données du tableau de bord.
    *   Un slice `exercise.slice.ts` pour gérer l'état d'un exercice en cours.

## Les question libres ne feront pas partie du questionnaire
![elementsofai screenshot](./elementsofai.png)
