# Elena Nutrition Coaching Hub – Quellcode

React 19 + TypeScript + Vite, Tailwind CSS 4, shadcn/ui, Recharts, Zustand.

Die Dateien sind relativ zum Ordner `src/` des App-Projekts angegeben. Fertige UI-Bausteine (`@/components/ui/*`, `@/components/pdf-export-button`, `@/components/states`, `@/hooks/use-confirm`, `@/lib/utils`) stammen aus der Standard-App-Vorlage und sind hier nicht enthalten.

## Inhalt

1. Datenmodell – `src/lib/types.ts`
2. Beispieldaten – `src/lib/seed.ts`
3. Datenspeicher (automatisches Speichern) – `src/lib/store.ts`
4. Dateiablage im Browser – `src/lib/file-store.ts`
5. Berechnungen & Formatierung – `src/lib/calc.ts`
6. Routen / Navigation – `src/routes.tsx`
7. Layout (Seitenleiste, mobile Navigation, Rollenumschalter) – `src/components/app-shell.tsx`
8. Diagramme & Fortschrittsring – `src/components/charts.tsx`
9. Lebensmittel-Verwaltung – `src/components/food-manager.tsx`
10. Seite: Dashboard – `src/pages/home.tsx`
11. Seite: Ernährungsplan – `src/pages/nutrition.tsx`
12. Seite: Check-in – `src/pages/check-in.tsx`
13. Seite: Fortschritt – `src/pages/progress.tsx`
14. Seite: Training – `src/pages/training.tsx`
15. Seite: Dokumente – `src/pages/documents.tsx`
16. Seite: Coach-Notizen – `src/pages/coach-notes.tsx`
17. Design-Farben (Theme) – `src/index.css`

## Datenmodell

`src/lib/types.ts`

````ts
// Datenmodell des Nutrition Coaching Hubs.
// Alle Bereiche teilen sich EIN Datenobjekt (HubData), das Coach und Klientin
// gemeinsam sehen und bearbeiten. Nur `coachNotes` ist rollenbeschränkt.

export type Role = "coach" | "klientin";

export interface ClientProfile {
  name: string;
  startDate: string; // ISO yyyy-mm-dd
  startWeight: number; // kg
  goalWeight: number; // kg
  heightCm: number;
  coachName: string;
}

export interface NutritionTargets {
  kcalMin: number;
  kcalMax: number;
  proteinMin: number; // g
  proteinMax: number; // g
  fatMin: number; // g
  fatMax: number; // g
  stepGoal: number; // Schritte / Tag
  workoutsPerWeek: number;
}

export type FoodCategory =
  | "protein"
  | "kohlenhydrate"
  | "fett"
  | "gemuese"
  | "obst"
  | "milchprodukt";

export interface Food {
  id: string;
  name: string;
  category: FoodCategory;
  // Nährwerte je 100 g
  kcal: number;
  protein: number;
  carbs: number;
  fat: number;
  /** Selbst angelegtes Lebensmittel (bearbeit- und löschbar) */
  custom?: boolean;
}

export type MealSlot = "fruehstueck" | "mittagessen" | "snack" | "abendessen";

export interface MealItem {
  id: string;
  foodId: string;
  grams: number;
}

export interface Meal {
  slot: MealSlot;
  title: string;
  items: MealItem[];
}

export interface CheckIn {
  id: string;
  date: string;
  weight: number;
  hip: number; // cm
  waist: number; // cm
  energy: number; // 1–10
  hunger: number; // 1–10
  sleep: number; // 1–10
  feedback: string;
  author: Role;
}

export interface DailyLog {
  date: string;
  steps: number;
}

export interface Exercise {
  id: string;
  name: string;
  sets: number;
  reps: string; // z. B. "8–10"
  weight: number; // kg
}

export interface WorkoutPlan {
  id: "A" | "B";
  name: string;
  focus: string;
  exercises: Exercise[];
}

export interface WorkoutLogEntry {
  exerciseId: string;
  name: string;
  sets: number;
  reps: string;
  weight: number;
}

export interface WorkoutLog {
  id: string;
  date: string;
  planId: "A" | "B";
  entries: WorkoutLogEntry[];
  note: string;
}

export type DocCategory =
  | "ernaehrungsplan"
  | "einkaufsliste"
  | "rezept"
  | "pdf"
  | "fortschrittsbericht";

export interface HubDocument {
  id: string;
  title: string;
  category: DocCategory;
  date: string;
  addedBy: Role;
  description: string;
  fileName?: string;
  sizeKb?: number;
  mimeType?: string;
  /** Schlüssel der im Browser gespeicherten Datei */
  fileId?: string;
}

export type CoachNoteType = "anpassung" | "empfehlung" | "naechster-schritt";

export interface CoachNote {
  id: string;
  date: string;
  type: CoachNoteType;
  text: string;
  done: boolean;
}

export interface ProgressPhoto {
  id: string;
  date: string;
  label: "vorher" | "nachher";
  url: string; // blob:-URL, nur in der laufenden Sitzung
}

export interface HubData {
  profile: ClientProfile;
  targets: NutritionTargets;
  foods: Food[];
  meals: Meal[];
  checkIns: CheckIn[];
  dailyLogs: DailyLog[];
  plans: WorkoutPlan[];
  workoutLogs: WorkoutLog[];
  documents: HubDocument[];
  coachNotes: CoachNote[];
}
````

## Beispieldaten

`src/lib/seed.ts`

````ts
import type {
  CheckIn,
  DailyLog,
  Food,
  HubData,
  WorkoutLog,
  WorkoutPlan,
} from "./types";

export const foods: Food[] = [
  // Protein
  { id: "skyr", name: "Skyr natur", category: "milchprodukt", kcal: 63, protein: 11, carbs: 4, fat: 0.2 },
  { id: "magerquark", name: "Magerquark", category: "milchprodukt", kcal: 67, protein: 12, carbs: 4, fat: 0.3 },
  { id: "griech-joghurt", name: "Griechischer Joghurt 2 %", category: "milchprodukt", kcal: 73, protein: 10, carbs: 3.6, fat: 2 },
  { id: "huettenkaese", name: "Hüttenkäse light", category: "milchprodukt", kcal: 74, protein: 12.5, carbs: 2.6, fat: 1.5 },
  { id: "haehnchen", name: "Hähnchenbrust", category: "protein", kcal: 105, protein: 23, carbs: 0, fat: 1.2 },
  { id: "pute", name: "Putenbrust", category: "protein", kcal: 107, protein: 24, carbs: 0, fat: 1 },
  { id: "lachs", name: "Lachsfilet", category: "protein", kcal: 197, protein: 20, carbs: 0, fat: 13 },
  { id: "kabeljau", name: "Kabeljau", category: "protein", kcal: 77, protein: 17.7, carbs: 0, fat: 0.6 },
  { id: "tofu", name: "Tofu natur", category: "protein", kcal: 122, protein: 13, carbs: 1.5, fat: 7 },
  { id: "eier", name: "Eier (Größe M)", category: "protein", kcal: 137, protein: 12, carbs: 1.5, fat: 9.3 },
  { id: "rinderhack", name: "Rinderhack 5 %", category: "protein", kcal: 121, protein: 21, carbs: 0, fat: 5 },
  // Kohlenhydrate
  { id: "haferflocken", name: "Haferflocken", category: "kohlenhydrate", kcal: 372, protein: 13.5, carbs: 58.7, fat: 7 },
  { id: "vollkornbrot", name: "Vollkornbrot", category: "kohlenhydrate", kcal: 214, protein: 7.5, carbs: 38, fat: 1.6 },
  { id: "reis", name: "Basmatireis (gekocht)", category: "kohlenhydrate", kcal: 130, protein: 2.7, carbs: 28, fat: 0.3 },
  { id: "kartoffeln", name: "Kartoffeln", category: "kohlenhydrate", kcal: 77, protein: 2, carbs: 17, fat: 0.1 },
  { id: "suesskartoffel", name: "Süßkartoffel", category: "kohlenhydrate", kcal: 86, protein: 1.6, carbs: 20, fat: 0.1 },
  { id: "vk-nudeln", name: "Vollkornnudeln (gekocht)", category: "kohlenhydrate", kcal: 149, protein: 6, carbs: 28, fat: 1.1 },
  { id: "quinoa", name: "Quinoa (gekocht)", category: "kohlenhydrate", kcal: 120, protein: 4.4, carbs: 21.3, fat: 1.9 },
  // Fett
  { id: "mandeln", name: "Mandeln", category: "fett", kcal: 579, protein: 21, carbs: 9, fat: 50 },
  { id: "walnuesse", name: "Walnüsse", category: "fett", kcal: 654, protein: 15, carbs: 7, fat: 65 },
  { id: "olivenoel", name: "Olivenöl", category: "fett", kcal: 884, protein: 0, carbs: 0, fat: 100 },
  { id: "avocado", name: "Avocado", category: "fett", kcal: 160, protein: 2, carbs: 2, fat: 15 },
  { id: "erdnussmus", name: "Erdnussmus", category: "fett", kcal: 598, protein: 25, carbs: 13, fat: 49 },
  // Gemüse
  { id: "brokkoli", name: "Brokkoli", category: "gemuese", kcal: 34, protein: 2.8, carbs: 4, fat: 0.4 },
  { id: "paprika", name: "Paprika", category: "gemuese", kcal: 31, protein: 1, carbs: 6, fat: 0.3 },
  { id: "zucchini", name: "Zucchini", category: "gemuese", kcal: 17, protein: 1.2, carbs: 2.2, fat: 0.3 },
  { id: "spinat", name: "Blattspinat", category: "gemuese", kcal: 23, protein: 2.9, carbs: 1.4, fat: 0.4 },
  { id: "gurke", name: "Gurke", category: "gemuese", kcal: 15, protein: 0.7, carbs: 2.2, fat: 0.1 },
  // Obst
  { id: "beeren", name: "Beerenmix (TK)", category: "obst", kcal: 45, protein: 1, carbs: 8, fat: 0.4 },
  { id: "apfel", name: "Apfel", category: "obst", kcal: 52, protein: 0.3, carbs: 12, fat: 0.2 },
  { id: "banane", name: "Banane", category: "obst", kcal: 89, protein: 1.1, carbs: 20, fat: 0.3 },
];

const checkInRaw: [string, number, number, number, number, number, number, string][] = [
  ["2026-07-07", 74.6, 104.0, 86.0, 6, 6, 6, "Startwoche – Plan gut verständlich, Frühstück noch ungewohnt."],
  ["2026-07-14", 74.1, 103.8, 85.6, 6, 7, 6, "Abends etwas Hunger, Snack nach vorne gezogen."],
  ["2026-07-21", 73.9, 103.5, 85.1, 7, 6, 7, "Training A fühlt sich gut an."],
  ["2026-07-28", 73.4, 103.1, 84.6, 7, 5, 7, ""],
  ["2026-08-04", 73.5, 103.0, 84.4, 6, 6, 5, "Wenig Schlaf wegen Projektabgabe."],
  ["2026-08-11", 72.9, 102.6, 83.8, 7, 5, 7, "Rezepte aus der Liste nachgekocht – sehr lecker."],
  ["2026-08-18", 72.6, 102.2, 83.4, 8, 5, 7, ""],
  ["2026-08-25", 72.2, 101.9, 83.0, 8, 4, 8, "Urlaubswoche, trotzdem Schritte geschafft."],
  ["2026-09-01", 72.3, 101.8, 82.7, 7, 5, 7, "Nach Urlaub etwas Wasser."],
  ["2026-09-08", 71.6, 101.3, 82.2, 8, 4, 8, ""],
  ["2026-09-15", 71.3, 101.0, 81.8, 8, 5, 8, "Kniebeugen auf 40 kg gesteigert."],
  ["2026-09-22", 70.9, 100.7, 81.4, 7, 5, 7, "Hunger am Wochenende höher."],
  ["2026-09-29", 70.6, 100.4, 81.1, 8, 4, 8, "Energie top, Hose sitzt lockerer."],
];

const checkIns: CheckIn[] = checkInRaw.map(
  ([date, weight, hip, waist, energy, hunger, sleep, feedback], i) => ({
    id: `ci-${i + 1}`,
    date,
    weight,
    hip,
    waist,
    energy,
    hunger,
    sleep,
    feedback,
    author: "klientin",
  }),
);

const dailyLogs: DailyLog[] = [
  { date: "2026-09-30", steps: 9412 },
  { date: "2026-10-01", steps: 7835 },
  { date: "2026-10-02", steps: 10764 },
  { date: "2026-10-03", steps: 12108 },
  { date: "2026-10-04", steps: 6390 },
  { date: "2026-10-05", steps: 8957 },
  { date: "2026-10-06", steps: 4281 },
];

const plans: WorkoutPlan[] = [
  {
    id: "A",
    name: "Ganzkörper A",
    focus: "Kniebeuge & Drücken",
    exercises: [
      { id: "a1", name: "Kniebeuge (Langhantel)", sets: 3, reps: "8–10", weight: 40 },
      { id: "a2", name: "Bankdrücken (Kurzhantel)", sets: 3, reps: "8–10", weight: 10 },
      { id: "a3", name: "Rudern am Kabelzug", sets: 3, reps: "10–12", weight: 32.5 },
      { id: "a4", name: "Ausfallschritte", sets: 3, reps: "10 je Bein", weight: 8 },
      { id: "a5", name: "Plank", sets: 3, reps: "40 s", weight: 0 },
    ],
  },
  {
    id: "B",
    name: "Ganzkörper B",
    focus: "Hüfte & Ziehen",
    exercises: [
      { id: "b1", name: "Rumänisches Kreuzheben", sets: 3, reps: "8–10", weight: 35 },
      { id: "b2", name: "Latzug", sets: 3, reps: "10–12", weight: 30 },
      { id: "b3", name: "Schulterdrücken (Kurzhantel)", sets: 3, reps: "10", weight: 7 },
      { id: "b4", name: "Hip Thrust", sets: 3, reps: "10–12", weight: 50 },
      { id: "b5", name: "Dead Bug", sets: 3, reps: "12", weight: 0 },
    ],
  },
];

function logFrom(
  id: string,
  date: string,
  plan: WorkoutPlan,
  delta: number,
  note: string,
): WorkoutLog {
  return {
    id,
    date,
    planId: plan.id,
    note,
    entries: plan.exercises.map((e) => ({
      exerciseId: e.id,
      name: e.name,
      sets: e.sets,
      reps: e.reps,
      weight: e.weight > 0 ? Math.max(0, e.weight - delta) : 0,
    })),
  };
}

const workoutLogs: WorkoutLog[] = [
  logFrom("w-1", "2026-08-24", plans[0], 7.5, ""),
  logFrom("w-2", "2026-08-27", plans[1], 7.5, "Hotel-Gym, leichtere Gewichte."),
  logFrom("w-3", "2026-08-31", plans[0], 5, ""),
  logFrom("w-4", "2026-09-03", plans[1], 5, ""),
  logFrom("w-5", "2026-09-07", plans[0], 5, ""),
  logFrom("w-6", "2026-09-10", plans[1], 2.5, ""),
  logFrom("w-7", "2026-09-14", plans[0], 2.5, "Kniebeuge sauber, nächstes Mal steigern."),
  logFrom("w-8", "2026-09-17", plans[1], 2.5, ""),
  logFrom("w-9", "2026-09-21", plans[0], 0, ""),
  logFrom("w-10", "2026-09-24", plans[1], 0, ""),
  logFrom("w-11", "2026-09-28", plans[0], 0, ""),
  logFrom("w-12", "2026-10-01", plans[1], 0, "Hip Thrust fühlt sich stark an."),
  logFrom("w-13", "2026-10-05", plans[0], 0, ""),
];

