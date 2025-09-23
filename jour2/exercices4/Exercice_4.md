# Exercice 4 — Intégration du Build OpenShift dans un pipeline CI/CD (GitHub Actions)

## Objectif

Automatiser le build et le déploiement d’une application sur OpenShift à chaque push ou PR sur GitHub, via un pipeline CI/CD.

---

## Prérequis

- Un projet OpenShift prêt avec un BuildConfig (exemple : `eshoponweb` ou `app-s2i`).
- Un repository GitHub (fork du projet ou ton propre projet).
- Un jeton (token) `oc` (service account) permettant à GitHub d’interagir avec OpenShift (à stocker dans les “Secrets” GitHub).

---

## Étapes détaillées

### 1. Générer un token d’accès OpenShift

Sur OpenShift, crée un ServiceAccount dédié, assigne-lui les droits de build, puis récupère le token :

```bash
oc create sa github-cicd
oc policy add-role-to-user edit -z github-cicd
oc sa get-token github-cicd
````

* Copie ce token, il sera utilisé côté GitHub.

---

### 2. Ajouter le token comme secret GitHub

* Dans ton repo GitHub :
  **Settings > Secrets and variables > Actions > New repository secret**
* Ajoute un secret nommé `OPENSHIFT_TOKEN` (et éventuellement `OPENSHIFT_SERVER`, `OPENSHIFT_PROJECT`).

---

### 3. Ajouter le workflow GitHub Actions `.github/workflows/openshift-build.yml`

```yaml
name: Build and Deploy to OpenShift

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install OC CLI
        run: |
          curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz
          tar -xzf openshift-client-linux.tar.gz
          sudo mv oc kubectl /usr/local/bin/

      - name: Login to OpenShift
        env:
          OPENSHIFT_TOKEN: ${{ secrets.OPENSHIFT_TOKEN }}
        run: |
          oc login https://api.<ton-cluster>:6443 --token="$OPENSHIFT_TOKEN" --insecure-skip-tls-verify=true
          oc project <nom-du-projet>

      - name: Start OpenShift Build
        run: |
          oc start-build <nom-du-buildconfig> --follow
```

* **Remplace** `<ton-cluster>`, `<nom-du-projet>`, `<nom-du-buildconfig>` par tes valeurs.

---

### 4. Valider le workflow

* Pousse un commit ou crée une PR.
* Vérifie que le workflow GitHub Actions s’exécute et lance bien le build OpenShift.

---

## Points de contrôle

* Le pipeline GitHub Actions s’exécute bien à chaque push/PR.
* Le build OpenShift se déclenche automatiquement et aboutit (logs visibles dans GitHub Actions).
* Le déploiement sur OpenShift est mis à jour sans intervention manuelle.

---

## Pour aller plus loin

* Ajoute une étape pour vérifier la santé de l’application après le déploiement (`oc rollout status`).
* Ajoute une étape pour publier un tag de version.
* Adapte le pipeline pour d’autres CI/CD (GitLab CI, Azure DevOps, Jenkins…).

---

**À retenir :**
L’intégration du build OpenShift dans une pipeline CI/CD te permet d’automatiser complètement tes déploiements, d’accélérer les releases, et d’améliorer la traçabilité de tes livraisons.

```

---