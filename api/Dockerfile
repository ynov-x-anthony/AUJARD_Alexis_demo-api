# demo-api — image de production DURCIE pour le serveur Node.
# Base epinglee sur une version mineure precise (ni :22-alpine, ni :latest).
FROM node:22.11-alpine

# Repertoire de travail a l'interieur du conteneur.
WORKDIR /app

# --- Couche 1 : dependances ---------------------------------------------
# On copie d'ABORD les manifestes (cache de "npm ci" tant qu'ils ne changent pas).
# --chown=node:node : les fichiers appartiennent a l'utilisateur non-root "node".
COPY --chown=node:node package.json package-lock.json ./
RUN npm ci --omit=dev

# --- Couche 2 : code applicatif -----------------------------------------
COPY --chown=node:node server.js db.js ./

# Port d'ecoute (>= 1024, donc pas besoin de privileges root).
EXPOSE 3000

# Verifie la liveness du serveur via /health (wget est fourni par busybox/alpine).
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -q -O - http://127.0.0.1:3000/health || exit 1

# On bascule en utilisateur non-root AVANT le CMD (uid 1000 = "node").
USER node

# Exec form : "node" devient PID 1 et recoit proprement SIGTERM a l'arret.
CMD ["node", "server.js"]