export const seedData: HubData = {
  profile: {
    name: "Elena",
    startDate: "2026-07-07",
    startWeight: 74.6,
    goalWeight: 66,
    heightCm: 168,
    coachName: "Coach",
  },
  targets: {
    kcalMin: 1650,
    kcalMax: 1700,
    proteinMin: 110,
    proteinMax: 120,
    fatMin: 50,
    fatMax: 60,
    stepGoal: 9000,
    workoutsPerWeek: 2,
  },
  foods,
  meals: [
    {
      slot: "fruehstueck",
      title: "Frühstück",
      items: [
        { id: "m1", foodId: "skyr", grams: 250 },
        { id: "m2", foodId: "haferflocken", grams: 40 },
        { id: "m3", foodId: "beeren", grams: 120 },
        { id: "m4", foodId: "walnuesse", grams: 10 },
      ],
    },
    {
      slot: "mittagessen",
      title: "Mittagessen",
      items: [
        { id: "m5", foodId: "haehnchen", grams: 150 },
        { id: "m6", foodId: "reis", grams: 160 },
        { id: "m7", foodId: "brokkoli", grams: 200 },
        { id: "m8", foodId: "olivenoel", grams: 10 },
      ],
    },
    {
      slot: "snack",
      title: "Snack",
      items: [
        { id: "m9", foodId: "huettenkaese", grams: 150 },
        { id: "m10", foodId: "apfel", grams: 150 },
        { id: "m11", foodId: "mandeln", grams: 15 },
      ],
    },
    {
      slot: "abendessen",
      title: "Abendessen",
      items: [
        { id: "m12", foodId: "lachs", grams: 120 },
        { id: "m13", foodId: "kartoffeln", grams: 200 },
        { id: "m14", foodId: "zucchini", grams: 200 },
        { id: "m15", foodId: "spinat", grams: 80 },
      ],
    },
  ],
  checkIns,
  dailyLogs,
  plans,
  workoutLogs,
  documents: [
    { id: "d1", title: "Ernährungsplan Phase 2", category: "ernaehrungsplan", date: "2026-09-01", addedBy: "coach", description: "1.650–1.700 kcal, 110–120 g Protein, inkl. Tauschliste.", fileName: "Ernaehrungsplan_Phase2.pdf", sizeKb: 412 },
    { id: "d2", title: "Ernährungsplan Phase 1", category: "ernaehrungsplan", date: "2026-07-07", addedBy: "coach", description: "Startplan mit 1.750 kcal.", fileName: "Ernaehrungsplan_Phase1.pdf", sizeKb: 388 },
    { id: "d3", title: "Einkaufsliste KW 41", category: "einkaufsliste", date: "2026-10-05", addedBy: "coach", description: "Basis für Frühstück, Meal Prep Mittag und 3 Abendessen.", fileName: "Einkaufsliste_KW41.pdf", sizeKb: 96 },
    { id: "d4", title: "Einkaufsliste KW 40", category: "einkaufsliste", date: "2026-09-28", addedBy: "klientin", description: "Ergänzt um Tofu und Quinoa.", fileName: "Einkaufsliste_KW40.pdf", sizeKb: 91 },
    { id: "d5", title: "Protein-Overnight-Oats", category: "rezept", date: "2026-08-11", addedBy: "coach", description: "ca. 410 kcal · 34 g Protein", fileName: "Rezept_Overnight_Oats.pdf", sizeKb: 210 },
    { id: "d6", title: "Ofenlachs mit Kartoffeln", category: "rezept", date: "2026-08-18", addedBy: "coach", description: "ca. 520 kcal · 32 g Protein", fileName: "Rezept_Ofenlachs.pdf", sizeKb: 245 },
    { id: "d7", title: "Trainingsanleitung Ganzkörper A/B", category: "pdf", date: "2026-07-07", addedBy: "coach", description: "Technikhinweise und Progressionsregeln.", fileName: "Training_Ganzkoerper.pdf", sizeKb: 1340 },
    { id: "d8", title: "Fortschrittsbericht August", category: "fortschrittsbericht", date: "2026-09-01", addedBy: "coach", description: "−2,3 kg, −2,2 cm Hüfte, −3,3 cm Bauch seit Start.", fileName: "Fortschritt_August.pdf", sizeKb: 520 },
  ],
  coachNotes: [
    { id: "n1", date: "2026-09-29", type: "anpassung", text: "Kalorien bleiben bei 1.650–1.700 kcal. Tempo von ca. 0,35 kg/Woche ist ideal – keine Reduktion nötig.", done: false },
    { id: "n2", date: "2026-09-22", type: "empfehlung", text: "Am Wochenende höherer Hunger: 30 g Protein mehr am Samstagmittag (z. B. Skyr-Bowl), dafür Snack-Mandeln reduzieren.", done: false },
    { id: "n3", date: "2026-09-29", type: "naechster-schritt", text: "Kniebeuge auf 42,5 kg und Hip Thrust auf 55 kg steigern, wenn alle Sätze sauber.", done: false },
    { id: "n4", date: "2026-09-15", type: "naechster-schritt", text: "Neue Vorher-/Nachher-Fotos zum 3-Monats-Check am 06.10. anfragen.", done: false },
    { id: "n5", date: "2026-08-04", type: "empfehlung", text: "Schlafroutine: Bildschirm 30 min vor dem Schlafen aus, Magnesium optional.", done: true },
  ],
};
````

## Datenspeicher (automatisches Speichern)

`src/lib/store.ts`

````ts
import { create } from "zustand";
import { createJSONStorage, persist, type StateStorage } from "zustand/middleware";
import { seedData } from "./seed";
import type {
  CheckIn,
  ClientProfile,
  CoachNote,
  DailyLog,
  Food,
  Exercise,
  HubData,
  HubDocument,
  MealItem,
  MealSlot,
  NutritionTargets,
  ProgressPhoto,
  Role,
  WorkoutLog,
} from "./types";

// Automatisches Speichern: jede Änderung wird sofort im Browser gesichert.
// Ohne verfügbaren Browser-Speicher fällt der Hub auf die laufende Sitzung zurück.
const memory = new Map<string, string>();
const safeStorage: StateStorage = {
  getItem: (k) => {
    try {
      return window.localStorage.getItem(k);
    } catch {
      return memory.get(k) ?? null;
    }
  },
  setItem: (k, v) => {
    try {
      window.localStorage.setItem(k, v);
    } catch {
      memory.set(k, v);
    }
  },
  removeItem: (k) => {
    try {
      window.localStorage.removeItem(k);
    } catch {
      memory.delete(k);
    }
  },
};

const uid = () => Math.random().toString(36).slice(2, 10);

interface HubState extends HubData {
  role: Role;
  photos: ProgressPhoto[];
  lastSaved: number | null;
  setRole: (r: Role) => void;
  setTargets: (t: Partial<NutritionTargets>) => void;
  setProfile: (p: Partial<ClientProfile>) => void;
  addFood: (f: Omit<Food, "id" | "custom">) => string;
  updateFood: (id: string, patch: Partial<Omit<Food, "id">>) => void;
  removeFood: (id: string) => boolean;
  updateMealItem: (slot: MealSlot, itemId: string, patch: Partial<MealItem>) => void;
  addMealItem: (slot: MealSlot, foodId: string, grams: number) => void;
  removeMealItem: (slot: MealSlot, itemId: string) => void;
  addCheckIn: (c: Omit<CheckIn, "id">) => void;
  removeCheckIn: (id: string) => void;
  setSteps: (log: DailyLog) => void;
  updateExercise: (planId: "A" | "B", exId: string, patch: Partial<Exercise>) => void;
  addExercise: (planId: "A" | "B") => void;
  removeExercise: (planId: "A" | "B", exId: string) => void;
  logWorkout: (log: Omit<WorkoutLog, "id">) => void;
  removeWorkoutLog: (id: string) => void;
  addDocument: (d: Omit<HubDocument, "id">) => void;
  updateDocument: (id: string, patch: Partial<Omit<HubDocument, "id">>) => void;
  removeDocument: (id: string) => void;
  addCoachNote: (n: Omit<CoachNote, "id" | "done">) => void;
  toggleCoachNote: (id: string) => void;
  removeCoachNote: (id: string) => void;
  addPhoto: (p: Omit<ProgressPhoto, "id">) => void;
  removePhoto: (id: string) => void;
  resetAll: () => void;
}

const stamp = () => ({ lastSaved: Date.now() });

export const useHub = create<HubState>()(
  persist(
    (set, get) => ({
      ...seedData,
      role: "klientin",
      photos: [],
      lastSaved: null,
      setRole: (role) => set({ role }),
      setTargets: (t) => set((s) => ({ targets: { ...s.targets, ...t }, ...stamp() })),
      setProfile: (p) => set((s) => ({ profile: { ...s.profile, ...p }, ...stamp() })),
      addFood: (f) => {
        const id = `custom-${uid()}`;
        set((s) => ({ foods: [...s.foods, { ...f, id, custom: true }], ...stamp() }));
        return id;
      },
      updateFood: (id, patch) =>
        set((s) => ({ foods: s.foods.map((f) => (f.id === id ? { ...f, ...patch } : f)), ...stamp() })),
      removeFood: (id) => {
        const used = get().meals.some((m) => m.items.some((i) => i.foodId === id));
        if (used) return false;
        set((s) => ({ foods: s.foods.filter((f) => f.id !== id), ...stamp() }));
        return true;
      },
      updateMealItem: (slot, itemId, patch) =>
        set((s) => ({
          meals: s.meals.map((m) =>
            m.slot !== slot
              ? m
              : { ...m, items: m.items.map((i) => (i.id === itemId ? { ...i, ...patch } : i)) },
          ),
          ...stamp(),
        })),
      addMealItem: (slot, foodId, grams) =>
        set((s) => ({
          meals: s.meals.map((m) =>
            m.slot !== slot ? m : { ...m, items: [...m.items, { id: uid(), foodId, grams }] },
          ),
          ...stamp(),
        })),
      removeMealItem: (slot, itemId) =>
        set((s) => ({
          meals: s.meals.map((m) =>
            m.slot !== slot ? m : { ...m, items: m.items.filter((i) => i.id !== itemId) },
          ),
          ...stamp(),
        })),
      addCheckIn: (c) =>
        set((s) => ({
          checkIns: [...s.checkIns, { ...c, id: uid() }].sort((a, b) => a.date.localeCompare(b.date)),
          ...stamp(),
        })),
      removeCheckIn: (id) =>
        set((s) => ({ checkIns: s.checkIns.filter((c) => c.id !== id), ...stamp() })),
      setSteps: (log) =>
        set((s) => {
          const rest = s.dailyLogs.filter((d) => d.date !== log.date);
          return {
            dailyLogs: [...rest, log].sort((a, b) => a.date.localeCompare(b.date)),
            ...stamp(),
          };
        }),
      updateExercise: (planId, exId, patch) =>
        set((s) => ({
          plans: s.plans.map((p) =>
            p.id !== planId
              ? p
              : { ...p, exercises: p.exercises.map((e) => (e.id === exId ? { ...e, ...patch } : e)) },
          ),
          ...stamp(),
        })),
      addExercise: (planId) =>
        set((s) => ({
          plans: s.plans.map((p) =>
            p.id !== planId
              ? p
              : {
                  ...p,
                  exercises: [
                    ...p.exercises,
                    { id: uid(), name: "Neue Übung", sets: 3, reps: "10", weight: 0 },
                  ],
                },
          ),
          ...stamp(),
        })),
      removeExercise: (planId, exId) =>
        set((s) => ({
          plans: s.plans.map((p) =>
            p.id !== planId ? p : { ...p, exercises: p.exercises.filter((e) => e.id !== exId) },
          ),
          ...stamp(),
        })),
      logWorkout: (log) =>
        set((s) => ({
          workoutLogs: [...s.workoutLogs, { ...log, id: uid() }].sort((a, b) =>
            a.date.localeCompare(b.date),
          ),
          ...stamp(),
        })),
      removeWorkoutLog: (id) =>
        set((s) => ({ workoutLogs: s.workoutLogs.filter((w) => w.id !== id), ...stamp() })),
      addDocument: (d) =>
        set((s) => ({ documents: [{ ...d, id: uid() }, ...s.documents], ...stamp() })),
      updateDocument: (id, patch) =>
        set((s) => ({ documents: s.documents.map((d) => (d.id === id ? { ...d, ...patch } : d)), ...stamp() })),
      removeDocument: (id) =>
        set((s) => ({ documents: s.documents.filter((d) => d.id !== id), ...stamp() })),
      addCoachNote: (n) =>
        set((s) => ({ coachNotes: [{ ...n, id: uid(), done: false }, ...s.coachNotes], ...stamp() })),
      toggleCoachNote: (id) =>
        set((s) => ({
          coachNotes: s.coachNotes.map((n) => (n.id === id ? { ...n, done: !n.done } : n)),
          ...stamp(),
        })),
      removeCoachNote: (id) =>
        set((s) => ({ coachNotes: s.coachNotes.filter((n) => n.id !== id), ...stamp() })),
      addPhoto: (p) => set((s) => ({ photos: [...s.photos, { ...p, id: uid() }] })),
      removePhoto: (id) => set((s) => ({ photos: s.photos.filter((p) => p.id !== id) })),
      resetAll: () => set({ ...seedData, photos: [], ...stamp() }),
    }),
    {
      name: "elena-coaching-hub-v1",
      storage: createJSONStorage(() => safeStorage),
      // Dateien liegen separat im Browser-Dateispeicher; Fotos sind nur Sitzungs-Links.
      partialize: (s) => ({
        role: s.role,
        profile: s.profile,
        targets: s.targets,
        foods: s.foods,
        meals: s.meals,
        checkIns: s.checkIns,
        dailyLogs: s.dailyLogs,
        plans: s.plans,
        workoutLogs: s.workoutLogs,
        documents: s.documents,
        coachNotes: s.coachNotes,
        lastSaved: s.lastSaved,
      }),
    },
  ),
);
````

## Dateiablage im Browser

`src/lib/file-store.ts`

````ts
// Dauerhafte Ablage hochgeladener Dateien (PDFs, Bilder) im Browser (IndexedDB).
// Fällt still auf die laufende Sitzung zurück, wenn der Browser-Speicher
// nicht verfügbar ist (z. B. in der Vorschau).

const DB = "elena-coaching-files";
const STORE = "files";
const memory = new Map<string, Blob>();

function open(): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    try {
      const req = indexedDB.open(DB, 1);
      req.onupgradeneeded = () => req.result.createObjectStore(STORE);
      req.onsuccess = () => resolve(req.result);
      req.onerror = () => reject(req.error);
    } catch (e) {
      reject(e);
    }
  });
}

async function tx<T>(mode: IDBTransactionMode, fn: (s: IDBObjectStore) => IDBRequest<T>): Promise<T> {
  const db = await open();
  return new Promise<T>((resolve, reject) => {
    const t = db.transaction(STORE, mode);
    const r = fn(t.objectStore(STORE));
    r.onsuccess = () => resolve(r.result);
    r.onerror = () => reject(r.error);
  });
}

export async function saveFile(blob: Blob): Promise<string> {
  const id = `f-${Date.now().toString(36)}-${Math.random().toString(36).slice(2, 8)}`;
  memory.set(id, blob);
  try {
    await tx("readwrite", (s) => s.put(blob, id));
  } catch {
    /* nur Sitzung */
  }
  return id;
}

export async function loadFile(id: string): Promise<Blob | null> {
  const m = memory.get(id);
  if (m) return m;
  try {
    const b = await tx<Blob | undefined>("readonly", (s) => s.get(id) as IDBRequest<Blob | undefined>);
    if (b) memory.set(id, b);
    return b ?? null;
  } catch {
    return null;
  }
}

export async function deleteFile(id: string): Promise<void> {
  memory.delete(id);
  try {
    await tx("readwrite", (s) => s.delete(id));
  } catch {
    /* ignore */
  }
}
````

## Berechnungen & Formatierung

`src/lib/calc.ts`

````ts
import type { CheckIn, Food, FoodCategory, Meal, NutritionTargets, WorkoutLog } from "./types";

export const TODAY = "2026-10-06";

const de1 = new Intl.NumberFormat("de-DE", { maximumFractionDigits: 1, minimumFractionDigits: 1 });
const de0 = new Intl.NumberFormat("de-DE", { maximumFractionDigits: 0 });
export const fmt1 = (n: number) => de1.format(n);
export const fmt0 = (n: number) => de0.format(n);
export const signed1 = (n: number) => (n > 0 ? "+" : n < 0 ? "−" : "±") + fmt1(Math.abs(n));

export function fmtDate(iso: string, opts: Intl.DateTimeFormatOptions = { day: "2-digit", month: "2-digit" }) {
  return new Date(iso + "T12:00:00").toLocaleDateString("de-DE", opts);
}

/** Kohlenhydrate automatisch aus Kalorien, Protein und Fett (4/4/9 kcal pro g). */
export function carbTargets(t: NutritionTargets) {
  const kcalMid = (t.kcalMin + t.kcalMax) / 2;
  const pMid = (t.proteinMin + t.proteinMax) / 2;
  const fMid = (t.fatMin + t.fatMax) / 2;
  const mid = Math.max(0, (kcalMid - pMid * 4 - fMid * 9) / 4);
  const min = Math.max(0, (t.kcalMin - t.proteinMax * 4 - t.fatMax * 9) / 4);
  const max = Math.max(0, (t.kcalMax - t.proteinMin * 4 - t.fatMin * 9) / 4);
  return { min: Math.round(min), max: Math.round(max), mid: Math.round(mid), kcalMid, pMid, fMid };
}

export interface Macros {
  kcal: number;
  protein: number;
  carbs: number;
  fat: number;
}

export function itemMacros(food: Food | undefined, grams: number): Macros {
  if (!food) return { kcal: 0, protein: 0, carbs: 0, fat: 0 };
  const f = grams / 100;
  return { kcal: food.kcal * f, protein: food.protein * f, carbs: food.carbs * f, fat: food.fat * f };
}

export function sumMacros(list: Macros[]): Macros {
  return list.reduce(
    (a, b) => ({ kcal: a.kcal + b.kcal, protein: a.protein + b.protein, carbs: a.carbs + b.carbs, fat: a.fat + b.fat }),
    { kcal: 0, protein: 0, carbs: 0, fat: 0 },
  );
}

export function mealMacros(meal: Meal, foods: Food[]): Macros {
  return sumMacros(meal.items.map((i) => itemMacros(foods.find((f) => f.id === i.foodId), i.grams)));
}

/** Menge des Ersatzlebensmittels mit gleichen Kalorien, auf 5 g gerundet. */
export function equivalentGrams(from: Food, grams: number, to: Food) {
  const kcal = (from.kcal * grams) / 100;
  const g = to.kcal > 0 ? (kcal / to.kcal) * 100 : grams;
  return Math.max(5, Math.round(g / 5) * 5);
}

export function sortedCheckIns(c: CheckIn[]) {
  return [...c].sort((a, b) => a.date.localeCompare(b.date));
}

/** Montag der Woche (ISO) für ein Datum. */
export function weekStart(iso: string) {
  const d = new Date(iso + "T12:00:00");
  const day = (d.getDay() + 6) % 7;
  d.setDate(d.getDate() - day);
  return d.toISOString().slice(0, 10);
}

export function addDays(iso: string, n: number) {
  const d = new Date(iso + "T12:00:00");
  d.setDate(d.getDate() + n);
  return d.toISOString().slice(0, 10);
}

export function workoutsThisWeek(logs: WorkoutLog[], today = TODAY) {
  const ws = weekStart(today);
  const we = addDays(ws, 6);
  return logs.filter((l) => l.date >= ws && l.date <= we);
}

export const catLabel: Record<FoodCategory, string> = {
  protein: "Protein",
  kohlenhydrate: "Kohlenhydrate",
  fett: "Fett",
  gemuese: "Gemüse",
  obst: "Obst",
  milchprodukt: "Milchprodukt",
};
````

## Routen / Navigation

`src/routes.tsx`

