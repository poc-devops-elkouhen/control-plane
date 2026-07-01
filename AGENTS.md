# AGENTS.md — control-plane

## Rôle du dépôt

`control-plane` orchestre tous les dépôts du POC. C'est le point d'entrée
unique pour provisionner l'environnement complet : cluster, plateforme et seed.
Il ne contient pas de code exécuté en production.

## Source de vérité

**`platform.yml` est la seule source de vérité pour les versions et les chemins.**
Toute modification de version (Kubernetes, ArgoCD, Helm charts, template CI...)
doit passer par ce fichier. `scripts/export-env.py` le transforme en variables
shell exportées et mémorisées dans `.control-plane.env`.

## Commandes principales

```bash
make env               # Afficher les variables exportées (sans les appliquer)
make platform-up       # Tout provisionner depuis zéro (images Packer + cluster + bootstrap)
make platform-fast-up  # Cluster depuis images Packer existantes + bootstrap
make cluster-up        # Cluster seul (sans images Packer)
make platform-bootstrap # Bootstrap ArgoCD + plateforme seule
make platform-bootstrap START_AT=gitlab-tf-credentials # Reprendre à une étape
make gitlab-tf-credentials # Créer/rotater le PAT GitLab Terraform
make status            # État ArgoCD
```

## Workflow Git

Ne jamais modifier les fichiers directement dans GitLab. Toujours :
1. Modifier en local.
2. Committer localement.
3. Pousser vers les deux remotes : `git push origin main` puis `git push gitlab main`.

## Ce qu'il ne faut pas faire

- Ne pas modifier les dépôts voisins (`cluster`, `platform-cicd`, etc.)
  directement depuis ce dépôt — passer par leurs propres Makefiles.
- Ne pas hardcoder des valeurs de versions dans les Makefiles ; les lire depuis
  les variables exportées par `export-env.py`.
- Ne pas committer `.control-plane.env` (contient des chemins locaux absolus).
