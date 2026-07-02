# Autres_backups

Ce répertoire centralise des sauvegardes locales de contenus hébergés à
distance (o2switch, etc.), synchronisées via des scripts `rsync.sh`.

Pour que la doc Autres_backups/CLAUDE.md soit recopiée dans le répertoire
docs-md, lancer claude depuis /media/laurent/42morrow_4To_backups, c'est dans
ce répertoire que se trouve le hook de recopie.

## Principe général

Chaque sous-répertoire synchronisé fonctionne en **miroir distant -> local** :
le serveur distant fait foi. Concrètement :

- un fichier supprimé côté distant est supprimé en local (`--delete`) ;
- un fichier présent côté distant mais absent en local est restauré ;
- un fichier créé uniquement en local (jamais poussé côté distant) sera donc
  effacé au prochain passage, puisqu'il n'a pas d'équivalent sur le serveur.

Ce sens de synchronisation a été choisi car le contenu de référence vit sur
le serveur distant ; le répertoire local n'est qu'une copie de secours.

## rsync.sh - Mallozzi_images

- **Local** : `Autres_backups/Mallozzi_images/`
- **Distant** : `arre1332@pacanier.o2switch.net:/home/arre1332/connivence/public/images_articles_v2/mallozzi/`
- **Authentification** : clé SSH `getyourtraveling_rsa`, déjà configurée
  pour l'hôte `pacanier.o2switch.net` dans `~/.ssh/config`.

Simulation avant tout transfert réel (recommandé) :

```bash
./rsync.sh --dry-run
```

Exécution réelle :

```bash
./rsync.sh
```

Ce script a été validé sur un couple de répertoires de test
(`test_backup_via_rsync` en local / `test_backup_via_rsync_remote` en
distant) avant d'être appliqué à `Mallozzi_images`.

```bash
#!/usr/bin/env bash
#
# Miroir distant (o2switch) -> local du dossier Mallozzi_images.
# Le DISTANT fait foi : un fichier supprime cote distant est supprime en
# local, un fichier absent en local mais present cote distant est restaure.
#
# Usage:
#   ./rsync.sh          synchronisation reelle
#   ./rsync.sh --dry-run   simulation, aucun fichier modifie/supprime

set -euo pipefail

LOCAL_DIR="/media/laurent/42morrow_4To_backups/Autres_backups/Mallozzi_images/"
REMOTE_USER="arre1332"
REMOTE_HOST="pacanier.o2switch.net"
REMOTE_DIR="/home/arre1332/connivence/public/images_articles_v2/mallozzi/"

RSYNC_OPTS=(-avh --delete --progress --stats)

if [[ "${1:-}" == "--dry-run" ]]; then
    RSYNC_OPTS+=(--dry-run)
    echo "== Mode simulation (dry-run) =="
fi

rsync "${RSYNC_OPTS[@]}" \
    -e "ssh -o IdentitiesOnly=yes" \
    "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}" \
    "$LOCAL_DIR"
```

## Ajouter un nouveau rsync

Pour un nouveau contenu distant à sauvegarder en local, dupliquer le
principe de `rsync.sh` :

1. Créer un sous-répertoire local dédié dans `Autres_backups/`.
2. Créer un script `rsync_<nom>.sh` sur le même modèle (variables
   `LOCAL_DIR`, `REMOTE_USER`, `REMOTE_HOST`, `REMOTE_DIR`, option
   `--dry-run`).
3. Toujours tester d'abord avec `--dry-run` pour valider ce qui serait
   créé/supprimé avant une exécution réelle, en particulier à cause du
   `--delete` qui peut effacer des fichiers locaux.
4. Documenter le nouveau script dans une section dédiée ci-dessous, sur le
   modèle de la section « rsync.sh - Mallozzi_images ».

<!-- Futurs rsync à documenter ici -->