````tsx
import type { ReactElement } from "react";
import {
  LayoutDashboard,
  Salad,
  ClipboardCheck,
  TrendingDown,
  Dumbbell,
  FolderOpen,
  NotebookPen,
  type LucideIcon,
} from "lucide-react";
import { HomePage } from "@/pages/home";
import { NutritionPage } from "@/pages/nutrition";
import { CheckInPage } from "@/pages/check-in";
import { ProgressPage } from "@/pages/progress";
import { TrainingPage } from "@/pages/training";
import { DocumentsPage } from "@/pages/documents";
import { CoachNotesPage } from "@/pages/coach-notes";

export interface AppRoute {
  path: string;
  element: ReactElement;
  label?: string;
  short?: string;
  icon?: LucideIcon;
  coachOnly?: boolean;
}

// Single source of truth for routes + navigation.
export const routes: AppRoute[] = [
  { path: "/", element: <HomePage />, label: "Dashboard", short: "Home", icon: LayoutDashboard },
  { path: "/ernaehrung", element: <NutritionPage />, label: "Ernährungsplan", short: "Ernährung", icon: Salad },
  { path: "/check-in", element: <CheckInPage />, label: "Check-in", short: "Check-in", icon: ClipboardCheck },
  { path: "/fortschritt", element: <ProgressPage />, label: "Fortschritt", short: "Fortschritt", icon: TrendingDown },
  { path: "/training", element: <TrainingPage />, label: "Training", short: "Training", icon: Dumbbell },
  { path: "/dokumente", element: <DocumentsPage />, label: "Dokumente", short: "Dokumente", icon: FolderOpen },
  { path: "/coach", element: <CoachNotesPage />, label: "Coach-Notizen", short: "Notizen", icon: NotebookPen, coachOnly: true },
];
````

## Layout (Seitenleiste, mobile Navigation, Rollenumschalter)

`src/components/app-shell.tsx`

````tsx
import { NavLink, Outlet, useLocation } from "react-router-dom";
import { motion } from "motion/react";
import { CheckCircle2 } from "lucide-react";
import { Toaster } from "@/components/ui/sonner";
import { routes } from "@/routes";
import { useHub } from "@/lib/store";
import { cn } from "@/lib/utils";
import type { Role } from "@/lib/types";

function RoleSwitch({ className }: { className?: string }) {
  const role = useHub((s) => s.role);
  const setRole = useHub((s) => s.setRole);
  const opts: { v: Role; l: string }[] = [
    { v: "klientin", l: "Elena" },
    { v: "coach", l: "Coach" },
  ];
  return (
    <div role="group" aria-label="Ansicht wählen" className={cn("inline-flex rounded-full bg-white/10 p-1", className)}>
      {opts.map((o) => (
        <button
          key={o.v}
          type="button"
          aria-pressed={role === o.v}
          onClick={() => setRole(o.v)}
          className={cn(
            "rounded-full px-3 py-1 text-xs font-semibold transition-colors focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-emerald-300",
            role === o.v ? "bg-emerald-300 text-blue-950" : "text-blue-100 hover:text-white",
          )}
        >
          {o.l}
        </button>
      ))}
    </div>
  );
}

function SavedIndicator({ className }: { className?: string }) {
  const lastSaved = useHub((s) => s.lastSaved);
  const text = lastSaved
    ? `Gespeichert ${new Date(lastSaved).toLocaleTimeString("de-DE", { hour: "2-digit", minute: "2-digit" })}`
    : "Automatisches Speichern aktiv";
  return (
    <p className={cn("flex items-center gap-1.5 text-xs text-blue-100", className)} aria-live="polite">
      <CheckCircle2 className="size-3.5 text-emerald-300" aria-hidden="true" />
      {text}
    </p>
  );
}

export function AppShell() {
  const { pathname } = useLocation();
  const role = useHub((s) => s.role);
  const nav = routes.filter((r) => r.label && (!r.coachOnly || role === "coach"));

  return (
    <div className="flex min-h-screen bg-background">
      <a
        href="#inhalt"
        className="sr-only focus:not-sr-only focus:fixed focus:left-3 focus:top-3 focus:z-50 focus:rounded-full focus:bg-primary focus:px-4 focus:py-2 focus:text-primary-foreground"
      >
        Zum Inhalt springen
      </a>

      {/* Desktop sidebar */}
      <aside className="sticky top-0 hidden h-screen w-64 shrink-0 flex-col bg-blue-950 px-5 py-6 text-white lg:flex">
        <div className="flex items-center gap-3">
          <div className="grid size-10 place-items-center rounded-2xl bg-emerald-300 text-lg font-bold text-blue-950" aria-hidden="true">
            E
          </div>
          <div>
            <p className="text-sm font-bold leading-tight">Elena</p>
            <p className="text-xs text-blue-100">Nutrition Coaching</p>
          </div>
        </div>
        <RoleSwitch className="mt-6 self-start" />
        <nav aria-label="Hauptnavigation" className="mt-8 flex flex-col gap-1">
          {nav.map((r) => {
            const Icon = r.icon!;
            return (
              <NavLink
                key={r.path}
                to={r.path}
                end={r.path === "/"}
                className={({ isActive }) =>
                  cn(
                    "flex items-center gap-3 rounded-2xl px-3 py-2.5 text-sm font-medium transition-colors focus-visible:outline-2 focus-visible:outline-emerald-300",
                    isActive ? "bg-white text-blue-950" : "text-blue-100 hover:bg-white/10 hover:text-white",
                  )
                }
              >
                <Icon className="size-4.5" aria-hidden="true" />
                {r.label}
              </NavLink>
            );
          })}
        </nav>
        <SavedIndicator className="mt-auto" />
      </aside>

      <div className="flex min-w-0 flex-1 flex-col">
        {/* Mobile top bar */}
        <header className="sticky top-0 z-30 flex items-center justify-between gap-3 bg-blue-950 px-4 py-3 text-white lg:hidden">
          <div className="flex items-center gap-2">
            <div className="grid size-8 place-items-center rounded-xl bg-emerald-300 text-sm font-bold text-blue-950" aria-hidden="true">
              E
            </div>
            <span className="text-sm font-bold">Elena · Coaching</span>
          </div>
          <RoleSwitch />
        </header>

        <main id="inhalt" className="flex flex-1 flex-col pb-24 lg:pb-0">
          <motion.div
            key={pathname}
            initial={{ opacity: 0, y: 6 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ duration: 0.15, ease: "easeOut" }}
            className="mx-auto w-full max-w-6xl flex-1 px-4 py-6 sm:px-6 lg:px-10 lg:py-10"
          >
            <Outlet />
          </motion.div>
        </main>

        {/* Mobile bottom nav */}
        <nav
          aria-label="Hauptnavigation mobil"
          className="fixed inset-x-0 bottom-0 z-30 border-t border-border bg-card/95 backdrop-blur lg:hidden"
        >
          <ul className="no-scrollbar flex overflow-x-auto px-2 py-1.5">
            {nav.map((r) => {
              const Icon = r.icon!;
              return (
                <li key={r.path} className="min-w-[4.5rem] flex-1">
                  <NavLink
                    to={r.path}
                    end={r.path === "/"}
                    className={({ isActive }) =>
                      cn(
                        "flex flex-col items-center gap-0.5 rounded-xl px-1 py-1.5 text-[11px] font-medium focus-visible:outline-2 focus-visible:outline-ring",
                        isActive ? "text-primary" : "text-muted-foreground",
                      )
                    }
                  >
                    {({ isActive }) => (
                      <>
                        <span className={cn("grid h-7 w-12 place-items-center rounded-full", isActive && "bg-emerald-100")}>
                          <Icon className="size-4.5" aria-hidden="true" />
                        </span>
                        {r.short}
                      </>
                    )}
                  </NavLink>
                </li>
              );
            })}
          </ul>
        </nav>
      </div>
      <Toaster />
    </div>
  );
}

export default AppShell;
````

## Diagramme & Fortschrittsring

`src/components/charts.tsx`

````tsx
import { Area, AreaChart, CartesianGrid, Line, LineChart, ReferenceLine, XAxis, YAxis } from "recharts";
import { ChartContainer, ChartTooltip, ChartTooltipContent, type ChartConfig } from "@/components/ui/chart";
import { fmt1, fmtDate } from "@/lib/calc";
import type { CheckIn } from "@/lib/types";

const weightConfig = {
  weight: { label: "Gewicht (kg)", color: "var(--color-primary)" },
} satisfies ChartConfig;

export function WeightChart({
  checkIns,
  goal,
  className = "h-64 w-full",
}: {
  checkIns: CheckIn[];
  goal?: number;
  className?: string;
}) {
  const data = checkIns.map((c) => ({ date: fmtDate(c.date), weight: c.weight }));
  const vals = checkIns.map((c) => c.weight);
  const min = Math.floor(Math.min(...vals, goal ?? Infinity) - 0.5);
  const max = Math.ceil(Math.max(...vals) + 0.5);
  return (
    <ChartContainer config={weightConfig} className={className}>
      <AreaChart data={data} margin={{ left: 0, right: 8, top: 8, bottom: 0 }}>
        <defs>
          <linearGradient id="wfill" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stopColor="var(--color-weight)" stopOpacity={0.35} />
            <stop offset="100%" stopColor="#6ee7b7" stopOpacity={0.05} />
          </linearGradient>
        </defs>
        <CartesianGrid vertical={false} strokeDasharray="3 3" />
        <XAxis dataKey="date" tickLine={false} axisLine={false} tickMargin={8} minTickGap={16} />
        <YAxis domain={[min, max]} tickLine={false} axisLine={false} width={36} tickFormatter={(v: number) => fmt1(v)} />
        {goal !== undefined && (
          <ReferenceLine y={goal} stroke="#059669" strokeDasharray="6 4" label={{ value: `Ziel ${fmt1(goal)} kg`, position: "insideBottomRight", fill: "#047857", fontSize: 11 }} />
        )}
        <ChartTooltip isAnimationActive={false} content={<ChartTooltipContent />} />
        <Area isAnimationActive={false} dataKey="weight" type="monotone" stroke="var(--color-weight)" strokeWidth={2.5} fill="url(#wfill)" dot={{ r: 3 }} activeDot={{ r: 5 }} />
      </AreaChart>
    </ChartContainer>
  );
}

const circConfig = {
  hip: { label: "Hüfte (cm)", color: "var(--color-primary)" },
  waist: { label: "Bauch (cm)", color: "#059669" },
} satisfies ChartConfig;

export function CircumferenceChart({ checkIns, className = "h-64 w-full" }: { checkIns: CheckIn[]; className?: string }) {
  const data = checkIns.map((c) => ({ date: fmtDate(c.date), hip: c.hip, waist: c.waist }));
  return (
    <ChartContainer config={circConfig} className={className}>
      <LineChart data={data} margin={{ left: 0, right: 8, top: 8, bottom: 0 }}>
        <CartesianGrid vertical={false} strokeDasharray="3 3" />
        <XAxis dataKey="date" tickLine={false} axisLine={false} tickMargin={8} minTickGap={16} />
        <YAxis domain={["dataMin - 2", "dataMax + 2"]} tickLine={false} axisLine={false} width={36} />
        <ChartTooltip isAnimationActive={false} content={<ChartTooltipContent />} />
        <Line isAnimationActive={false} dataKey="hip" type="monotone" stroke="var(--color-hip)" strokeWidth={2.5} dot={{ r: 3 }} />
        <Line isAnimationActive={false} dataKey="waist" type="monotone" stroke="var(--color-waist)" strokeWidth={2.5} strokeDasharray="5 3" dot={{ r: 3 }} />
      </LineChart>
    </ChartContainer>
  );
}

/** Kreisförmige Fortschrittsanzeige (0–1). */
export function ProgressRing({
  value,
  size = 140,
  stroke = 12,
  label,
  children,
}: {
  value: number;
  size?: number;
  stroke?: number;
  label: string;
  children?: React.ReactNode;
}) {
  const r = (size - stroke) / 2;
  const c = 2 * Math.PI * r;
  const v = Math.max(0, Math.min(1, value));
  return (
    <div className="relative grid place-items-center" style={{ width: size, height: size }}>
      <svg width={size} height={size} role="img" aria-label={label} className="-rotate-90">
        <circle cx={size / 2} cy={size / 2} r={r} fill="none" stroke="currentColor" strokeOpacity={0.15} strokeWidth={stroke} />
        <circle
          cx={size / 2}
          cy={size / 2}
          r={r}
          fill="none"
          stroke="#6ee7b7"
          strokeWidth={stroke}
          strokeLinecap="round"
          strokeDasharray={c}
          strokeDashoffset={c * (1 - v)}
        />
      </svg>
      <div className="absolute inset-0 grid place-items-center text-center">{children}</div>
    </div>
  );
}

