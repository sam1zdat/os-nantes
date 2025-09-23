# Exercice 2 — Automatisation du Versioning dans OpenShift

## Objectif

Appliquer automatiquement un tag de version à chaque build d’une application Node.js déployée avec S2I.

---

## Étapes détaillées

### 1. Modifier le BuildConfig pour ajouter un post-commit hook

```bash
oc edit bc/app-s2i
```

Ajoute (ou adapte) la section suivante sous `spec:` :

```yaml
postCommit:
  script: "echo Hello from post-commit hook"
```

---

### 2. Relancer un build

```bash
oc start-build app-s2i --follow
```

**Explication :** Déclenche un nouveau build et exécute le post-commit hook pour afficher le message "Hello from post-commit hook".

---

### 3. Vérifier les tags créés dans l’ImageStream

```bash
oc get is app-s2i -o yaml
```

**Explication :**

* Permet de visualiser tous les tags présents dans l’ImageStream, notamment le tag correspondant à la version extraite.

---

## Points de contrôle

* L’ImageStream comporte au moins deux tags : `latest` et la version extraite (`1.0.0` par exemple).
* Chaque nouveau build met à jour ou ajoute le tag correspondant à la nouvelle version.