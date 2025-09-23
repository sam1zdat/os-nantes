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