/** Tabellarische Alternative für Diagramme (Barrierefreiheit). */
export function DataTableDisclosure({
  summary,
  head,
  rows,
}: {
  summary: string;
  head: string[];
  rows: (string | number)[][];
}) {
  return (
    <details className="group mt-3 text-sm">
      <summary className="cursor-pointer rounded-lg py-1 text-muted-foreground hover:text-foreground focus-visible:outline-2 focus-visible:outline-ring">
        {summary}
      </summary>
      <div className="mt-2 max-h-64 overflow-auto rounded-xl border border-border">
        <table className="w-full text-left text-sm">
          <thead className="sticky top-0 bg-muted">
            <tr>
              {head.map((h) => (
                <th key={h} scope="col" className="px-3 py-2 font-semibold">
                  {h}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {rows.map((r, i) => (
              <tr key={i} className="border-t border-border">
                {r.map((c, j) => (
                  <td key={j} className="px-3 py-1.5 tabular-nums">
                    {c}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </details>
  );
}
````

## Lebensmittel-Verwaltung

`src/components/food-manager.tsx`

````tsx
import { useState } from "react";
import { Pencil, Plus, Search, Trash2, X } from "lucide-react";
import { useHub } from "@/lib/store";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Badge } from "@/components/ui/badge";
import { NativeSelect, NativeSelectOption } from "@/components/ui/native-select";
import { catLabel, fmt0, fmt1 } from "@/lib/calc";
import type { Food, FoodCategory } from "@/lib/types";
import { toast } from "sonner";


type Draft = { name: string; category: FoodCategory; kcal: string; protein: string; carbs: string; fat: string };
const empty: Draft = { name: "", category: "protein", kcal: "", protein: "", carbs: "", fat: "" };
const num = (v: string) => Number(v.replace(",", "."));

export function FoodManager() {
  const foods = useHub((s) => s.foods);
  const addFood = useHub((s) => s.addFood);
  const updateFood = useHub((s) => s.updateFood);
  const removeFood = useHub((s) => s.removeFood);
  const [draft, setDraft] = useState<Draft>(empty);
  const [editId, setEditId] = useState<string | null>(null);
  const [q, setQ] = useState("");

  const p = num(draft.protein) || 0;
  const c = num(draft.carbs) || 0;
  const f = num(draft.fat) || 0;
  const macroKcal = p * 4 + c * 4 + f * 9;

  const startEdit = (food: Food) => {
    setEditId(food.id);
    setDraft({
      name: food.name,
      category: food.category,
      kcal: String(food.kcal),
      protein: String(food.protein),
      carbs: String(food.carbs),
      fat: String(food.fat),
    });
  };
  const reset = () => {
    setEditId(null);
    setDraft(empty);
  };

  const shown = foods
    .filter((x) => x.name.toLowerCase().includes(q.trim().toLowerCase()))
    .sort((a, b) => Number(!!b.custom) - Number(!!a.custom) || a.name.localeCompare(b.name, "de"));

  return (
    <section aria-labelledby="foods-h" className="space-y-4">
      <h2 id="foods-h" className="text-xl font-bold">
        Lebensmittel & Nährwerte
      </h2>
      <div className="grid gap-4 lg:grid-cols-[1fr_1.3fr]">
        <Card className="gap-4 p-5">
          <h3 className="font-bold">{editId ? "Lebensmittel bearbeiten" : "Eigenes Lebensmittel anlegen"}</h3>
          <form
            className="space-y-3"
            onSubmit={(e) => {
              e.preventDefault();
              const name = draft.name.trim();
              const values = { kcal: num(draft.kcal || String(Math.round(macroKcal))), protein: p, carbs: c, fat: f };
              if (!name) return toast.error("Bitte einen Namen eingeben.");
              if (Object.values(values).some((v) => !Number.isFinite(v) || v < 0)) return toast.error("Bitte gültige Nährwerte eingeben.");
              if (editId) {
                updateFood(editId, { name, category: draft.category, ...values });
                toast.success(`${name} aktualisiert`);
              } else {
                addFood({ name, category: draft.category, ...values });
                toast.success(`${name} hinzugefügt – jetzt in allen Mahlzeiten wählbar`);
              }
              reset();
            }}
          >
            <div className="grid grid-cols-2 gap-3">
              <div className="col-span-2 space-y-1 sm:col-span-1">
                <Label htmlFor="fname">Name</Label>
                <Input id="fname" value={draft.name} onChange={(e) => setDraft({ ...draft, name: e.target.value })} placeholder="z. B. Proteinbrot" />
              </div>
              <div className="col-span-2 space-y-1 sm:col-span-1">
                <Label htmlFor="fcat">Kategorie</Label>
                <NativeSelect id="fcat" value={draft.category} onChange={(e) => setDraft({ ...draft, category: e.target.value as FoodCategory })} className="w-full">
                  {Object.entries(catLabel).map(([k, l]) => (
                    <NativeSelectOption key={k} value={k}>{l}</NativeSelectOption>
                  ))}
                </NativeSelect>
              </div>
            </div>
            <fieldset>
              <legend className="mb-2 text-sm font-semibold">Nährwerte je 100 g</legend>
              <div className="grid grid-cols-2 gap-3 sm:grid-cols-4">
                {([
                  ["protein", "Protein (g)"],
                  ["carbs", "Kohlenhydrate (g)"],
                  ["fat", "Fett (g)"],
                  ["kcal", "Kalorien (kcal)"],
                ] as const).map(([k, l]) => (
                  <div key={k} className="space-y-1">
                    <Label htmlFor={`f-${k}`} className="text-xs">{l}</Label>
                    <Input
                      id={`f-${k}`}
                      inputMode="decimal"
                      value={draft[k]}
                      placeholder={k === "kcal" && macroKcal > 0 ? String(Math.round(macroKcal)) : "0"}
                      onChange={(e) => setDraft({ ...draft, [k]: e.target.value })}
                      className="tabular-nums"
                    />
                  </div>
                ))}
              </div>
              <p className="mt-2 text-xs text-muted-foreground">
                Aus den Makros berechnet: {fmt0(macroKcal)} kcal. Bleibt das Kalorienfeld leer, wird dieser Wert übernommen.
              </p>
            </fieldset>
            <div className="flex flex-wrap gap-2">
              <Button type="submit" className="rounded-full">
                <Plus className="size-4" aria-hidden="true" /> {editId ? "Änderungen speichern" : "Lebensmittel speichern"}
              </Button>
              {editId && (
                <Button type="button" variant="ghost" className="rounded-full" onClick={reset}>
                  <X className="size-4" aria-hidden="true" /> Abbrechen
                </Button>
              )}
            </div>
          </form>
        </Card>

        <Card className="gap-3 p-5">
          <div className="relative">
            <Search className="pointer-events-none absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" aria-hidden="true" />
            <label htmlFor="fsearch" className="sr-only">Lebensmittel suchen</label>
            <Input id="fsearch" value={q} onChange={(e) => setQ(e.target.value)} placeholder="Lebensmittel suchen" className="rounded-full pl-9" />
          </div>
          <div className="max-h-[26rem] overflow-auto rounded-xl border border-border">
            <table className="w-full text-sm">
              <thead className="sticky top-0 bg-muted text-left text-xs">
                <tr>
                  <th scope="col" className="px-3 py-2">Lebensmittel</th>
                  <th scope="col" className="px-2 py-2 text-right">kcal</th>
                  <th scope="col" className="px-2 py-2 text-right">P</th>
                  <th scope="col" className="px-2 py-2 text-right">KH</th>
                  <th scope="col" className="px-2 py-2 text-right">F</th>
                  <th scope="col" className="w-20 px-2 py-2"><span className="sr-only">Aktionen</span></th>
                </tr>
              </thead>
              <tbody>
                {shown.map((x) => (
                  <tr key={x.id} className="border-t border-border">
                    <td className="px-3 py-1.5">
                      <span className="font-medium">{x.name}</span>
                      <span className="block text-xs text-muted-foreground">
                        {catLabel[x.category]} {x.custom && <Badge variant="secondary" className="ml-1">eigenes</Badge>}
                      </span>
                    </td>
                    <td className="px-2 text-right tabular-nums">{fmt0(x.kcal)}</td>
                    <td className="px-2 text-right tabular-nums">{fmt1(x.protein)}</td>
                    <td className="px-2 text-right tabular-nums">{fmt1(x.carbs)}</td>
                    <td className="px-2 text-right tabular-nums">{fmt1(x.fat)}</td>
                    <td className="px-1 text-right">
                      <Button type="button" variant="ghost" size="icon" aria-label={`${x.name} bearbeiten`} onClick={() => startEdit(x)}>
                        <Pencil className="size-4" aria-hidden="true" />
                      </Button>
                      {x.custom && (
                        <Button
                          type="button"
                          variant="ghost"
                          size="icon"
                          aria-label={`${x.name} löschen`}
                          onClick={() => {
                            if (removeFood(x.id)) toast.success(`${x.name} gelöscht`);
                            else toast.error(`${x.name} wird noch in einer Mahlzeit verwendet.`);
                          }}
                        >
                          <Trash2 className="size-4" aria-hidden="true" />
                        </Button>
                      )}
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          <p className="text-xs text-muted-foreground">Werte je 100 g · P = Protein, KH = Kohlenhydrate, F = Fett</p>
        </Card>
      </div>
    </section>
  );
}
````

## Seite: Dashboard

`src/pages/home.tsx`

````tsx
import { Link } from "react-router-dom";
import { useState } from "react";
import { ArrowRight, Check, Flame, Footprints, Beef, Dumbbell, Pencil, Scale } from "lucide-react";
import { useHub } from "@/lib/store";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Progress } from "@/components/ui/progress";
import { Label } from "@/components/ui/label";
import { Dialog, DialogContent, DialogDescription, DialogFooter, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { ProgressRing, WeightChart, DataTableDisclosure } from "@/components/charts";
import { addDays, carbTargets, fmt0, fmt1, fmtDate, signed1, sortedCheckIns, TODAY, weekStart, workoutsThisWeek } from "@/lib/calc";
import { toast } from "sonner";

export function HomePage() {
  const profile = useHub((s) => s.profile);
  const targets = useHub((s) => s.targets);
  const checkInsRaw = useHub((s) => s.checkIns);
  const dailyLogs = useHub((s) => s.dailyLogs);
  const workoutLogs = useHub((s) => s.workoutLogs);
  const plans = useHub((s) => s.plans);
  const setSteps = useHub((s) => s.setSteps);
  const setProfile = useHub((s) => s.setProfile);
  const setTargets = useHub((s) => s.setTargets);
  const addCheckIn = useHub((s) => s.addCheckIn);
  const removeCheckIn = useHub((s) => s.removeCheckIn);
  const logWorkout = useHub((s) => s.logWorkout);
  const removeWorkoutLog = useHub((s) => s.removeWorkoutLog);
  const role = useHub((s) => s.role);
  const [profileOpen, setProfileOpen] = useState(false);
  const [weightOpen, setWeightOpen] = useState(false);
  const [editGoals, setEditGoals] = useState(false);

  const checkIns = sortedCheckIns(checkInsRaw);
  const latest = checkIns[checkIns.length - 1];
  const prev = checkIns[checkIns.length - 2];
  const current = latest?.weight ?? profile.startWeight;
  const lost = profile.startWeight - current;
  const toLose = profile.startWeight - profile.goalWeight;
  const progress = toLose > 0 ? lost / toLose : 0;
  const weeks = Math.max(1, Math.round((new Date(TODAY).getTime() - new Date(profile.startDate).getTime()) / (7 * 864e5)));
  const carbs = carbTargets(targets);

  const today = dailyLogs.find((d) => d.date === TODAY)?.steps ?? 0;
  const ws = weekStart(TODAY);
  const last7 = Array.from({ length: 7 }, (_, i) => addDays(TODAY, i - 6)).map((d) => ({
    date: d,
    steps: dailyLogs.find((x) => x.date === d)?.steps ?? 0,
  }));
  const avg7 = last7.reduce((a, b) => a + b.steps, 0) / 7;
  const weekLogs = workoutsThisWeek(workoutLogs);
  const nextCheckIn = latest ? addDays(latest.date, 7) : TODAY;
  const checkInDue = nextCheckIn <= TODAY;

  return (
    <div className="space-y-6">
      {/* Hero */}
      <section className="relative overflow-hidden rounded-[1.75rem] bg-gradient-to-br from-blue-950 via-blue-900 to-blue-800 p-6 text-white sm:p-8">
        <div className="pointer-events-none absolute -right-16 -top-20 size-64 rounded-full bg-emerald-300/20 blur-2xl" aria-hidden="true" />
        <div className="relative flex flex-col gap-6 sm:flex-row sm:items-center sm:justify-between">
          <div>
            <h1 className="text-3xl font-bold tracking-tight sm:text-4xl">Hallo {profile.name}</h1>
            <p className="mt-1 text-blue-100">Woche {weeks} · seit {fmtDate(profile.startDate, { day: "2-digit", month: "long" })}</p>
            <div className="mt-6 flex flex-wrap items-end gap-x-8 gap-y-4">
              <div>
                <p className="text-xs uppercase tracking-wider text-blue-200">Aktuelles Gewicht</p>
                <p className="text-5xl font-bold tabular-nums">
                  {fmt1(current)}
                  <span className="ml-1 text-xl font-medium text-blue-200">kg</span>
                </p>
              </div>
              <div className="space-y-1 text-sm">
                <p>
                  <span className="font-semibold text-emerald-300">{signed1(-lost)} kg</span> seit Start
                </p>
                {prev && (
                  <p>
                    <span className="font-semibold text-emerald-300">{signed1(current - prev.weight)} kg</span> zur Vorwoche
                  </p>
                )}
              </div>
            </div>
            <div className="mt-5 flex flex-wrap gap-2">
              <Button type="button" onClick={() => setWeightOpen(true)} className="rounded-full bg-emerald-300 text-blue-950 hover:bg-emerald-200">
                <Scale className="size-4" aria-hidden="true" /> Gewicht eintragen
              </Button>
              <Button type="button" variant="outline" onClick={() => setProfileOpen(true)} className="rounded-full border-white/40 bg-transparent text-white hover:bg-white/10 hover:text-white">
                <Pencil className="size-4" aria-hidden="true" /> Profil & Ziel bearbeiten
              </Button>
            </div>
          </div>
          <ProgressRing value={progress} label={`${Math.round(progress * 100)} Prozent des Zielgewichts erreicht`}>
            <div>
              <p className="text-3xl font-bold tabular-nums">{Math.round(Math.max(0, progress) * 100)}%</p>
              <p className="text-[11px] text-blue-100">zum Ziel {fmt1(profile.goalWeight)} kg</p>
            </div>
          </ProgressRing>
        </div>
      </section>

      <div className="grid gap-6 lg:grid-cols-[1.6fr_1fr]">
        {/* Weight chart */}
        <Card className="gap-0 p-5 sm:p-6">
          <div className="flex items-baseline justify-between gap-3">
            <h2 className="text-lg font-bold">Gewichtsverlauf</h2>
            <Link to="/fortschritt" className="inline-flex items-center gap-1 text-sm font-medium text-primary hover:underline">
              Fortschritt <ArrowRight className="size-4" aria-hidden="true" />
            </Link>
          </div>
          <p className="mb-3 text-sm text-muted-foreground">
            {checkIns.length} Check-ins · von {fmt1(checkIns[0]?.weight ?? 0)} auf {fmt1(current)} kg
          </p>
          <WeightChart checkIns={checkIns} goal={profile.goalWeight} className="h-64 w-full sm:h-72" />
          <DataTableDisclosure
            summary="Werte als Tabelle"
            head={["Datum", "Gewicht (kg)"]}
            rows={checkIns.map((c) => [fmtDate(c.date, { day: "2-digit", month: "2-digit", year: "numeric" }), fmt1(c.weight)])}
          />
        </Card>

        {/* Ziele */}
        <div className="space-y-4">
          <Card className="gap-4 p-5">
            <div className="flex items-center justify-between">
              <h2 className="text-lg font-bold">Tagesziele</h2>
              <Button type="button" variant="ghost" size="sm" aria-expanded={editGoals} onClick={() => setEditGoals((v) => !v)}>
                <Pencil className="size-4" aria-hidden="true" /> {editGoals ? "Fertig" : "Anpassen"}
              </Button>
            </div>
            {editGoals ? (
              <div className="grid grid-cols-2 gap-3">
                {([
                  ["kcalMin", "Kalorien min", 10],
                  ["kcalMax", "Kalorien max", 10],
                  ["proteinMin", "Protein min (g)", 1],
                  ["proteinMax", "Protein max (g)", 1],
                  ["fatMin", "Fett min (g)", 1],
                  ["fatMax", "Fett max (g)", 1],
                  ["stepGoal", "Schrittziel", 500],
                  ["workoutsPerWeek", "Trainings / Woche", 1],
                ] as const).map(([k, l, step]) => (
                  <div key={k} className="space-y-1">
                    <Label htmlFor={`g-${k}`} className="text-xs text-muted-foreground">{l}</Label>
                    <Input
                      id={`g-${k}`}
                      type="number"
                      min={0}
                      step={step}
                      value={targets[k]}
                      onChange={(e) => {
                        const n = Number(e.target.value);
                        if (Number.isFinite(n) && n >= 0) setTargets({ [k]: n });
                      }}
                      className="h-9 tabular-nums"
                    />
                  </div>
                ))}
                <p className="col-span-2 text-xs text-muted-foreground">Änderungen werden sofort gespeichert. Kohlenhydrate: ca. {carbs.mid} g</p>
              </div>
            ) : (
              <>
                <dl className="grid grid-cols-2 gap-3">
                  <div className="rounded-2xl bg-accent p-3">
                    <dt className="flex items-center gap-1.5 text-xs font-medium text-accent-foreground">
                      <Flame className="size-4" aria-hidden="true" /> Kalorien
                    </dt>
                    <dd className="mt-1 text-lg font-bold tabular-nums">
                      {fmt0(targets.kcalMin)}–{fmt0(targets.kcalMax)}
                      <span className="ml-1 text-xs font-medium text-muted-foreground">kcal</span>
                    </dd>
                  </div>
                  <div className="rounded-2xl bg-emerald-50 p-3 text-emerald-950 dark:bg-emerald-950 dark:text-emerald-50">
                    <dt className="flex items-center gap-1.5 text-xs font-medium">
                      <Beef className="size-4" aria-hidden="true" /> Protein
                    </dt>
                    <dd className="mt-1 text-lg font-bold tabular-nums">
                      {targets.proteinMin}–{targets.proteinMax}
                      <span className="ml-1 text-xs font-medium">g</span>
                    </dd>
                  </div>
                </dl>
                <p className="text-xs text-muted-foreground">
                  Fett {targets.fatMin}–{targets.fatMax} g · Kohlenhydrate ca. {carbs.mid} g ({carbs.min}–{carbs.max} g)
                </p>
              </>
            )}
          </Card>

          <Card className="gap-3 p-5">
            <div className="flex items-center justify-between">
              <h2 className="flex items-center gap-2 text-lg font-bold">
                <Footprints className="size-5 text-primary" aria-hidden="true" /> Schritte
              </h2>
              <span className="text-sm text-muted-foreground">Ziel {fmt0(targets.stepGoal)}</span>
            </div>
            <p className="text-2xl font-bold tabular-nums">
              {fmt0(today)} <span className="text-sm font-medium text-muted-foreground">heute</span>
            </p>
            <Progress value={Math.min(100, (today / targets.stepGoal) * 100)} aria-label="Schritte heute im Verhältnis zum Ziel" />
            <form
              className="flex gap-2"
              onSubmit={(e) => {
                e.preventDefault();
                const v = Number(new FormData(e.currentTarget).get("steps"));
                if (!Number.isFinite(v) || v < 0) return;
                setSteps({ date: TODAY, steps: Math.round(v) });
                toast.success("Schritte aktualisiert");
                e.currentTarget.reset();
              }}
            >
              <label htmlFor="steps" className="sr-only">Schritte heute eintragen</label>
              <Input id="steps" name="steps" type="number" min={0} inputMode="numeric" placeholder="Schritte eintragen" className="rounded-full" />
              <Button type="submit" className="rounded-full">Speichern</Button>
            </form>
            <div>
              <ul className="flex h-14 items-end gap-1.5" aria-label="Schritte der letzten 7 Tage">
                {last7.map((d) => {
                  const hit = d.steps >= targets.stepGoal;
                  return (
                    <li key={d.date} className="flex flex-1 flex-col items-center gap-1">
                      <span
                        className={hit ? "w-full rounded-md bg-emerald-400" : "w-full rounded-md bg-primary/30"}
                        style={{ height: `${Math.max(6, Math.min(100, (d.steps / (targets.stepGoal * 1.4)) * 100))}%` }}
                        aria-hidden="true"
                      />
                      <span className="sr-only">
                        {fmtDate(d.date)}: {fmt0(d.steps)} Schritte{hit ? ", Ziel erreicht" : ""}
                      </span>
                    </li>
                  );
                })}
              </ul>
              <p className="mt-1 text-xs text-muted-foreground">Ø 7 Tage: {fmt0(avg7)} Schritte</p>
            </div>
          </Card>

          <Card className="gap-3 p-5">
            <div className="flex items-center justify-between">
              <h2 className="flex items-center gap-2 text-lg font-bold">
                <Dumbbell className="size-5 text-primary" aria-hidden="true" /> Training diese Woche
              </h2>
              <span className="text-sm font-semibold tabular-nums">
                {Math.min(weekLogs.length, targets.workoutsPerWeek)}/{targets.workoutsPerWeek}
              </span>
            </div>
            <ul className="grid grid-cols-2 gap-2">
              {plans.map((p) => {
                const done = weekLogs.find((l) => l.planId === p.id);
                return (
                  <li key={p.id}>
                    <button
                      type="button"
                      aria-pressed={!!done}
                      onClick={() => {
                        if (done) {
                          removeWorkoutLog(done.id);
                          toast(`${p.name} wieder auf offen gesetzt`);
                        } else {
                          logWorkout({
                            date: TODAY,
                            planId: p.id,
                            note: "",
                            entries: p.exercises.map((e) => ({ exerciseId: e.id, name: e.name, sets: e.sets, reps: e.reps, weight: e.weight })),
                          });
                          toast.success(`${p.name} als erledigt eingetragen`);
                        }
                      }}
                      className={
                        done
                          ? "w-full rounded-2xl bg-emerald-100 p-3 text-left text-emerald-950 focus-visible:outline-2 focus-visible:outline-ring"
                          : "w-full rounded-2xl border border-dashed border-border p-3 text-left hover:bg-muted focus-visible:outline-2 focus-visible:outline-ring"
                      }
                    >
                    <p className="flex items-center gap-1.5 text-sm font-semibold">
                      {done && <Check className="size-4" aria-hidden="true" />}
                      {p.name}
                    </p>
                    <p className={done ? "text-xs" : "text-xs text-muted-foreground"}>
                      {done ? `erledigt ${fmtDate(done.date, { weekday: "short", day: "2-digit", month: "2-digit" })}` : "offen – antippen zum Abhaken"}
                    </p>
                    </button>
                  </li>
                );
              })}
            </ul>
            <p className="text-xs text-muted-foreground">Woche ab {fmtDate(ws, { day: "2-digit", month: "long" })}</p>
          </Card>

          <Card className={checkInDue ? "gap-2 border-emerald-400 p-5" : "gap-2 p-5"}>
            <h2 className="text-lg font-bold">Nächster Check-in</h2>
            <p className="text-sm">
              {checkInDue ? "Heute fällig" : fmtDate(nextCheckIn, { weekday: "long", day: "2-digit", month: "long" })}
            </p>
            <Button asChild className="mt-1 self-start rounded-full">
              <Link to="/check-in">Check-in ausfüllen</Link>
            </Button>
          </Card>
        </div>
      </div>
      <Dialog open={weightOpen} onOpenChange={setWeightOpen}>
        <DialogContent className="sm:max-w-md">
          <DialogHeader>
            <DialogTitle>Gewicht eintragen</DialogTitle>
            <DialogDescription>Umfänge und Skalen werden aus dem letzten Check-in übernommen. Am selben Tag wird der Eintrag ersetzt.</DialogDescription>
          </DialogHeader>
          <form
            className="space-y-3"
            onSubmit={(e) => {
              e.preventDefault();
              const fd = new FormData(e.currentTarget);
              const w = Number(String(fd.get("w")).replace(",", "."));
              const date = String(fd.get("d") || TODAY);
              if (!(w > 30 && w < 250)) return toast.error("Bitte ein gültiges Gewicht eingeben.");
              const same = checkIns.find((c) => c.date === date);
              const base = same ?? latest;
              if (same) removeCheckIn(same.id);
              addCheckIn({
                date,
                weight: w,
                hip: base?.hip ?? 0,
                waist: base?.waist ?? 0,
                energy: base?.energy ?? 5,
                hunger: base?.hunger ?? 5,
                sleep: base?.sleep ?? 5,
                feedback: same?.feedback ?? "",
                author: role,
              });
              toast.success(`${fmt1(w)} kg gespeichert`);
              setWeightOpen(false);
            }}
          >
            <div className="grid grid-cols-2 gap-3">
              <div className="space-y-1">
                <Label htmlFor="qw">Gewicht (kg)</Label>
                <Input id="qw" name="w" inputMode="decimal" autoFocus placeholder={fmt1(current)} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="qd">Datum</Label>
                <Input id="qd" name="d" type="date" defaultValue={TODAY} />
              </div>
            </div>
            <DialogFooter>
              <Button type="submit" className="rounded-full">Speichern</Button>
            </DialogFooter>
          </form>
        </DialogContent>
      </Dialog>

      <Dialog open={profileOpen} onOpenChange={setProfileOpen}>
        <DialogContent className="sm:max-w-md">
          <DialogHeader>
            <DialogTitle>Profil & Ziel bearbeiten</DialogTitle>
            <DialogDescription>Gilt für Dashboard, Fortschritt und Berichte.</DialogDescription>
          </DialogHeader>
          <form
            className="space-y-3"
            onSubmit={(e) => {
              e.preventDefault();
              const fd = new FormData(e.currentTarget);
              const n = (k: string) => Number(String(fd.get(k)).replace(",", "."));
              const patch = {
                name: String(fd.get("name") ?? "").trim() || profile.name,
                startDate: String(fd.get("startDate") || profile.startDate),
                startWeight: n("startWeight"),
                goalWeight: n("goalWeight"),
                heightCm: n("heightCm"),
              };
              if (![patch.startWeight, patch.goalWeight, patch.heightCm].every((v) => v > 0)) return toast.error("Bitte gültige Werte eingeben.");
              setProfile(patch);
              toast.success("Profil gespeichert");
              setProfileOpen(false);
            }}
          >
            <div className="grid grid-cols-2 gap-3">
              <div className="col-span-2 space-y-1">
                <Label htmlFor="pn">Name</Label>
                <Input id="pn" name="name" defaultValue={profile.name} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="psd">Startdatum</Label>
                <Input id="psd" name="startDate" type="date" defaultValue={profile.startDate} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="ph">Größe (cm)</Label>
                <Input id="ph" name="heightCm" inputMode="decimal" defaultValue={profile.heightCm} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="psw">Startgewicht (kg)</Label>
                <Input id="psw" name="startWeight" inputMode="decimal" defaultValue={profile.startWeight} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="pgw">Zielgewicht (kg)</Label>
                <Input id="pgw" name="goalWeight" inputMode="decimal" defaultValue={profile.goalWeight} />
              </div>
            </div>
            <DialogFooter>
              <Button type="submit" className="rounded-full">Speichern</Button>
            </DialogFooter>
          </form>
        </DialogContent>
      </Dialog>
    </div>
  );
}
````

## Seite: Ernährungsplan

`src/pages/nutrition.tsx`

````tsx
import { useState } from "react";
import { Plus, Repeat, Trash2 } from "lucide-react";
import { useHub } from "@/lib/store";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { NativeSelect, NativeSelectOption } from "@/components/ui/native-select";
import { catLabel, carbTargets, equivalentGrams, fmt0, itemMacros, mealMacros, sumMacros } from "@/lib/calc";
import type { Meal, NutritionTargets } from "@/lib/types";
import { cn } from "@/lib/utils";
import { FoodManager } from "@/components/food-manager";
import { toast } from "sonner";


function NumField({
  id,
  label,
  value,
  onChange,
  step = 1,
}: {
  id: string;
  label: string;
  value: number;
  onChange: (n: number) => void;
  step?: number;
}) {
  return (
    <div className="space-y-1">
      <Label htmlFor={id} className="text-xs text-muted-foreground">
        {label}
      </Label>
      <Input
        id={id}
        type="number"
        step={step}
        min={0}
        value={value}
        onChange={(e) => {
          const n = Number(e.target.value);
          if (Number.isFinite(n) && n >= 0) onChange(n);
        }}
        className="tabular-nums"
      />
    </div>
  );
}

function MacroBar({
  label,
  value,
  min,
  max,
  unit,
  color,
}: {
  label: string;
  value: number;
  min: number;
  max: number;
  unit: string;
  color: string;
}) {
  const inRange = value >= min && value <= max;
  const scale = max * 1.25 || 1;
  return (
    <div>
      <div className="flex items-baseline justify-between text-sm">
        <span className="font-semibold">{label}</span>
        <span className="tabular-nums">
          <span className="font-bold">{fmt0(value)}</span>
          <span className="text-muted-foreground">
            {" "}
            / {fmt0(min)}–{fmt0(max)} {unit}
          </span>
        </span>
      </div>
      <div className="relative mt-1.5 h-3 rounded-full bg-muted" aria-hidden="true">
        <div
          className="absolute inset-y-0 rounded-full bg-emerald-200/70 dark:bg-emerald-900"
          style={{ left: `${(min / scale) * 100}%`, width: `${((max - min) / scale) * 100}%` }}
        />
        <div className={cn("absolute inset-y-0 left-0 rounded-full", color)} style={{ width: `${Math.min(100, (value / scale) * 100)}%` }} />
      </div>
      <p className={cn("mt-1 text-xs", inRange ? "text-emerald-800 dark:text-emerald-300" : "text-amber-800 dark:text-amber-300")}>
        {inRange ? "im Zielbereich" : value < min ? `${fmt0(min - value)} ${unit} unter Ziel` : `${fmt0(value - max)} ${unit} über Ziel`}
      </p>
    </div>
  );
}

function MealCard({ meal }: { meal: Meal }) {
  const foods = useHub((s) => s.foods);
  const updateMealItem = useHub((s) => s.updateMealItem);
  const addMealItem = useHub((s) => s.addMealItem);
  const removeMealItem = useHub((s) => s.removeMealItem);
  const [newFood, setNewFood] = useState(foods[0]?.id ?? "");
  const [newGrams, setNewGrams] = useState(100);
  const m = mealMacros(meal, foods);

  return (
    <Card className="gap-4 p-5">
      <div className="flex flex-wrap items-baseline justify-between gap-2">
        <h3 className="text-lg font-bold">{meal.title}</h3>
        <p className="text-sm tabular-nums text-muted-foreground">
          <span className="font-semibold text-foreground">{fmt0(m.kcal)} kcal</span> · P {fmt0(m.protein)} g · F {fmt0(m.fat)} g · KH {fmt0(m.carbs)} g
        </p>
      </div>
      <ul className="divide-y divide-border">
        {meal.items.map((item) => {
          const food = foods.find((f) => f.id === item.foodId);
          if (!food) return null;
          const im = itemMacros(food, item.grams);
          const alternatives = foods.filter((f) => f.category === food.category);
          return (
            <li key={item.id} className="grid grid-cols-[1fr_auto] gap-x-3 gap-y-2 py-3 sm:grid-cols-[minmax(0,1.4fr)_6rem_minmax(0,1fr)_auto] sm:items-center">
              <div className="min-w-0">
                <p className="truncate font-medium">{food.name}</p>
                <p className="text-xs text-muted-foreground">
                  {catLabel[food.category]} · {fmt0(im.kcal)} kcal · P {fmt0(im.protein)} g
                </p>
              </div>
              <div className="flex items-center gap-1 sm:order-none">
                <label htmlFor={`g-${item.id}`} className="sr-only">
                  Menge {food.name} in Gramm
                </label>
                <Input
                  id={`g-${item.id}`}
                  type="number"
                  min={0}
                  step={5}
                  value={item.grams}
                  onChange={(e) => {
                    const n = Number(e.target.value);
                    if (Number.isFinite(n) && n >= 0) updateMealItem(meal.slot, item.id, { grams: n });
                  }}
                  className="h-8 w-20 tabular-nums"
                />
                <span className="text-xs text-muted-foreground">g</span>
              </div>
              <div className="col-span-2 flex items-center gap-2 sm:col-span-1">
                <Repeat className="size-4 shrink-0 text-muted-foreground" aria-hidden="true" />
                <label htmlFor={`swap-${item.id}`} className="sr-only">
                  {food.name} austauschen
                </label>
                <NativeSelect
                  id={`swap-${item.id}`}
                  size="sm"
                  value={food.id}
                  onChange={(e) => {
                    const to = foods.find((f) => f.id === e.target.value);
                    if (!to) return;
                    const g = equivalentGrams(food, item.grams, to);
                    updateMealItem(meal.slot, item.id, { foodId: to.id, grams: g });
                    toast.success(`${food.name} → ${to.name} (${g} g, gleiche Kalorien)`);
                  }}
                  className="w-full"
                >
                  {alternatives.map((a) => (
                    <NativeSelectOption key={a.id} value={a.id}>
                      {a.name}
                    </NativeSelectOption>
                  ))}
                </NativeSelect>
              </div>
              <Button
                type="button"
                variant="ghost"
                size="icon"
                aria-label={`${food.name} entfernen`}
                onClick={() => removeMealItem(meal.slot, item.id)}
                className="row-start-1 col-start-2 justify-self-end sm:row-auto sm:col-auto"
              >
                <Trash2 className="size-4" aria-hidden="true" />
              </Button>
            </li>
          );
        })}
      </ul>
      <form
        className="flex flex-wrap items-end gap-2 rounded-2xl bg-muted/60 p-3"
        onSubmit={(e) => {
          e.preventDefault();
          if (!newFood || newGrams <= 0) return;
          addMealItem(meal.slot, newFood, newGrams);
        }}
      >
        <div className="min-w-0 flex-1 space-y-1">
          <Label htmlFor={`add-${meal.slot}`} className="text-xs">
            Lebensmittel hinzufügen
          </Label>
          <NativeSelect id={`add-${meal.slot}`} size="sm" value={newFood} onChange={(e) => setNewFood(e.target.value)} className="w-full">
            {foods.map((f) => (
              <NativeSelectOption key={f.id} value={f.id}>
                {f.name}
              </NativeSelectOption>
            ))}
          </NativeSelect>
        </div>
        <div className="space-y-1">
          <Label htmlFor={`addg-${meal.slot}`} className="text-xs">
            Gramm
          </Label>
          <Input id={`addg-${meal.slot}`} type="number" min={5} step={5} value={newGrams} onChange={(e) => setNewGrams(Number(e.target.value))} className="h-8 w-20" />
        </div>
        <Button type="submit" size="sm" className="rounded-full">
          <Plus className="size-4" aria-hidden="true" /> Hinzufügen
        </Button>
      </form>
    </Card>
  );
}

export function NutritionPage() {
  const targets = useHub((s) => s.targets);
  const setTargets = useHub((s) => s.setTargets);
  const meals = useHub((s) => s.meals);
  const foods = useHub((s) => s.foods);
  const carbs = carbTargets(targets);
  const total = sumMacros(meals.map((m) => mealMacros(m, foods)));
  const set = (k: keyof NutritionTargets) => (n: number) => setTargets({ [k]: n });

  const kcalP = carbs.pMid * 4;
  const kcalF = carbs.fMid * 9;
  const kcalC = carbs.mid * 4;
  const kcalSum = kcalP + kcalF + kcalC || 1;
  const split = [
    { l: "Protein", v: kcalP, c: "bg-blue-900" },
    { l: "Fett", v: kcalF, c: "bg-emerald-400" },
    { l: "Kohlenhydrate", v: kcalC, c: "bg-sky-300" },
  ];

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold tracking-tight">Ernährungsplan</h1>

      <div className="grid gap-6 lg:grid-cols-[1fr_1.2fr]">
        <Card className="gap-5 p-5 sm:p-6">
          <h2 className="text-lg font-bold">Ziele anpassen</h2>
          <div className="grid grid-cols-2 gap-3">
            <NumField id="kmin" label="Kalorien min (kcal)" value={targets.kcalMin} onChange={set("kcalMin")} step={10} />
            <NumField id="kmax" label="Kalorien max (kcal)" value={targets.kcalMax} onChange={set("kcalMax")} step={10} />
            <NumField id="pmin" label="Protein min (g)" value={targets.proteinMin} onChange={set("proteinMin")} />
            <NumField id="pmax" label="Protein max (g)" value={targets.proteinMax} onChange={set("proteinMax")} />
            <NumField id="fmin" label="Fett min (g)" value={targets.fatMin} onChange={set("fatMin")} />
            <NumField id="fmax" label="Fett max (g)" value={targets.fatMax} onChange={set("fatMax")} />
            <NumField id="steps" label="Schrittziel / Tag" value={targets.stepGoal} onChange={set("stepGoal")} step={500} />
            <NumField id="wpw" label="Trainings / Woche" value={targets.workoutsPerWeek} onChange={set("workoutsPerWeek")} />
          </div>
          <div className="rounded-2xl bg-accent p-4 text-accent-foreground">
            <p className="text-xs font-semibold uppercase tracking-wider">Kohlenhydrate (automatisch)</p>
            <p className="mt-1 text-3xl font-bold tabular-nums">
              {carbs.mid} g <span className="text-base font-medium">({carbs.min}–{carbs.max} g)</span>
            </p>
            <p className="mt-1 text-xs">(Kalorien − Protein × 4 − Fett × 9) ÷ 4</p>
          </div>
          <div>
            <p className="mb-2 text-sm font-semibold">Energieverteilung</p>
            <div className="flex h-4 overflow-hidden rounded-full" aria-hidden="true">
              {split.map((s) => (
                <div key={s.l} className={s.c} style={{ width: `${(s.v / kcalSum) * 100}%` }} />
              ))}
            </div>
            <ul className="mt-2 flex flex-wrap gap-x-4 gap-y-1 text-xs">
              {split.map((s) => (
                <li key={s.l} className="flex items-center gap-1.5">
                  <span className={cn("size-2.5 rounded-full", s.c)} aria-hidden="true" />
                  {s.l} {Math.round((s.v / kcalSum) * 100)} %
                </li>
              ))}
            </ul>
          </div>
        </Card>

        <Card className="gap-5 p-5 sm:p-6">
          <div className="flex items-baseline justify-between">
            <h2 className="text-lg font-bold">Tagesplan vs. Ziel</h2>
            <p className="text-2xl font-bold tabular-nums">
              {fmt0(total.kcal)} <span className="text-sm font-medium text-muted-foreground">kcal</span>
            </p>
          </div>
          <MacroBar label="Kalorien" value={total.kcal} min={targets.kcalMin} max={targets.kcalMax} unit="kcal" color="bg-blue-900" />
          <MacroBar label="Protein" value={total.protein} min={targets.proteinMin} max={targets.proteinMax} unit="g" color="bg-blue-700" />
          <MacroBar label="Fett" value={total.fat} min={targets.fatMin} max={targets.fatMax} unit="g" color="bg-emerald-500" />
          <MacroBar label="Kohlenhydrate" value={total.carbs} min={carbs.min} max={carbs.max} unit="g" color="bg-sky-400" />
        </Card>
      </div>

      <section className="space-y-4" aria-labelledby="meals-h">
        <div className="flex flex-wrap items-baseline justify-between gap-2">
          <h2 id="meals-h" className="text-xl font-bold">
            Mahlzeiten
          </h2>
          <p className="text-sm text-muted-foreground">Beim Austauschen wird die Menge auf gleiche Kalorien umgerechnet.</p>
        </div>
        <div className="grid gap-4 xl:grid-cols-2">
          {meals.map((m) => (
            <MealCard key={m.slot} meal={m} />
          ))}
        </div>
      </section>

      <FoodManager />
    </div>
  );
}
````

## Seite: Check-in

`src/pages/check-in.tsx`

````tsx
import { useState } from "react";
import { Trash2 } from "lucide-react";
import { useHub } from "@/lib/store";
import { useConfirm } from "@/hooks/use-confirm";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { fmt1, fmtDate, signed1, sortedCheckIns, TODAY } from "@/lib/calc";
import { cn } from "@/lib/utils";
import { toast } from "sonner";

function Scale({
  name,
  label,
  low,
  high,
  value,
  onChange,
}: {
  name: string;
  label: string;
  low: string;
  high: string;
  value: number;
  onChange: (n: number) => void;
}) {
  return (
    <fieldset className="space-y-2">
      <legend className="flex w-full items-baseline justify-between text-sm font-semibold">
        <span>{label}</span>
        <span className="tabular-nums text-primary">{value}/10</span>
      </legend>
      <div className="grid grid-cols-10 gap-1">
        {Array.from({ length: 10 }, (_, i) => i + 1).map((n) => (
          <label key={n} className="relative">
            <input
              type="radio"
              name={name}
              value={n}
              checked={value === n}
              onChange={() => onChange(n)}
              className="peer sr-only"
            />
            <span
              className={cn(
                "grid h-9 cursor-pointer place-items-center rounded-xl border text-sm font-semibold tabular-nums transition-colors peer-focus-visible:outline-2 peer-focus-visible:outline-offset-2 peer-focus-visible:outline-ring",
                value === n
                  ? "border-blue-950 bg-blue-950 text-white dark:border-emerald-300 dark:bg-emerald-300 dark:text-blue-950"
                  : n <= value
                    ? "border-emerald-300 bg-emerald-100 text-emerald-950"
                    : "border-border bg-card hover:bg-muted",
              )}
            >
              {n}
            </span>
          </label>
        ))}
      </div>
      <div className="flex justify-between text-xs text-muted-foreground">
        <span>{low}</span>
        <span>{high}</span>
      </div>
    </fieldset>
  );
}

export function CheckInPage() {
  const role = useHub((s) => s.role);
  const raw = useHub((s) => s.checkIns);
  const addCheckIn = useHub((s) => s.addCheckIn);
  const removeCheckIn = useHub((s) => s.removeCheckIn);
  const confirm = useConfirm();
  const checkIns = sortedCheckIns(raw);
  const last = checkIns[checkIns.length - 1];

  const [energy, setEnergy] = useState(7);
  const [hunger, setHunger] = useState(5);
  const [sleep, setSleep] = useState(7);

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold tracking-tight">Wöchentlicher Check-in</h1>

      <div className="grid gap-6 lg:grid-cols-[1.1fr_1fr]">
        <Card className="p-5 sm:p-6">
          <form
            className="space-y-6"
            onSubmit={(e) => {
              e.preventDefault();
              const fd = new FormData(e.currentTarget);
              const weight = Number(fd.get("weight"));
              const hip = Number(fd.get("hip"));
              const waist = Number(fd.get("waist"));
              const date = String(fd.get("date") || TODAY);
              if (!(weight > 0 && hip > 0 && waist > 0)) {
                toast.error("Bitte Gewicht, Hüft- und Bauchumfang eintragen.");
                return;
              }
              addCheckIn({
                date,
                weight,
                hip,
                waist,
                energy,
                hunger,
                sleep,
                feedback: String(fd.get("feedback") ?? "").trim(),
                author: role,
              });
              toast.success("Check-in gespeichert");
              e.currentTarget.reset();
            }}
          >
            <h2 className="text-lg font-bold">Neuer Check-in</h2>
            <div className="grid grid-cols-2 gap-3 sm:grid-cols-4">
              <div className="col-span-2 space-y-1 sm:col-span-1">
                <Label htmlFor="date">Datum</Label>
                <Input id="date" name="date" type="date" defaultValue={TODAY} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="weight">Gewicht (kg)</Label>
                <Input id="weight" name="weight" type="number" step="0.1" min="30" placeholder={last ? fmt1(last.weight).replace(",", ".") : ""} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="hip">Hüfte (cm)</Label>
                <Input id="hip" name="hip" type="number" step="0.1" min="40" placeholder={last ? String(last.hip) : ""} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="waist">Bauch (cm)</Label>
                <Input id="waist" name="waist" type="number" step="0.1" min="40" placeholder={last ? String(last.waist) : ""} />
              </div>
            </div>
            <Scale name="energy" label="Energielevel" low="sehr niedrig" high="sehr hoch" value={energy} onChange={setEnergy} />
            <Scale name="hunger" label="Hungergefühl" low="kaum Hunger" high="ständig hungrig" value={hunger} onChange={setHunger} />
            <Scale name="sleep" label="Schlafqualität" low="sehr schlecht" high="sehr gut" value={sleep} onChange={setSleep} />
            <div className="space-y-1">
              <Label htmlFor="feedback">Feedback an {role === "coach" ? "Elena" : "den Coach"}</Label>
              <Textarea id="feedback" name="feedback" rows={4} placeholder="Wie lief die Woche? Was war leicht, was schwer?" />
            </div>
            <Button type="submit" className="w-full rounded-full sm:w-auto">
              Check-in speichern
            </Button>
          </form>
        </Card>

        <section className="space-y-3" aria-labelledby="hist-h">
          <h2 id="hist-h" className="text-lg font-bold">
            Verlauf
          </h2>
          <ol className="space-y-3">
            {[...checkIns].reverse().map((c) => {
              const idx = checkIns.findIndex((x) => x.id === c.id);
              const prev = checkIns[idx - 1];
              const d = prev ? c.weight - prev.weight : 0;
              return (
                <li key={c.id}>
                  <Card className="gap-2 p-4">
                    <div className="flex items-start justify-between gap-2">
                      <div>
                        <p className="font-semibold">{fmtDate(c.date, { weekday: "short", day: "2-digit", month: "long", year: "numeric" })}</p>
                        <p className="text-sm tabular-nums">
                          <span className="font-bold">{fmt1(c.weight)} kg</span>
                          {prev && (
                            <span className={d <= 0 ? "ml-2 text-emerald-800 dark:text-emerald-300" : "ml-2 text-amber-800 dark:text-amber-300"}>
                              {signed1(d)} kg
                            </span>
                          )}
                          <span className="ml-3 text-muted-foreground">
                            Hüfte {fmt1(c.hip)} · Bauch {fmt1(c.waist)} cm
                          </span>
                        </p>
                      </div>
                      <Button
                        type="button"
                        variant="ghost"
                        size="icon"
                        aria-label={`Check-in vom ${fmtDate(c.date)} löschen`}
                        onClick={async () => {
                          if (await confirm({ title: "Check-in löschen?", description: "Dieser Eintrag wird für Coach und Klientin entfernt.", confirmLabel: "Löschen", destructive: true })) {
                            removeCheckIn(c.id);
                          }
                        }}
                      >
                        <Trash2 className="size-4" aria-hidden="true" />
                      </Button>
                    </div>
                    <div className="flex flex-wrap gap-1.5">
                      <Badge variant="secondary">Energie {c.energy}</Badge>
                      <Badge variant="secondary">Hunger {c.hunger}</Badge>
                      <Badge variant="secondary">Schlaf {c.sleep}</Badge>
                      <Badge variant="outline">{c.author === "coach" ? "vom Coach" : "von Elena"}</Badge>
                    </div>
                    {c.feedback && <p className="text-sm">{c.feedback}</p>}
                  </Card>
                </li>
              );
            })}
          </ol>
        </section>
      </div>
    </div>
  );
}
````

## Seite: Fortschritt

`src/pages/progress.tsx`

````tsx
import { useRef } from "react";
import { ImagePlus, Trash2 } from "lucide-react";
import { useHub } from "@/lib/store";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { PdfExportButton } from "@/components/pdf-export-button";
import { CircumferenceChart, DataTableDisclosure, WeightChart } from "@/components/charts";
import { fmt1, fmtDate, signed1, sortedCheckIns, TODAY } from "@/lib/calc";
import type { CheckIn, ProgressPhoto } from "@/lib/types";
import { toast } from "sonner";
import { saveFile } from "@/lib/file-store";

const avg = (list: CheckIn[], k: "energy" | "hunger" | "sleep") =>
  list.length ? list.reduce((a, c) => a + c[k], 0) / list.length : 0;

function PhotoSlot({ label, photos }: { label: ProgressPhoto["label"]; photos: ProgressPhoto[] }) {
  const addPhoto = useHub((s) => s.addPhoto);
  const removePhoto = useHub((s) => s.removePhoto);
  const inputId = `photo-${label}`;
  const latest = photos[photos.length - 1];
  const title = label === "vorher" ? "Vorher" : "Nachher";
  return (
    <div className="space-y-2">
      <div className="flex items-center justify-between">
        <h3 className="font-semibold">{title}</h3>
        {latest && (
          <Button type="button" variant="ghost" size="icon" aria-label={`${title}-Bild entfernen`} onClick={() => removePhoto(latest.id)}>
            <Trash2 className="size-4" aria-hidden="true" />
          </Button>
        )}
      </div>
      <div className="relative aspect-[3/4] overflow-hidden rounded-2xl border border-dashed border-border bg-muted/50">
        {latest ? (
          <>
            <img src={latest.url} alt={`${title}-Bild vom ${fmtDate(latest.date)}`} className="size-full object-cover" />
            <span className="absolute bottom-2 left-2 rounded-full bg-blue-950/85 px-2.5 py-1 text-xs font-medium text-white">
              {fmtDate(latest.date, { day: "2-digit", month: "2-digit", year: "numeric" })}
            </span>
          </>
        ) : (
          <label htmlFor={inputId} className="flex size-full cursor-pointer flex-col items-center justify-center gap-2 p-4 text-center text-sm text-muted-foreground hover:bg-muted">
            <ImagePlus className="size-8" aria-hidden="true" />
            Bild hochladen
          </label>
        )}
      </div>
      <input
        id={inputId}
        type="file"
        accept="image/*"
        className="sr-only"
        onChange={(e) => {
          const file = e.target.files?.[0];
          if (!file) return;
          addPhoto({ label, date: TODAY, url: URL.createObjectURL(file) });
          e.target.value = "";
        }}
      />
      {latest && (
        <Button asChild variant="outline" size="sm" className="w-full rounded-full">
          <label htmlFor={inputId} className="cursor-pointer">Bild ersetzen</label>
        </Button>
      )}
    </div>
  );
}

export function ProgressPage() {
  const profile = useHub((s) => s.profile);
  const raw = useHub((s) => s.checkIns);
  const photos = useHub((s) => s.photos);
  const addDocument = useHub((s) => s.addDocument);
  const reportRef = useRef<HTMLDivElement>(null);
  const checkIns = sortedCheckIns(raw);
  const first = checkIns[0];
  const last = checkIns[checkIns.length - 1];

  if (!first || !last) {
    return <p>Noch keine Check-ins vorhanden.</p>;
  }

  const weeks = Math.max(1, (new Date(last.date).getTime() - new Date(first.date).getTime()) / (7 * 864e5));
  const dW = last.weight - first.weight;
  const perWeek = dW / weeks;
  const dHip = last.hip - first.hip;
  const dWaist = last.waist - first.waist;
  const toGoal = last.weight - profile.goalWeight;
  const early = checkIns.slice(0, 4);
  const recent = checkIns.slice(-4);
  const weeksToGoal = perWeek < 0 ? Math.ceil(toGoal / Math.abs(perWeek)) : null;

  const stats = [
    { l: "Gewicht", v: `${signed1(dW)} kg`, s: `${fmt1(first.weight)} → ${fmt1(last.weight)} kg` },
    { l: "Ø pro Woche", v: `${signed1(perWeek)} kg`, s: `über ${Math.round(weeks)} Wochen` },
    { l: "Hüfte", v: `${signed1(dHip)} cm`, s: `${fmt1(first.hip)} → ${fmt1(last.hip)} cm` },
    { l: "Bauch", v: `${signed1(dWaist)} cm`, s: `${fmt1(first.waist)} → ${fmt1(last.waist)} cm` },
  ];

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-3">
        <h1 className="text-3xl font-bold tracking-tight">Fortschritt</h1>
        <div data-pdf-exclude>
          <PdfExportButton
            targetRef={reportRef}
            fileName={`Fortschrittsbericht_Elena_${TODAY}.pdf`}
            onGenerated={async ({ blob, fileName, byteLength }) => {
              const fileId = await saveFile(blob);
              addDocument({
                title: `Fortschrittsbericht ${fmtDate(TODAY, { day: "2-digit", month: "long", year: "numeric" })}`,
                category: "fortschrittsbericht",
                date: TODAY,
                addedBy: useHub.getState().role,
                description: `${signed1(dW)} kg, ${signed1(dHip)} cm Hüfte, ${signed1(dWaist)} cm Bauch seit Start.`,
                fileName,
                sizeKb: Math.round(byteLength / 1024),
                mimeType: "application/pdf",
                fileId,
              });
              toast.success("Bericht im Dokumentenbereich abgelegt");
            }}
          >
            Bericht als PDF
          </PdfExportButton>
        </div>
      </div>

      <div ref={reportRef} className="space-y-6 bg-background">
        <Card className="gap-4 bg-gradient-to-br from-blue-950 to-blue-800 p-5 text-white sm:p-6">
          <h2 className="text-lg font-bold">Zusammenfassung der Entwicklung</h2>
          <dl className="grid grid-cols-2 gap-4 lg:grid-cols-4">
            {stats.map((s) => (
              <div key={s.l}>
                <dt className="text-xs uppercase tracking-wider text-blue-200">{s.l}</dt>
                <dd className="text-2xl font-bold tabular-nums text-emerald-300">{s.v}</dd>
                <dd className="text-xs text-blue-100">{s.s}</dd>
              </div>
            ))}
          </dl>
          <p className="text-sm leading-relaxed text-blue-50">
            Seit dem {fmtDate(first.date, { day: "2-digit", month: "long" })} hat Elena {fmt1(Math.abs(dW))} kg verloren – im Schnitt{" "}
            {fmt1(Math.abs(perWeek))} kg pro Woche. Bis zum Zielgewicht von {fmt1(profile.goalWeight)} kg fehlen noch {fmt1(Math.max(0, toGoal))} kg
            {weeksToGoal !== null && toGoal > 0 ? `, bei gleichem Tempo etwa ${weeksToGoal} Wochen` : ""}. Energie im Schnitt{" "}
            {fmt1(avg(early, "energy"))} → {fmt1(avg(recent, "energy"))}, Hunger {fmt1(avg(early, "hunger"))} → {fmt1(avg(recent, "hunger"))}, Schlaf{" "}
            {fmt1(avg(early, "sleep"))} → {fmt1(avg(recent, "sleep"))} (erste vs. letzte 4 Check-ins).
          </p>
        </Card>

        <div className="grid gap-6 lg:grid-cols-2">
          <Card className="gap-1 p-5">
            <h2 className="text-lg font-bold">Gewicht</h2>
            <p className="mb-2 text-sm text-muted-foreground">
              {signed1(dW)} kg seit Start · Ziellinie {fmt1(profile.goalWeight)} kg
            </p>
            <WeightChart checkIns={checkIns} goal={profile.goalWeight} />
            <div data-pdf-exclude>
              <DataTableDisclosure summary="Werte als Tabelle" head={["Datum", "kg"]} rows={checkIns.map((c) => [fmtDate(c.date), fmt1(c.weight)])} />
            </div>
          </Card>
          <Card className="gap-1 p-5">
            <h2 className="text-lg font-bold">Umfänge</h2>
            <p className="mb-2 text-sm text-muted-foreground">
              Hüfte {signed1(dHip)} cm (durchgezogen) · Bauch {signed1(dWaist)} cm (gestrichelt)
            </p>
            <CircumferenceChart checkIns={checkIns} />
            <div data-pdf-exclude>
              <DataTableDisclosure
                summary="Werte als Tabelle"
                head={["Datum", "Hüfte (cm)", "Bauch (cm)"]}
                rows={checkIns.map((c) => [fmtDate(c.date), fmt1(c.hip), fmt1(c.waist)])}
              />
            </div>
          </Card>
        </div>
      </div>

      <Card className="gap-4 p-5 sm:p-6">
        <h2 className="text-lg font-bold">Vorher-Nachher-Bilder</h2>
        <div className="grid grid-cols-2 gap-4 sm:max-w-xl">
          <PhotoSlot label="vorher" photos={photos.filter((p) => p.label === "vorher")} />
          <PhotoSlot label="nachher" photos={photos.filter((p) => p.label === "nachher")} />
        </div>
        <p className="text-xs text-muted-foreground">Bilder bleiben nur in der aktuellen Sitzung sichtbar und werden nicht dauerhaft gespeichert.</p>
      </Card>
    </div>
  );
}
````

## Seite: Training

`src/pages/training.tsx`

````tsx
import { useState } from "react";
import { CartesianGrid, Line, LineChart, XAxis, YAxis } from "recharts";
import { CheckCircle2, Plus, Trash2 } from "lucide-react";
import { useHub } from "@/lib/store";
import { useConfirm } from "@/hooks/use-confirm";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { NativeSelect, NativeSelectOption } from "@/components/ui/native-select";
import { ChartContainer, ChartTooltip, ChartTooltipContent, type ChartConfig } from "@/components/ui/chart";
import { DataTableDisclosure } from "@/components/charts";
import { fmt0, fmt1, fmtDate, TODAY, workoutsThisWeek } from "@/lib/calc";
import { cn } from "@/lib/utils";
import { toast } from "sonner";

const chartConfig = { weight: { label: "Gewicht (kg)", color: "var(--color-primary)" } } satisfies ChartConfig;

export function TrainingPage() {
  const plans = useHub((s) => s.plans);
  const logs = useHub((s) => s.workoutLogs);
  const targets = useHub((s) => s.targets);
  const updateExercise = useHub((s) => s.updateExercise);
  const addExercise = useHub((s) => s.addExercise);
  const removeExercise = useHub((s) => s.removeExercise);
  const logWorkout = useHub((s) => s.logWorkout);
  const removeWorkoutLog = useHub((s) => s.removeWorkoutLog);
  const confirm = useConfirm();

  const [active, setActive] = useState<"A" | "B">("A");
  const plan = plans.find((p) => p.id === active)!;
  const [date, setDate] = useState(TODAY);
  const [note, setNote] = useState("");

  const allExercises = Array.from(new Set(logs.flatMap((l) => l.entries.filter((e) => e.weight > 0).map((e) => e.name))));
  const [exName, setExName] = useState(allExercises[0] ?? "");
  const series = logs
    .map((l) => ({ date: fmtDate(l.date), weight: l.entries.find((e) => e.name === exName)?.weight }))
    .filter((d): d is { date: string; weight: number } => typeof d.weight === "number");

  const weekCount = workoutsThisWeek(logs).length;

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-end justify-between gap-3">
        <h1 className="text-3xl font-bold tracking-tight">Training</h1>
        <p className="text-sm">
          Diese Woche{" "}
          <span className="font-bold tabular-nums">
            {weekCount}/{targets.workoutsPerWeek}
          </span>{" "}
          Ganzkörpertrainings
        </p>
      </div>

      <div role="tablist" aria-label="Trainingsplan" className="inline-flex rounded-full bg-muted p-1">
        {plans.map((p) => (
          <button
            key={p.id}
            role="tab"
            type="button"
            id={`tab-${p.id}`}
            aria-selected={active === p.id}
            aria-controls="plan-panel"
            onClick={() => setActive(p.id)}
            className={cn(
              "rounded-full px-4 py-2 text-sm font-semibold transition-colors focus-visible:outline-2 focus-visible:outline-ring",
              active === p.id ? "bg-blue-950 text-white" : "text-muted-foreground hover:text-foreground",
            )}
          >
            {p.name}
          </button>
        ))}
      </div>

      <Card id="plan-panel" role="tabpanel" aria-labelledby={`tab-${active}`} className="gap-4 p-5 sm:p-6">
        <div className="flex flex-wrap items-baseline justify-between gap-2">
          <h2 className="text-lg font-bold">{plan.name}</h2>
          <p className="text-sm text-muted-foreground">{plan.focus}</p>
        </div>
        <div className="overflow-x-auto">
          <table className="w-full min-w-[34rem] text-sm">
            <thead>
              <tr className="text-left text-xs uppercase tracking-wider text-muted-foreground">
                <th scope="col" className="pb-2 font-semibold">Übung</th>
                <th scope="col" className="w-20 pb-2 font-semibold">Sätze</th>
                <th scope="col" className="w-28 pb-2 font-semibold">Wdh.</th>
                <th scope="col" className="w-28 pb-2 font-semibold">Gewicht (kg)</th>
                <th scope="col" className="w-10 pb-2"><span className="sr-only">Aktionen</span></th>
              </tr>
            </thead>
            <tbody className="divide-y divide-border">
              {plan.exercises.map((e) => (
                <tr key={e.id}>
                  <td className="py-2 pr-2">
                    <Input aria-label="Übung" value={e.name} onChange={(ev) => updateExercise(plan.id, e.id, { name: ev.target.value })} className="h-9" />
                  </td>
                  <td className="py-2 pr-2">
                    <Input aria-label={`Sätze ${e.name}`} type="number" min={1} value={e.sets} onChange={(ev) => updateExercise(plan.id, e.id, { sets: Math.max(1, Number(ev.target.value)) })} className="h-9 tabular-nums" />
                  </td>
                  <td className="py-2 pr-2">
                    <Input aria-label={`Wiederholungen ${e.name}`} value={e.reps} onChange={(ev) => updateExercise(plan.id, e.id, { reps: ev.target.value })} className="h-9" />
                  </td>
                  <td className="py-2 pr-2">
                    <Input aria-label={`Gewicht ${e.name}`} type="number" min={0} step={0.5} value={e.weight} onChange={(ev) => updateExercise(plan.id, e.id, { weight: Math.max(0, Number(ev.target.value)) })} className="h-9 tabular-nums" />
                  </td>
                  <td className="py-2">
                    <Button type="button" variant="ghost" size="icon" aria-label={`${e.name} entfernen`} onClick={() => removeExercise(plan.id, e.id)}>
                      <Trash2 className="size-4" aria-hidden="true" />
                    </Button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
        <Button type="button" variant="outline" size="sm" className="self-start rounded-full" onClick={() => addExercise(plan.id)}>
          <Plus className="size-4" aria-hidden="true" /> Übung hinzufügen
        </Button>

        <form
          className="grid gap-3 rounded-2xl bg-emerald-50 p-4 text-emerald-950 sm:grid-cols-[10rem_1fr_auto] sm:items-end dark:bg-emerald-950 dark:text-emerald-50"
          onSubmit={(ev) => {
            ev.preventDefault();
            logWorkout({
              date,
              planId: plan.id,
              note: note.trim(),
              entries: plan.exercises.map((e) => ({ exerciseId: e.id, name: e.name, sets: e.sets, reps: e.reps, weight: e.weight })),
            });
            setNote("");
            toast.success(`${plan.name} eingetragen`);
          }}
        >
          <div className="space-y-1">
            <Label htmlFor="wdate">Datum</Label>
            <Input id="wdate" type="date" value={date} onChange={(e) => setDate(e.target.value)} className="bg-card text-foreground" />
          </div>
          <div className="space-y-1">
            <Label htmlFor="wnote">Notiz (optional)</Label>
            <Textarea id="wnote" rows={1} value={note} onChange={(e) => setNote(e.target.value)} className="min-h-9 bg-card text-foreground" />
          </div>
          <Button type="submit" className="rounded-full">
            <CheckCircle2 className="size-4" aria-hidden="true" /> Training abschließen
          </Button>
        </form>
      </Card>

      <div className="grid gap-6 lg:grid-cols-[1fr_1.2fr]">
        <Card className="gap-2 p-5">
          <h2 className="text-lg font-bold">Kraftentwicklung</h2>
          <div className="space-y-1">
            <Label htmlFor="exsel" className="text-xs text-muted-foreground">Übung</Label>
            <NativeSelect id="exsel" size="sm" value={exName} onChange={(e) => setExName(e.target.value)}>
              {allExercises.map((n) => (
                <NativeSelectOption key={n} value={n}>{n}</NativeSelectOption>
              ))}
            </NativeSelect>
          </div>
          {series.length > 0 ? (
            <>
              <p className="text-sm text-muted-foreground">
                {fmt1(series[0].weight)} → {fmt1(series[series.length - 1].weight)} kg in {series.length} Einheiten
              </p>
              <ChartContainer config={chartConfig} className="h-52 w-full">
                <LineChart data={series} margin={{ left: 0, right: 8, top: 8 }}>
                  <CartesianGrid vertical={false} strokeDasharray="3 3" />
                  <XAxis dataKey="date" tickLine={false} axisLine={false} minTickGap={16} />
                  <YAxis tickLine={false} axisLine={false} width={32} domain={["dataMin - 5", "dataMax + 5"]} />
                  <ChartTooltip content={<ChartTooltipContent />} />
                  <Line dataKey="weight" type="stepAfter" stroke="var(--color-weight)" strokeWidth={2.5} dot={{ r: 3 }} />
                </LineChart>
              </ChartContainer>
              <DataTableDisclosure summary="Werte als Tabelle" head={["Datum", "kg"]} rows={series.map((s) => [s.date, fmt1(s.weight)])} />
            </>
          ) : (
            <p className="text-sm text-muted-foreground">Noch keine Einträge für diese Übung.</p>
          )}
        </Card>

        <section className="space-y-3" aria-labelledby="th-h">
          <h2 id="th-h" className="text-lg font-bold">Trainingshistorie</h2>
          <ol className="space-y-2">
            {[...logs].reverse().map((l) => {
              const volume = l.entries.reduce((a, e) => a + e.sets * (parseInt(e.reps, 10) || 0) * e.weight, 0);
              return (
                <li key={l.id}>
                  <details className="group rounded-2xl border border-border bg-card">
                    <summary className="flex cursor-pointer items-center justify-between gap-2 rounded-2xl p-3 focus-visible:outline-2 focus-visible:outline-ring">
                      <span>
                        <span className="font-semibold">{l.planId === "A" ? "Ganzkörper A" : "Ganzkörper B"}</span>
                        <span className="ml-2 text-sm text-muted-foreground">
                          {fmtDate(l.date, { weekday: "short", day: "2-digit", month: "2-digit", year: "numeric" })}
                        </span>
                      </span>
                      <span className="text-xs tabular-nums text-muted-foreground">Volumen ca. {fmt0(volume)} kg</span>
                    </summary>
                    <div className="space-y-2 px-3 pb-3">
                      <ul className="text-sm">
                        {l.entries.map((e) => (
                          <li key={e.exerciseId} className="flex justify-between border-t border-border py-1.5">
                            <span>{e.name}</span>
                            <span className="tabular-nums text-muted-foreground">
                              {e.sets} × {e.reps}
                              {e.weight > 0 ? ` · ${fmt1(e.weight)} kg` : ""}
                            </span>
                          </li>
                        ))}
                      </ul>
                      {l.note && <p className="text-sm italic">{l.note}</p>}
                      <Button
                        type="button"
                        variant="ghost"
                        size="sm"
                        onClick={async () => {
                          if (await confirm({ title: "Eintrag löschen?", confirmLabel: "Löschen", destructive: true })) removeWorkoutLog(l.id);
                        }}
                      >
                        <Trash2 className="size-4" aria-hidden="true" /> Eintrag löschen
                      </Button>
                    </div>
                  </details>
                </li>
              );
            })}
          </ol>
        </section>
      </div>
    </div>
  );
}
````

## Seite: Dokumente

`src/pages/documents.tsx`

````tsx
import { useEffect, useState } from "react";
import {
  BookOpen,
  ClipboardList,
  Download,
  Eye,
  FileText,
  LineChart,
  Paperclip,
  Pencil,
  ShoppingCart,
  Trash2,
  Upload,
  Utensils,
} from "lucide-react";
import { useHub } from "@/lib/store";
import { useConfirm } from "@/hooks/use-confirm";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { NativeSelect, NativeSelectOption } from "@/components/ui/native-select";
import { Dialog, DialogContent, DialogDescription, DialogFooter, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { EmptyState } from "@/components/states";
import { fmtDate, TODAY } from "@/lib/calc";
import { deleteFile, loadFile, saveFile } from "@/lib/file-store";
import type { DocCategory, HubDocument } from "@/lib/types";
import { cn } from "@/lib/utils";
import { toast } from "sonner";

const cats: { id: DocCategory; label: string; icon: typeof FileText }[] = [
  { id: "ernaehrungsplan", label: "Ernährungsplan", icon: Utensils },
  { id: "einkaufsliste", label: "Einkaufslisten", icon: ShoppingCart },
  { id: "rezept", label: "Rezepte", icon: BookOpen },
  { id: "pdf", label: "PDFs", icon: FileText },
  { id: "fortschrittsbericht", label: "Fortschrittsberichte", icon: LineChart },
];

const ACCEPT = ".pdf,.jpg,.jpeg,.png,.webp,.doc,.docx,.xlsx,.txt";
const MAX_MB = 25;

async function storeUpload(file: File) {
  if (file.size > MAX_MB * 1024 * 1024) throw new Error(`Die Datei ist größer als ${MAX_MB} MB.`);
  const fileId = await saveFile(file);
  return {
    fileId,
    fileName: file.name,
    sizeKb: Math.max(1, Math.round(file.size / 1024)),
    mimeType: file.type || (file.name.toLowerCase().endsWith(".pdf") ? "application/pdf" : ""),
  };
}

function useFileUrl(fileId: string | undefined, mimeType?: string) {
  const [state, setState] = useState<{ url: string | null; missing: boolean }>({ url: null, missing: false });
  useEffect(() => {
    if (!fileId) return;
    let url: string | null = null;
    let alive = true;
    loadFile(fileId).then((blob) => {
      if (!alive) return;
      if (!blob) return setState({ url: null, missing: true });
      const typed = mimeType && blob.type !== mimeType ? new Blob([blob], { type: mimeType }) : blob;
      url = URL.createObjectURL(typed);
      setState({ url, missing: false });
    });
    return () => {
      alive = false;
      if (url) URL.revokeObjectURL(url);
    };
  }, [fileId, mimeType]);
  return state;
}

function Viewer({ doc, onClose }: { doc: HubDocument; onClose: () => void }) {
  const { url, missing } = useFileUrl(doc.fileId, doc.mimeType);
  const isPdf = doc.mimeType === "application/pdf" || doc.fileName?.toLowerCase().endsWith(".pdf");
  const isImg = doc.mimeType?.startsWith("image/");
  return (
    <Dialog open onOpenChange={(o) => !o && onClose()}>
      <DialogContent className="flex h-[90dvh] max-w-5xl flex-col gap-3 sm:max-w-5xl">
        <DialogHeader>
          <DialogTitle>{doc.title}</DialogTitle>
          <DialogDescription>{doc.fileName}</DialogDescription>
        </DialogHeader>
        <div className="min-h-0 flex-1 overflow-hidden rounded-xl border border-border bg-muted">
          {missing ? (
            <p className="p-6 text-sm">Die Datei ist auf diesem Gerät nicht gespeichert. Bitte erneut hochladen.</p>
          ) : !url ? (
            <p className="p-6 text-sm text-muted-foreground">Datei wird geladen …</p>
          ) : isPdf ? (
            <iframe src={url} title={doc.title} className="size-full bg-white" />
          ) : isImg ? (
            <img src={url} alt={doc.title} className="size-full object-contain" />
          ) : (
            <p className="p-6 text-sm">Für diesen Dateityp gibt es keine Vorschau. Bitte herunterladen.</p>
          )}
        </div>
        {url && (
          <DialogFooter className="flex-row flex-wrap gap-2 sm:justify-start">
            <Button asChild className="rounded-full">
              <a href={url} download={doc.fileName}>
                <Download className="size-4" aria-hidden="true" /> Herunterladen
              </a>
            </Button>
            <Button asChild variant="outline" className="rounded-full">
              <a href={url} target="_blank" rel="noopener noreferrer">
                In neuem Tab öffnen
              </a>
            </Button>
          </DialogFooter>
        )}
      </DialogContent>
    </Dialog>
  );
}

function Editor({ doc, onClose }: { doc: HubDocument; onClose: () => void }) {
  const updateDocument = useHub((s) => s.updateDocument);
  const [busy, setBusy] = useState(false);
  return (
    <Dialog open onOpenChange={(o) => !o && !busy && onClose()}>
      <DialogContent className="sm:max-w-lg">
        <DialogHeader>
          <DialogTitle>Dokument bearbeiten</DialogTitle>
          <DialogDescription>Titel, Kategorie, Beschreibung oder Datei ändern.</DialogDescription>
        </DialogHeader>
        <form
          className="space-y-3"
          onSubmit={async (e) => {
            e.preventDefault();
            const fd = new FormData(e.currentTarget);
            const file = fd.get("file");
            const patch: Partial<HubDocument> = {
              title: String(fd.get("title") ?? "").trim() || doc.title,
              category: String(fd.get("category")) as DocCategory,
              description: String(fd.get("description") ?? "").trim(),
            };
            try {
              setBusy(true);
              if (file instanceof File && file.size > 0) {
                const stored = await storeUpload(file);
                if (doc.fileId) await deleteFile(doc.fileId);
                Object.assign(patch, stored);
              }
              updateDocument(doc.id, patch);
              toast.success("Dokument aktualisiert");
              onClose();
            } catch (err) {
              toast.error(err instanceof Error ? err.message : "Speichern fehlgeschlagen");
            } finally {
              setBusy(false);
            }
          }}
        >
          <div className="space-y-1">
            <Label htmlFor="etitle">Titel</Label>
            <Input id="etitle" name="title" defaultValue={doc.title} />
          </div>
          <div className="space-y-1">
            <Label htmlFor="ecat">Kategorie</Label>
            <NativeSelect id="ecat" name="category" defaultValue={doc.category} className="w-full">
              {cats.map((c) => (
                <NativeSelectOption key={c.id} value={c.id}>{c.label}</NativeSelectOption>
              ))}
            </NativeSelect>
          </div>
          <div className="space-y-1">
            <Label htmlFor="edesc">Beschreibung</Label>
            <Textarea id="edesc" name="description" rows={3} defaultValue={doc.description} />
          </div>
          <div className="space-y-1">
            <Label htmlFor="efile">{doc.fileId ? "Datei ersetzen" : "Datei anhängen"}</Label>
            <Input id="efile" name="file" type="file" accept={ACCEPT} />
            {doc.fileName && <p className="text-xs text-muted-foreground">Aktuell: {doc.fileName}</p>}
          </div>
          <DialogFooter>
            <Button type="button" variant="ghost" onClick={onClose} disabled={busy}>Abbrechen</Button>
            <Button type="submit" className="rounded-full" disabled={busy}>
              {busy ? "Speichert …" : "Speichern"}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}

export function DocumentsPage() {
  const docs = useHub((s) => s.documents);
  const role = useHub((s) => s.role);
  const addDocument = useHub((s) => s.addDocument);
  const removeDocument = useHub((s) => s.removeDocument);
  const confirm = useConfirm();
  const [filter, setFilter] = useState<DocCategory | "alle">("alle");
  const [viewing, setViewing] = useState<HubDocument | null>(null);
  const [editing, setEditing] = useState<HubDocument | null>(null);
  const [uploading, setUploading] = useState(false);
  const shown = docs.filter((d) => filter === "alle" || d.category === filter).sort((a, b) => b.date.localeCompare(a.date));

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold tracking-tight">Dokumente</h1>

      <div className="grid gap-6 lg:grid-cols-[16rem_1fr]">
        <div className="space-y-4">
          <nav aria-label="Kategorien" className="no-scrollbar flex gap-2 overflow-x-auto lg:flex-col">
            {[{ id: "alle" as const, label: "Alle", icon: ClipboardList }, ...cats].map((c) => {
              const count = c.id === "alle" ? docs.length : docs.filter((d) => d.category === c.id).length;
              const Icon = c.icon;
              return (
                <button
                  key={c.id}
                  type="button"
                  aria-pressed={filter === c.id}
                  onClick={() => setFilter(c.id)}
                  className={cn(
                    "flex shrink-0 items-center gap-2 rounded-2xl px-3 py-2 text-sm font-medium transition-colors focus-visible:outline-2 focus-visible:outline-ring",
                    filter === c.id ? "bg-blue-950 text-white" : "bg-card text-foreground hover:bg-muted",
                  )}
                >
                  <Icon className="size-4" aria-hidden="true" />
                  {c.label}
                  <span className="ml-auto pl-2 text-xs tabular-nums opacity-80">{count}</span>
                </button>
              );
            })}
          </nav>

          <Card className="gap-3 p-4">
            <h2 className="font-bold">Dokument hochladen</h2>
            <form
              className="space-y-3"
              onSubmit={async (e) => {
                e.preventDefault();
                const form = e.currentTarget;
                const fd = new FormData(form);
                const file = fd.get("file");
                const title = String(fd.get("title") ?? "").trim();
                const f = file instanceof File && file.size > 0 ? file : null;
                if (!title && !f) {
                  toast.error("Bitte Titel oder Datei angeben.");
                  return;
                }
                try {
                  setUploading(true);
                  const stored = f ? await storeUpload(f) : {};
                  addDocument({
                    title: title || f!.name.replace(/\.[^.]+$/, ""),
                    category: String(fd.get("category")) as DocCategory,
                    date: TODAY,
                    addedBy: role,
                    description: String(fd.get("description") ?? "").trim(),
                    ...stored,
                  });
                  toast.success("Dokument hinzugefügt");
                  form.reset();
                } catch (err) {
                  toast.error(err instanceof Error ? err.message : "Hochladen fehlgeschlagen");
                } finally {
                  setUploading(false);
                }
              }}
            >
              <div className="space-y-1">
                <Label htmlFor="dfile">Datei (PDF, Bild, Word, Excel)</Label>
                <Input id="dfile" name="file" type="file" accept={ACCEPT} />
              </div>
              <div className="space-y-1">
                <Label htmlFor="dtitle">Titel</Label>
                <Input id="dtitle" name="title" placeholder="z. B. Rezept Protein-Pancakes" />
              </div>
              <div className="space-y-1">
                <Label htmlFor="dcat">Kategorie</Label>
                <NativeSelect id="dcat" name="category" defaultValue={filter === "alle" ? "rezept" : filter} key={filter} className="w-full">
                  {cats.map((c) => (
                    <NativeSelectOption key={c.id} value={c.id}>{c.label}</NativeSelectOption>
                  ))}
                </NativeSelect>
              </div>
              <div className="space-y-1">
                <Label htmlFor="ddesc">Beschreibung</Label>
                <Input id="ddesc" name="description" placeholder="z. B. 420 kcal · 35 g Protein" />
              </div>
              <Button type="submit" className="w-full rounded-full" disabled={uploading}>
                <Upload className="size-4" aria-hidden="true" /> {uploading ? "Lädt hoch …" : "Hochladen"}
              </Button>
            </form>
          </Card>
        </div>

        <section aria-label="Dokumentliste">
          {shown.length === 0 ? (
            <EmptyState icon={FileText} title="Keine Dokumente in dieser Kategorie" />
          ) : (
            <ul className="grid gap-3 sm:grid-cols-2">
              {shown.map((d) => {
                const cat = cats.find((c) => c.id === d.category) ?? cats[3];
                const Icon = cat.icon;
                return (
                  <li key={d.id}>
                    <Card className="h-full gap-3 p-4">
                      <div className="flex items-start gap-3">
                        <div className="grid size-11 shrink-0 place-items-center rounded-2xl bg-emerald-100 text-emerald-900" aria-hidden="true">
                          <Icon className="size-5" />
                        </div>
                        <div className="min-w-0 flex-1">
                          <h3 className="font-semibold leading-snug">{d.title}</h3>
                          <p className="text-xs text-muted-foreground">
                            {cat.label} · {fmtDate(d.date, { day: "2-digit", month: "2-digit", year: "numeric" })} · {d.addedBy === "coach" ? "Coach" : "Elena"}
                          </p>
                        </div>
                      </div>
                      {d.description && <p className="text-sm">{d.description}</p>}
                      <p className="min-w-0 truncate text-xs text-muted-foreground">
                        {d.fileId ? `${d.fileName ?? "Datei"}${d.sizeKb ? ` · ${d.sizeKb} KB` : ""}` : "Noch keine Datei angehängt"}
                      </p>
                      <div className="mt-auto flex flex-wrap items-center gap-1 border-t border-border pt-3">
                        {d.fileId ? (
                          <Button type="button" size="sm" className="rounded-full" onClick={() => setViewing(d)}>
                            <Eye className="size-4" aria-hidden="true" /> Ansehen
                          </Button>
                        ) : (
                          <Button type="button" size="sm" variant="outline" className="rounded-full" onClick={() => setEditing(d)}>
                            <Paperclip className="size-4" aria-hidden="true" /> Datei anhängen
                          </Button>
                        )}
                        <Button type="button" size="sm" variant="ghost" className="rounded-full" onClick={() => setEditing(d)}>
                          <Pencil className="size-4" aria-hidden="true" /> Bearbeiten
                        </Button>
                        <Button
                          type="button"
                          variant="ghost"
                          size="icon"
                          className="ml-auto"
                          aria-label={`${d.title} löschen`}
                          onClick={async () => {
                            if (await confirm({ title: "Dokument löschen?", description: d.title, confirmLabel: "Löschen", destructive: true })) {
                              if (d.fileId) await deleteFile(d.fileId);
                              removeDocument(d.id);
                            }
                          }}
                        >
                          <Trash2 className="size-4" aria-hidden="true" />
                        </Button>
                      </div>
                    </Card>
                  </li>
                );
              })}
            </ul>
          )}
        </section>
      </div>

      {viewing && <Viewer doc={viewing} onClose={() => setViewing(null)} />}
      {editing && <Editor key={editing.id} doc={editing} onClose={() => setEditing(null)} />}
    </div>
  );
}
````

## Seite: Coach-Notizen

`src/pages/coach-notes.tsx`

````tsx
import { useState } from "react";
import { Lock, Trash2 } from "lucide-react";
import { useHub } from "@/lib/store";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Checkbox } from "@/components/ui/checkbox";
import { EmptyState } from "@/components/states";
import { fmtDate, TODAY } from "@/lib/calc";
import type { CoachNoteType } from "@/lib/types";
import { cn } from "@/lib/utils";

const types: { id: CoachNoteType; label: string; tone: string }[] = [
  { id: "anpassung", label: "Anpassungen", tone: "bg-blue-950 text-white" },
  { id: "empfehlung", label: "Empfehlungen", tone: "bg-emerald-200 text-emerald-950" },
  { id: "naechster-schritt", label: "Nächste Schritte", tone: "bg-sky-100 text-sky-950" },
];

export function CoachNotesPage() {
  const role = useHub((s) => s.role);
  const notes = useHub((s) => s.coachNotes);
  const add = useHub((s) => s.addCoachNote);
  const toggle = useHub((s) => s.toggleCoachNote);
  const remove = useHub((s) => s.removeCoachNote);
  const setRole = useHub((s) => s.setRole);
  const [type, setType] = useState<CoachNoteType>("anpassung");
  const [text, setText] = useState("");

  if (role !== "coach") {
    return (
      <EmptyState
        icon={Lock}
        title="Nur für den Coach sichtbar"
        action={
          <Button variant="outline" className="rounded-full" onClick={() => setRole("coach")}>
            Als Coach ansehen
          </Button>
        }
      />
    );
  }

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center gap-3">
        <h1 className="text-3xl font-bold tracking-tight">Coach-Notizen</h1>
        <span className="inline-flex items-center gap-1 rounded-full bg-muted px-3 py-1 text-xs font-medium">
          <Lock className="size-3.5" aria-hidden="true" /> nur Coach
        </span>
      </div>

      <Card className="p-5">
        <form
          className="space-y-3"
          onSubmit={(e) => {
            e.preventDefault();
            if (!text.trim()) return;
            add({ type, text: text.trim(), date: TODAY });
            setText("");
          }}
        >
          <fieldset>
            <legend className="mb-2 text-sm font-semibold">Art der Notiz</legend>
            <div className="flex flex-wrap gap-2">
              {types.map((t) => (
                <button
                  key={t.id}
                  type="button"
                  aria-pressed={type === t.id}
                  onClick={() => setType(t.id)}
                  className={cn(
                    "rounded-full border px-3 py-1.5 text-sm font-medium focus-visible:outline-2 focus-visible:outline-ring",
                    type === t.id ? cn(t.tone, "border-transparent") : "border-border bg-card hover:bg-muted",
                  )}
                >
                  {t.label}
                </button>
              ))}
            </div>
          </fieldset>
          <div className="space-y-1">
            <Label htmlFor="note">Notiz</Label>
            <Textarea id="note" rows={3} value={text} onChange={(e) => setText(e.target.value)} />
          </div>
          <Button type="submit" className="rounded-full">Notiz speichern</Button>
        </form>
      </Card>

      <div className="grid gap-4 lg:grid-cols-3">
        {types.map((t) => {
          const list = notes.filter((n) => n.type === t.id).sort((a, b) => Number(a.done) - Number(b.done) || b.date.localeCompare(a.date));
          return (
            <section key={t.id} aria-labelledby={`h-${t.id}`} className="space-y-3">
              <h2 id={`h-${t.id}`} className="flex items-center justify-between">
                <span className={cn("rounded-full px-3 py-1 text-sm font-bold", t.tone)}>{t.label}</span>
                <span className="text-xs tabular-nums text-muted-foreground">{list.filter((n) => !n.done).length} offen</span>
              </h2>
              <ul className="space-y-2">
                {list.map((n) => (
                  <li key={n.id}>
                    <Card className={cn("gap-2 p-4", n.done && "opacity-75")}>
                      <div className="flex items-start gap-3">
                        <Checkbox id={`n-${n.id}`} checked={n.done} onCheckedChange={() => toggle(n.id)} className="mt-0.5" />
                        <label htmlFor={`n-${n.id}`} className={cn("flex-1 text-sm leading-relaxed", n.done && "line-through")}>
                          {n.text}
                        </label>
                        <Button type="button" variant="ghost" size="icon" aria-label="Notiz löschen" onClick={() => remove(n.id)}>
                          <Trash2 className="size-4" aria-hidden="true" />
                        </Button>
                      </div>
                      <p className="pl-7 text-xs text-muted-foreground">{fmtDate(n.date, { day: "2-digit", month: "2-digit", year: "numeric" })}</p>
                    </Card>
                  </li>
                ))}
                {list.length === 0 && <li className="text-sm text-muted-foreground">Keine Einträge.</li>}
              </ul>
            </section>
          );
        })}
      </div>
    </div>
  );
}
````

## Design-Farben (Theme)

`src/index.css`

````css
@import "tailwindcss";
@import "tw-animate-css";
@import "@fontsource-variable/geist";

@custom-variant data-open {
  &:where([data-state="open"]),
  &:where([data-open]:not([data-open="false"])) { @slot; }
}
@custom-variant data-closed {
  &:where([data-state="closed"]),
  &:where([data-closed]:not([data-closed="false"])) { @slot; }
}
@custom-variant data-checked {
  &:where([data-state="checked"]),
  &:where([data-checked]:not([data-checked="false"])) { @slot; }
}
@custom-variant data-unchecked {
  &:where([data-state="unchecked"]),
  &:where([data-unchecked]:not([data-unchecked="false"])) { @slot; }
}
@custom-variant data-selected {
  &:where([data-selected="true"]) { @slot; }
}
@custom-variant data-disabled {
  &:where([data-state="disabled"]),
  &:where([data-disabled]:not([data-disabled="false"])) { @slot; }
}
@custom-variant data-active {
  &:where([data-state="active"]),
  &:where([data-active]:not([data-active="false"])) { @slot; }
}
@custom-variant data-horizontal {
  &:where([data-orientation="horizontal"]) { @slot; }
}
@custom-variant data-vertical {
  &:where([data-orientation="vertical"]) { @slot; }
}

@utility no-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}

@custom-variant dark (&:is(.dark *));

:root {
  --background: oklch(0.975 0 265);
  --foreground: oklch(0.145 0 0);
  --card: oklch(0.993 0.002 265);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(0.993 0.002 265);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(.47 .16 265);
  --primary-foreground: oklch(.99 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.47 0 0);
  --accent: oklch(.90 .04 265);
  --accent-foreground: oklch(.38 .13 265);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.84 0.045 265);
  --input: oklch(0.84 0.045 265);
  --ring: oklch(.47 .16 265);
  --radius: 1.25rem;
  --chart-1: oklch(.6 .1 265);
  --chart-2: oklch(.65 .1 200);
  --chart-3: oklch(.6 .14 150);
  --chart-4: oklch(.65 .13 70);
  --chart-5: oklch(.6 .16 25);
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.145 0 0);
  --sidebar-primary: oklch(0.205 0 0);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.97 0 0);
  --sidebar-accent-foreground: oklch(0.205 0 0);
  --sidebar-border: oklch(0.922 0 0);
  --sidebar-ring: oklch(0.556 0 0);
}

body {
  background-color: var(--background);
  color: var(--foreground);
  margin: 0;
}

@theme inline {
  --font-heading: var(--font-sans);
  --font-sans: 'Geist Variable', sans-serif;
  --color-sidebar-ring: var(--sidebar-ring);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar: var(--sidebar);
  --color-chart-5: var(--chart-5);
  --color-chart-4: var(--chart-4);
  --color-chart-3: var(--chart-3);
  --color-chart-2: var(--chart-2);
  --color-chart-1: var(--chart-1);
  --color-ring: var(--ring);
  --color-input: var(--input);
  --color-border: var(--border);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-accent-foreground: var(--accent-foreground);
  --color-accent: var(--accent);
  --color-muted-foreground: var(--muted-foreground);
  --color-muted: var(--muted);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-secondary: var(--secondary);
  --color-primary-foreground: var(--primary-foreground);
  --color-primary: var(--primary);
  --color-popover-foreground: var(--popover-foreground);
  --color-popover: var(--popover);
  --color-card-foreground: var(--card-foreground);
  --color-card: var(--card);
  --color-foreground: var(--foreground);
  --color-background: var(--background);
  --radius-sm: calc(var(--radius) * 0.6);
  --radius-md: calc(var(--radius) * 0.8);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) * 1.4);
  --radius-2xl: calc(var(--radius) * 1.8);
  --radius-3xl: calc(var(--radius) * 2.2);
  --radius-4xl: calc(var(--radius) * 2.6);
}

.dark {
  --background: oklch(0.13 0.025 265);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.19 0.03 265);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.19 0.03 265);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(.63 .16 265);
  --primary-foreground: oklch(.15 .01 265);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(.26 .05 265);
  --accent-foreground: oklch(.9 .03 265);
  --destructive: oklch(0.704 0.191 22.216);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.34 0.05 265);
  --input: oklch(0.34 0.05 265);
  --ring: oklch(.63 .16 265);
  --chart-1: oklch(.6 .1 265);
  --chart-2: oklch(.65 .1 200);
  --chart-3: oklch(.6 .14 150);
  --chart-4: oklch(.65 .13 70);
  --chart-5: oklch(.6 .16 25);
  --sidebar: oklch(0.205 0 0);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.269 0 0);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.556 0 0);
}

