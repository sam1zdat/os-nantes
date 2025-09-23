# Exercice 3 — Build et Déploiement d'une Application Node.js Personnalisée (Stratégie Docker)

## Objectif

Déployer une application Node.js simple (créée localement) sur OpenShift à l’aide d’un Dockerfile personnalisé, via la stratégie Docker.

---

## Étapes détaillées

### 1. Créer un dossier de projet Node.js

```bash
mkdir tp-node-docker
cd tp-node-docker
npm init -y
````

---

### 2. Créer le code Node.js

**server.js**

```js
const http = require('http');
const PORT = 8080;
http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from Node.js Docker on OpenShift!\n');
}).listen(PORT, () => {
  console.log(`Server running on http://0.0.0.0:${PORT}`);
});
```

---

### 3. Ajouter un Dockerfile

**Dockerfile**

```Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY server.js .
EXPOSE 8080
CMD ["node", "server.js"]
```

---

### 4. Créer le BuildConfig et l’ImageStream

```bash
oc new-build --strategy=docker --name=nodejs-demo --binary
```

---

### 5. Lancer le build en envoyant le code local

Place-toi dans le dossier du projet (où se trouve le Dockerfile) puis :

```bash
oc start-build nodejs-demo --from-dir=. --follow
```

---

### 6. Déployer et exposer l’application

```bash
oc new-app nodejs-demo
oc expose svc/nodejs-demo
oc get route
```

---

### 7. Tester dans le navigateur

* Ouvre l’URL donnée par `oc get route` dans ton navigateur.
* Tu dois voir : **Hello from Node.js Docker on OpenShift!**

---

## Points de contrôle

* Le build utilise bien le Dockerfile local (stratégie Docker, pas S2I).
* L’application Node.js répond correctement via l’URL publique d’OpenShift.
* Les objets ImageStream, BuildConfig, Deployment, Service et Route sont créés.

---

## Bonus

* Modifie le message de `server.js`, rebuild, et vérifie la mise à jour.
* Expérimente avec différentes versions de Node.js (change la version dans le Dockerfile).

```

---