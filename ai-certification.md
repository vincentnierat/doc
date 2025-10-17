# Spécifications Techniques : Cours de Certification IA

**Version : 4.0 (Draft)**
**Date : 16/10/2025**

## 1. Objectif de la fonctionnalité

L'objectif est de créer un **cours de certification sur l'IA** interactif et monétisé pour les candidats, disponible en niveaux **Base** et **Pro**. Le cours est présenté sous la forme d'un tableau de bord, structuré en chapitres et sections. Il intègre des exercices interactifs (QCM) et du contenu de lecture, et s'interface avec la gamification et le score d'employabilité du candidat.

## 2. Parcours Utilisateur

### 2.1. Accès et Écran de Sélection (Base/Pro)

1.  **Points d'Entrée :** L'utilisateur accède à la certification via la section "Amélioration du score d'employabilité" ou la section "Gamification".
2.  **Écran de Sélection :** L'utilisateur arrive sur un écran avec deux onglets : "Base" et "Pro".
3.  **Logique d'Accès :** En cliquant sur un niveau, le système vérifie le statut de l'utilisateur :
    *   **Cas 1 : Non acheté :** Redirection vers la page de paiement Stripe (en utilisant l'intégration existante).
    *   **Cas 2 : Acheté et terminé :** Accès direct au tableau de bord du cours pour consultation.
    *   **Cas 3 : Acheté mais non terminé :** Accès au tableau de bord du cours pour continuer.

### 2.2. Tableau de Bord du Cours

1.  **En-tête :** Affiche les statistiques globales : "Progression totale" en % et "Exercices complétés" (ex: 20/25).
2.  **Grille des Chapitres :**
    *   Présente les chapitres sous forme de cartes avec illustration et titre.
    *   Chaque carte liste les sections du chapitre avec leur progression (ex: 1/2).
    *   **Règle d'affichage :** Pour un cours inachevé, le chapitre en cours est affiché en premier.

### 2.3. Déroulement d'un Chapitre

*   **Type `exercise` :** L'utilisateur clique sur une section, accède à l'écran d'exercices, répond aux questions et obtient une correction immédiate.
*   **Type `readonly` (Chapitre 6) :** L'utilisateur accède à un écran de lecture et doit faire défiler le contenu jusqu'en bas pour valider le chapitre.

### 2.4. Post-Certification

1.  **Validation Finale :** La validation du chapitre 6 déclenche la fin du cours.
2.  **Indicateurs de Réussite :** Si le score est suffisant, une icône "check" et la date d'obtention apparaissent sur le bouton d'accès et le profil public. L'utilisateur accède à son certificat.

## 3. Architecture Technique

### 3.1. Modèles de Données (Typescript)

```typescript
// types/aiCourse.ts

export type PurchaseStatus = 'not_bought' | 'in_progress' | 'completed';
export type ChapterType = 'exercise' | 'readonly';
// Le type 'text' est volontairement omis suite à la nouvelle contrainte.
export type QuestionType = 'radio' | 'checkbox' | 'number';

export interface CourseDashboard {
  purchaseStatus: PurchaseStatus;
  completionDate?: string;
  stats: { completionPercentage: number; completedExercises: number; totalExercises: number; };
  chapters: Chapter[];
}

export interface Chapter {
  chapterId: string;
  titleKey: string;
  illustrationUrl: string;
  type: ChapterType;
  sections: SectionProgress[];
}

export interface SectionProgress {
  sectionId: string;
  titleKey: string;
  completedExercises: number;
  totalExercises: number;
}

export interface SectionContent {
  sectionId: string;
  titleKey: string;
  // Le contenu peut être un exercice ou de la lecture
  content: SectionExercise | ReadonlyContent;
}

export interface SectionExercise {
  questions: Question[];
}

export interface ReadonlyContent {
  body: string; // Contenu au format Markdown ou HTML
}

export interface Question { /* ... */ }
```

### 3.2. Contrat d'API (Endpoints)

*   **`GET /api/ai-course/dashboard/{level}` :** Renvoie l'objet `CourseDashboard`.
*   **Paiement :** Utilisation de l'endpoint de paiement Stripe existant en passant un identifiant de produit mis à jour (ex: `ai_course_base`).
*   **`GET /api/ai-course/section/{sectionId}/content` :** Renvoie l'objet `SectionContent` contenant soit les exercices, soit le contenu `readonly`.
*   **`POST /api/ai-course/section/{sectionId}/submit` :** Pour les sections de type `exercise`.
*   **`POST /api/ai-course/section/{sectionId}/validate-readonly` :** Pour les sections de type `readonly`.

### 3.3. Architecture Front-end

*   **Composants React :** `CourseSelectionScreen`, `CourseDashboardScreen`, `ExerciseScreen`, `ReadOnlyScreen`, `ChapterCard`, `QuestionRenderer`.
*   **Gestion de l'État (Redux) :** Un slice `course.slice.ts` gère l'état global du cours, incluant le statut d'achat.

### 3.4. Gestion des Traductions (i18n)

La stratégie s'appuie sur le système existant (clés de traduction fournies par l'API et résolues via les fichiers JSON locaux).

#### 3.4.1. Exemple de Structure dans les Fichiers JSON

```json
{
  "course_ai": {
    "dashboard_title": "Bienvenue au cours !",
    "chapters": {
      "chapter1": {
        "title": "Qu'est-ce que l'IA ?",
        "sections": {
          "section1_1": {
            "title": "Comment définir l'IA ?"
          }
        }
      }
    }
  }
}
```

## 4. Contraintes et Références

*   **Contrainte :** Les questions de type "texte libre" (`text`) ne feront pas partie du questionnaire.
*   **Référence Visuelle :**
    ![elementsofai screenshot](./elementsofai.png)