/*
 * Auto-dark via OS preference (issue aether#7883).
 *
 * The home page renders before any user code can wire a dark-mode class
 * onto <html>, so it inherits :root's pure-white --background. When cowork
 * is itself in dark mode, the iframe pops out as a white block inside dark
 * chrome.
 *
 * Mirror the .dark vars under prefers-color-scheme: dark so the initial
 * paint (and any user code that hasn't wired its own toggle yet)
 * picks up the user agent's color scheme automatically. The :not(.light)
 * carve-out preserves an explicit-light escape hatch: user code that
 * forces <html class="light"> wins over the OS preference.
 *
 * Explicit <html class="dark"> remains the highest-priority signal: this
 * @media block targets `:root:not(.light)`, so user code that sets
 * <html class="light"> opts out of OS-dark, and an explicit `.dark` toggle
 * still themes correctly (the OS-dark block and the `.dark` class agree).
 */
@media (prefers-color-scheme: dark) {
  :root:not(.light) {
    --background: oklch(0.13 0.025 265);
    --foreground: oklch(0.985 0 0);
    --card: oklch(0.19 0.03 265);
    --card-foreground: oklch(0.985 0 0);
    --popover: oklch(0.19 0.03 265);
    --popover-foreground: oklch(0.985 0 0);
    --primary: oklch(.63 .16 265);
    --primary-foreground: oklch(.15 .01 265);
    --secondary: oklch(0.269 0 0);
    --secondary-foreground: oklch(0.985 0 0);
    --muted: oklch(0.269 0 0);
    --muted-foreground: oklch(0.708 0 0);
    --accent: oklch(.26 .05 265);
    --accent-foreground: oklch(.9 .03 265);
    --destructive: oklch(0.704 0.191 22.216);
    --destructive-foreground: oklch(0.985 0 0);
    --border: oklch(0.34 0.05 265);
    --input: oklch(0.34 0.05 265);
    --ring: oklch(.63 .16 265);
    --chart-1: oklch(.6 .1 265);
    --chart-2: oklch(.65 .1 200);
    --chart-3: oklch(.6 .14 150);
    --chart-4: oklch(.65 .13 70);
    --chart-5: oklch(.6 .16 25);
    --sidebar: oklch(0.205 0 0);
    --sidebar-foreground: oklch(0.985 0 0);
    --sidebar-primary: oklch(0.488 0.243 264.376);
    --sidebar-primary-foreground: oklch(0.985 0 0);
    --sidebar-accent: oklch(0.269 0 0);
    --sidebar-accent-foreground: oklch(0.985 0 0);
    --sidebar-border: oklch(1 0 0 / 10%);
    --sidebar-ring: oklch(0.556 0 0);
  }
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
  }
  html {
    @apply font-sans;
  }
}
````
