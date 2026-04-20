# TP Jour 1 — Chirpy : layout statique du feed

**Durée : 1 h 15** · **Livrable : votre `examples/chirpy/frontend/` complété** · **Solution de référence : `examples/chirpy/solutions/jour-1/`**

## Objectif

Construire le **layout statique** de la page d'accueil de Chirpy à partir d'une liste de posts fournie en dur. **Aucun `useState`** ne doit apparaître dans votre code : la journée porte sur composants & props, pas sur l'état.

## Ce que vous devez obtenir

Une page unique avec :

```text
┌───────────────────────────────────────────────────────────┐
│  Chirpy                                       [connexion] │   ← Header
├──────────────────┬────────────────────────────────────────┤
│                  │  ┌─────────────────────────────────┐   │
│  Qui suivre      │  │ Alice Martin · il y a 7 j       │   │
│  · Bob Durand    │  │ Premier post sur Chirpy ! 🎉    │   │
│  · Charlie D.    │  │ ♥ 2                              │   │
│                  │  └─────────────────────────────────┘   │
│   (Sidebar)      │  ┌─────────────────────────────────┐   │
│                  │  │ Bob Durand · il y a 6 j         │   │
│                  │  │ Aujourd'hui : debug de 3h...    │   │
│                  │  │ ♥ 1                              │   │
│                  │  └─────────────────────────────────┘   │
│                  │         (Feed + PostCard × N)          │
└──────────────────┴────────────────────────────────────────┘
```

## Contraintes techniques

1. **4 composants minimum** dans `src/components/` :
   - `Header.tsx`
   - `Sidebar.tsx`
   - `Feed.tsx`
   - `PostCard.tsx`
2. **Types** définis dans `src/types/post.ts` (fourni, ne pas modifier).
3. **`interface` dédiée** par composant pour ses props.
4. **Aucun `any`** dans votre code. TS strict doit passer (`npm run typecheck`).
5. **Aucun `useState`**, **aucun `useEffect`**. Juste des fonctions qui reçoivent des props.
6. **Données** : utilisez la constante `POSTS` fournie dans `src/data/seed.ts` (ci-dessous).

## Données à utiliser

Créez `src/data/seed.ts` avec :

```tsx
import type { Post } from "../types/post";

export const CURRENT_USER = {
  id: "u-alice",
  username: "alice",
  displayName: "Alice Martin",
  bio: "",
  createdAt: "2026-01-15T10:00:00Z",
};

export const POSTS: Post[] = [
  {
    id: "p-1",
    author: CURRENT_USER,
    content: "Premier post sur Chirpy ! 🎉",
    createdAt: "2026-04-12T10:00:00Z",
    likes: 2,
    likedByMe: false,
  },
  {
    id: "p-2",
    author: {
      id: "u-bob",
      username: "bob",
      displayName: "Bob Durand",
      bio: "",
      createdAt: "2026-01-15T10:00:00Z",
    },
    content:
      "Aujourd'hui : debug de 3h pour un Thread.sleep(100) oublié. RIP.",
    createdAt: "2026-04-13T10:00:00Z",
    likes: 1,
    likedByMe: true,
  },
  {
    id: "p-3",
    author: {
      id: "u-charlie",
      username: "charlie",
      displayName: "Charlie Dubois",
      bio: "",
      createdAt: "2026-01-15T10:00:00Z",
    },
    content:
      "", // ← intentionnellement vide, testez votre placeholder
    createdAt: "2026-04-14T10:00:00Z",
    likes: 0,
    likedByMe: false,
  },
  {
    id: "p-4",
    author: {
      id: "u-longname",
      username: "un_utilisateur_avec_un_tres_long_nom",
      displayName: "Un Utilisateur Avec Un Très Long Nom De Démonstration",
      bio: "",
      createdAt: "2026-01-15T10:00:00Z",
    },
    content: "Mon nom est volontairement interminable — testez votre layout.",
    createdAt: "2026-04-15T10:00:00Z",
    likes: 0,
    likedByMe: false,
  },
  {
    id: "p-5",
    author: CURRENT_USER,
    content: "Rappel : une key={index} cache un bug tant qu'on ne réordonne pas la liste.",
    createdAt: "2026-04-16T10:00:00Z",
    likes: 5,
    likedByMe: false,
  },
];
```

## Pièges à gérer

- **Post vide** : afficher *« (Contenu supprimé) »* en italique grisé, pas une chaîne vide.
- **Nom d'auteur très long** : ne doit pas casser le layout (CSS `text-overflow: ellipsis` ou `word-break`).
- **Date** : formatez avec `new Intl.DateTimeFormat("fr-FR", { dateStyle: "medium" })` ou équivalent. **Pas** `toString()` brut.

## Organisation conseillée

```tsx
// src/App.tsx
import Header from "./components/Header";
import Sidebar from "./components/Sidebar";
import Feed from "./components/Feed";
import { POSTS } from "./data/seed";

export default function App() {
  return (
    <>
      <Header />
      <div className="layout">
        <Sidebar />
        <Feed posts={POSTS} />
      </div>
    </>
  );
}
```

## Critères de validation

| | Critère |
|---|---|
| ✅ | `npm run dev` ouvre la page à `http://localhost:5173` sans erreur. |
| ✅ | Les 5 posts sont visibles. |
| ✅ | Le post avec contenu vide affiche un placeholder (pas de blanc suspect). |
| ✅ | Le nom long ne déborde pas. |
| ✅ | Dates formatées en français (« 12 avr. 2026 » ou similaire). |
| ✅ | `npm run build` passe sans erreur TS. |
| ✅ | Aucun `useState`, aucun `useEffect`. |
| ✅ | `key` sur la liste = `post.id`, **pas** `index`. |

## Bonus (si fini avant 1h15)

- Ajoutez un composant `PostHeader` (avatar stub + nom + date) et composez-le dans `PostCard`.
- Ajoutez un composant `LikeBadge` affichant `♥ 42` avec un style différent si `likedByMe`.
- Écrivez un composant `Card` générique avec `children` et utilisez-le dans `PostCard`.

## Aide

Bloqué ? Commencez petit :

1. Faites `PostCard` afficher **un seul** post hardcodé.
2. Puis rendez-le paramétrable via `props`.
3. Puis faites `Feed` qui mappe `posts`.
4. Puis ajoutez `Header` et `Sidebar`.

Toujours bloqué ? Comparez avec la solution de référence :

```bash
diff -ru ../solutions/jour-1/src src/
```

Ou ouvrez les fichiers dans `examples/chirpy/solutions/jour-1/src/` côte à côte avec les vôtres.
