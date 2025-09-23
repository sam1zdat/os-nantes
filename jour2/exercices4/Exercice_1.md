# Exercice 1 — Construction S2I Basique sur OpenShift

## Objectif

Déployer une application Node.js via Source-to-Image (S2I) sur OpenShift.

---

## Étapes détaillées

### 1. Créer un projet (action non autorisée dans Sandbox)

```bash
oc new-project tp-s2i-simple
```

**Explication :** Crée un nouvel espace de travail (namespace) pour isoler tes ressources.

---

### 2. Créer un BuildConfig S2I à partir d’un dépôt Git

```bash
oc new-app nodejs~https://github.com/sclorg/nodejs-ex.git --name=app-s2i
```

**Explication :**

* Crée un BuildConfig S2I, une ImageStream, un Deployment et un Service pour l’application.
* Utilise le code source du dépôt `nodejs-ex`.

---

### 3. Vérifier les objets créés

```bash
oc get all
oc get pods
oc get bc
oc get is
oc get deployment
oc get svc
oc get route
```

**Explication :**

* `oc get all` : Affiche tous les objets du namespace.
* `oc get bc` : Vérifie le BuildConfig (stratégie de build).
* `oc get is` : Vérifie l’ImageStream (stocke l’image buildée).
* `oc get deployment` : Vérifie le déploiement de l’app.
* `oc get svc` : Vérifie le service exposé.
* `oc get route` : Vérifie la route HTTP externe.

---

### 4. Démarrer un build

```bash
oc start-build app-s2i --follow
```

**Explication :** Démarre la construction de l’image et affiche les logs en direct.

---

### 5. Vérifier le build et l’image créée

```bash
oc get builds
oc get is
```

* `oc get builds` : Affiche la liste et l’état des builds réalisés.
* `oc get is` : Vérifie que l’image a bien été créée et taguée.

---

### 6. Exposer l’application et accéder à l’URL

```bash
oc expose svc/app-s2i
oc get route
```

* `oc expose` : Crée une route externe pour accéder à l’application.
* `oc get route` : Affiche l’URL à ouvrir dans le navigateur.

---