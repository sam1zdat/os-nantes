Voici ton TP réécrit avec les corrections + vérifications après chaque commande (et sans l’erreur oc new-app node-app) ✅

Exercice pratique 1 : Multi-stage build avec OpenShift
Objectif
Optimiser une image Node.js avec multi-stage build et la construire/déployer via OpenShift (CRI-O).

Étapes
1) Préparation
mkdir node-app && cd node-app
server.js

const http=require('http');
const server=http.createServer((req,res)=>{
  if(req.url==='/health'){res.writeHead(200);return res.end('OK');}
  res.writeHead(200,{'Content-Type':'text/plain'});res.end('Hello OpenShift!\n');
});
server.listen(process.env.PORT||8080,()=>console.log('Server running'));
package.json

{"name":"node-app","version":"1.0.0","main":"server.js",
 "scripts":{"start":"node server.js"},
 "dependencies":{"express":"^4.17.1"}}
2) Dockerfile multi-stage
# Exercice pratique 1 : Multi-stage build avec OpenShift
# Objectif : séparer build et runtime pour une image plus légère et sécurisée.

# ---------- STAGE 1 : BUILD ----------
# Image légère Node.js pour compiler/installer les dépendances
FROM node:18-alpine AS build

# Répertoire de travail dans l’image
WORKDIR /app

# 1) Copier uniquement les manifestes pour profiter du cache
COPY package*.json ./

# 2) Installer les deps
#    - Sans package-lock.json : npm install (puis on retire les devDeps)
#    - Avec package-lock.json : préférez `npm ci` (reproductible)
RUN npm install && npm prune --omit=dev
# RUN npm ci  # ← décommentez si vous avez un package-lock.json

# 3) Copier le reste du code (sources de l’application)
COPY . .

# 4) Ne garder que les dépendances runtime
RUN npm prune --omit=dev

# ---------- STAGE 2 : RUNTIME ----------
# Image d’exécution minimaliste
FROM node:18-alpine

# Variables d’environnement :
# - NODE_ENV=production optimise Node
# - PORT=8080 requis/recommandé sur OpenShift
ENV NODE_ENV=production \
    PORT=8080

# Répertoire de travail
WORKDIR /app

# Copier uniquement ce qui est nécessaire depuis le stage de build
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app .

# Permissions compatibles OpenShift (UID aléatoire, groupe 0)
RUN chgrp -R 0 /app && chmod -R g=u /app

# Exécuter en utilisateur non-root (bonnes pratiques sécurité)
USER 1001

# Exposer le port applicatif
EXPOSE 8080

# Commande de démarrage (doit correspondre au script "start")
CMD ["npm", "start"]

# Sonde de santé HTTP (endpoint /health)
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://127.0.0.1:8080/health || exit 1

3) Build dans OpenShift (binaire)
oc new-build --name=node-app --strategy=docker --binary
# Vérifs
oc get bc node-app -o wide
oc get is node-app -o wide
oc describe bc/node-app 
# Déclencher un nouveau build à partir du répertoire courant et suivre les logs
oc start-build node-app --from-dir=. --follow

# Vérifier les builds
oc get builds -o wide 
# Décrire le build
oc describe build node-app-1
# Vérifier l'ImageStreamTag
oc get istag node-app:latest -o jsonpath='{.image.dockerImageReference}{"\n"}'
4) Déploiement depuis l’ImageStreamTag
# Créer une nouvelle application à partir de l'ImageStreamTag
oc new-app -i node-app:latest --name=node-app

# Vérifier le déploiement et le service
oc get deploy,svc -l app=node-app
# Vérifier l'image utilisée par le déploiement
oc get deploy node-app -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
# Exposer le service node-app pour y accéder depuis l'extérieur
oc expose service/node-app
# Vérifier la route créée
oc get route node-app -o jsonpath='{.spec.host}{"\n"}'
# Vérifier le statut du déploiement
oc rollout status deploy/node-app
# Vérifier les pods
oc get pods 
# Vérifier les logs du déploiement
oc logs -f deploy/node-app
5) Test d’accès
# Tester l'accès à l'application via la route créée
curl -i http://$(oc get route node-app -o jsonpath='{.spec.host}')/health
Structure attendue
node-app/
├── server.js
├── package.json
└── Dockerfile
Résultat attendu
Image buildée via OpenShift (multi-stage).
Déploiement node-app en non-root, écoute 8080.
Route exposée, /health → 200 OK.
Validation & Debug
Build
oc logs -f bc/node-app
oc get builds; oc describe build $(oc get builds -o name | tail -n1)
oc get is/node-app -o yaml | grep -E 'tag:|dockerImageReference'
Déploiement
oc rollout status deploy/node-app
oc get pods -l app=node-app
oc logs -f deploy/node-app
oc get events --sort-by=.lastTimestamp | tail -n20
Astuces
Si des ressources node (et non node-app) ont été créées par erreur :

oc delete all -l app=node --ignore-not-found
CrashLoopBackOff : vérifier que l’app écoute PORT=8080 et que CMD ["npm","start"] existe.

ImagePullBackOff : vérifier oc get istag node-app:latest et l’image utilisée par le déploiement.
