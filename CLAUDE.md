# CLAUDE.md — Conventions du projet docs-md

Ce dépôt contient une documentation personnelle technique (React, React Native, Symfony, JavaScript...), servie en local via `docsify` et destinée à être consultable en ligne via GitHub.

## Dépôt Git

- Remote GitHub configuré : `git remote add github git@github.com:42morrow/docs-md.git`
- Remote `origin` : hébergement OVH (`ftp.cluster010.hosting.ovh.net`), qui sert la doc en ligne. Ce remote est inaccessible en push depuis cette machine (pas de clé SSH acceptée) — c'est le serveur OVH qui doit **puller** depuis GitHub, à la main, pas l'inverse.
- Le dossier local est `docs-md/`, racine du dépôt.
- **Après tout `git push github master`** (ou tout commit destiné à être publié), terminer la réponse par un rappel : *"N'oublie pas de puller depuis OVH (/homez.2187/morrow/docs-md)"*.

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

6. **Tout nouveau document doit exister en deux formats : `.md` ET `.pdf`**. Le `.md` permet la lecture en ligne dans docsify, le `.pdf` permet la lecture hors-ligne. Conversion via Chrome headless (pandoc non disponible sur la machine) :
   ```bash
   # 1. Convertir le .md en HTML stylisé via Python
   python3 -c "import markdown, pathlib; ..."  # voir session précédente
   # 2. Convertir le HTML en PDF
   google-chrome --headless --disable-gpu --print-to-pdf=fichier.pdf --print-to-pdf-no-header fichier.html
   ```

7. **Format d'entrée dans `_sidebar.md` pour les docs dual-format** (titre non-cliquable + liens md et pdf sur la même ligne) :
   ```markdown
   - <span class="bvp-item">Titre du document&nbsp;<a class="bvp-link" href="#/dossier/fichier">md</a>&nbsp;<a class="bvp-link" href="dossier/fichier.pdf">pdf</a></span>
   ```
   Les classes `.bvp-item` et `.bvp-link` sont définies dans `index.html` (vert `#42b983`, `display:inline !important` pour contrer le `display:block` du thème docsify).

8. **Tous les chemins dans `_sidebar.md` doivent être relatifs** (sans `/` initial). Un chemin absolu comme `/react/fichier.pdf` casse sur GitHub Pages qui sert le dépôt depuis le sous-chemin `/docs-md/`.

9. **`README.md` racine = page d'accueil statique**, message de bienvenue uniquement. Ne jamais y ajouter de liens vers les documents — toute la navigation passe par `_sidebar.md`.

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
- `backups/rsync-autres-backups.md` + `backups/claude_guide_rsync_autres_backups.pdf` — Documentation des sauvegardes locales de contenus distants via scripts rsync (principe miroir distant→local, exemple Mallozzi_images, procédure d'ajout d'un nouveau rsync).
- `symfony/README.md` + `symfony/claude_guide_symfony.pdf` — Doc Symfony (projet Synoptic), **fichier unique organisé en sections** (pas un dossier multi-fichiers comme react/backups/bvp) pour garder une seule ligne dans la sidebar ; 1ère section : cache et environnements multi-tenant (Symfony Runtime / Doctrine `when@` / Twig auto_reload / piège APP_DEBUG en CLI / nom de classe container incluant le flag debug).

*Mettre à jour cette liste à chaque nouveau document ajouté, pour qu'une future session Claude Code ait le contexte de ce qui existe déjà et évite les doublons.*
