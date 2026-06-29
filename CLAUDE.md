# CLAUDE.md — Conventions du projet docs-md

Ce dépôt contient une documentation personnelle technique (React, React Native, Symfony, JavaScript...), servie en local via `docsify` et destinée à être consultable en ligne via GitHub.

## Dépôt Git

- Remote GitHub configuré : `git remote add github git@github.com:42morrow/docs-md.git`
- Le dossier local est `docs-md/`, racine du dépôt.

## Structure du projet

```
docs-md/
├── index.html       (généré par `docsify init .`, NE PAS regénérer sans vérifier)
├── README.md        (page d'accueil, sommaire général des sections)
├── _sidebar.md       (menu de navigation latéral, source de vérité pour la nav)
├── react/
│   ├── README.md
│   └── *.md / *.pdf
├── symfony/
│   ├── README.md
│   └── *.md / *.pdf
├── javascript/
│   ├── README.md
│   └── *.md / *.pdf
└── ... (un dossier par thème futur)
```

## Règles à respecter pour toute nouvelle doc

1. **Chaque thème a son propre dossier**, nommé en minuscules (`react`, `symfony`, `javascript`, etc.).

2. **Chaque dossier de thème DOIT contenir un `README.md`**, même minimal, par exemple :
   ```markdown
   # Docs Symfony

   - [Nom du document](nom-fichier.md)
   ```
   Sans ce `README.md`, cliquer sur l'entrée de section dans `docsify` mène à un 404 (comportement par défaut de docsify : il cherche un `README.md` comme point d'entrée de chaque dossier).

3. **Tout nouveau document doit être référencé dans `_sidebar.md`** (à la racine), avec un lien relatif depuis la racine. Exemple :
   ```markdown
   - React
     - [Aide-mémoire React Native](react/aide-memoire-react-native.md)
   - JavaScript
     - [Promises & async/await](javascript/claude_guide_promises_async_await.pdf ':ignore')
   ```

4. **Tout fichier PDF (ou tout fichier binaire non-Markdown) doit utiliser la syntaxe `:ignore`** dans son lien, pour empêcher `docsify` de l'intercepter via son routing par hash et de générer un faux 404 :
   ```markdown
   - [Nom du document (PDF)](chemin/vers/fichier.pdf ':ignore')
   ```

5. **Convention de nommage des PDF générés par Claude** : préfixe `claude_guide_` suivi du sujet en snake_case, par exemple `claude_guide_react_hooks.pdf`, `claude_guide_promises_async_await.pdf`.

## À savoir sur le service local (pour mémoire)

- Le service `docsify serve` tourne en local via un service systemd utilisateur : `~/.config/systemd/user/docsify-docs.service`, sur le port `6419`.
- Accès local : `http://localhost:6419`
- Commandes utiles :
  ```bash
  systemctl --user status docsify-docs.service
  systemctl --user restart docsify-docs.service
  ```

## Historique des documents déjà produits

- `react/aide-memoire-react-native.md` — équivalences ReactJS ↔ React Native, navigation, style.
- `react/claude_guide_react_native.pdf` — guide complet React (fondamentaux/hooks) + spécificités React Native.
- `react/claude_guide_react_hooks.pdf` — guide approfondi sur les 6 hooks de base (useState, useEffect, useContext, useMemo, useCallback, useRef).
- `javascript/claude_guide_promises_async_await.pdf` — Promises JS, usage pratique + approfondissement théorique sur async/await (event loop, microtasks/macrotasks).
- `bvp/bvp-app-documentation.md` + `bvp/claude_guide_bvp_react_native.pdf` — Documentation technique et fonctionnelle de l'app BVP React Native (migration Cordova → Expo SDK 54) : architecture, DB SQLite, API Symfony, synchro, écrans, env de dev, roadmap.

*Mettre à jour cette liste à chaque nouveau document ajouté, pour qu'une future session Claude Code ait le contexte de ce qui existe déjà et évite les doublons.*
