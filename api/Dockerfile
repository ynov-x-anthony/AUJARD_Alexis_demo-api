# demo-api — image de production pour le serveur Node.
# Base epinglee et legere, demandee par la quete.
FROM node:22-alpine

# Repertoire de travail a l'interieur du conteneur.
WORKDIR /app

# --- Couche 1 : dependances ---------------------------------------------
# On copie d'ABORD seulement les manifestes. Tant que package.json /
# package-lock.json ne changent pas, Docker reutilise le cache de "npm ci"
# meme si le code applicatif change ensuite -> builds rapides.
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# --- Couche 2 : code applicatif -----------------------------------------
# Copie APRES l'install : une modif de server.js n'invalide pas la couche
# des dependances au-dessus.
COPY server.js db.js ./

# Le serveur ecoute sur 3000 (documentation + outillage).
EXPOSE 3000

# Exec form : "node" devient PID 1 et recoit proprement SIGTERM a l'arret.
CMD ["node", "server.js"]
