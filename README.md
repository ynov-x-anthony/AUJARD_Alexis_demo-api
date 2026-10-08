# AUJARD Alexis — Démo API (Fil rouge Docker)

Dépôt du fil rouge des quêtes Docker.

## Quête 1 — Une base PostgreSQL dans un conteneur

**Objectif :** lancer une base PostgreSQL officielle dans un conteneur Docker,
y créer une table, insérer une ligne, puis nettoyer le conteneur.

### Étapes réalisées

1. **Lancer le conteneur** (image officielle `postgres:16-alpine`, en tâche de fond) :

   ```bash
   docker run -d --name demo-db \
     -e POSTGRES_USER=demo \
     -e POSTGRES_PASSWORD=demo \
     -e POSTGRES_DB=demo \
     postgres:16-alpine
   ```

2. **Vérifier le démarrage :**

   ```bash
   docker ps
   docker logs demo-db   # repérer : "database system is ready to accept connections"
   ```

3. **Ouvrir un client SQL dans le conteneur :**

   ```bash
   docker exec -it demo-db psql -U demo -d demo
   ```

4. **Créer une table et insérer une ligne (dans psql) :**

   ```sql
   CREATE TABLE products (id serial primary key, name text, price_cents int);
   INSERT INTO products (name, price_cents) VALUES ('Sticker Démo', 150);
   SELECT * FROM products;
   \dt
   \q
   ```

5. **Arrêter et supprimer le conteneur :**

   ```bash
   docker stop demo-db
   docker rm demo-db
   ```

### Résultats

**`SELECT * FROM products;`**

```
 id |     name     | price_cents
----+--------------+-------------
  1 | Sticker Démo |         150
(1 row)
```

**`\dt`**

```
         List of relations
 Schema |   Name   | Type  | Owner
--------+----------+-------+-------
 public | products | table | demo
(1 row)
```

**3 dernières lignes de `docker logs demo-db`**

```
2026-10-08 13:14:16.256 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2026-10-08 13:14:16.260 UTC [57] LOG:  database system was shut down at 2026-10-08 13:14:16 UTC
2026-10-08 13:14:16.263 UTC [1] LOG:  database system is ready to accept connections
```

### Critères d'acceptation

- [x] La table `products` apparaît dans `\dt` et contient au moins une ligne.
- [x] Les journaux montrent que PostgreSQL a démarré (`ready to accept connections`).

### Notes / ce que j'ai appris

- `-d` lance le conteneur en **détaché** (tâche de fond).
- `-e` passe les **variables d'environnement** qui configurent PostgreSQL.
- On entre dans le conteneur avec `docker exec`, pas besoin de publier un port (`-p`).
- `psql` vit **dans** l'image `postgres`, donc on l'exécute via `docker exec`.
- Sous **PowerShell**, j'ai séparé `docker stop` et `docker rm` (le `&&` du shell Unix
  n'est pas garanti sous Windows PowerShell).

---

## Quête 2 — Fil rouge étape 1 : le Dockerfile de `demo-api`

**Objectif :** produire une image de `demo-api` qui démarre le serveur Node.

Le code (`server.js`, `db.js`, `package.json`, `package-lock.json`) vient du repo
de départ fourni par le formateur. J'ai écrit le `Dockerfile` et le `.dockerignore`
dans [`api/`](./api).

### Le Dockerfile (points clés)

- Base **épinglée et légère** : `node:22-alpine`.
- **Dépendances en couche séparée du code** : `COPY package.json package-lock.json`
  → `RUN npm ci --omit=dev` → **puis** `COPY server.js db.js`.
  Résultat : modifier le code ne recasse pas l'installation des dépendances.
- `EXPOSE 3000`.
- Démarrage en **exec form** : `CMD ["node", "server.js"]` (node = PID 1, reçoit SIGTERM).

### Build, run et test

```bash
docker build -t demo-api:1.0 ./api

docker run -d --name api -p 8080:3000 demo-api:1.0
curl -s localhost:8080/health   # {"status":"UP"}
curl -s localhost:8080/         # {"ok":true,"app":"demo-api","version":"dev"}
docker rm -f api
```

Sorties obtenues :

```
GET /health  -> {"status":"UP"}
GET /        -> {"ok":true,"app":"demo-api","version":"dev"}
```

### Preuve du cache

Après modification d'un simple commentaire dans `server.js`, rebuild avec
`docker build --progress=plain -t demo-api:1.0 ./api` :

```
#6 [3/5] COPY package.json package-lock.json ./
#6 CACHED
#8 [4/5] RUN npm ci --omit=dev
#8 CACHED
#9 [5/5] COPY server.js db.js ./
#9 DONE 0.0s
```

`npm ci` reste **CACHED** ; seule la couche `COPY server.js db.js` est reconstruite.

### `docker image ls demo-api`

```
IMAGE          ID             DISK USAGE   CONTENT SIZE
demo-api:1.0   d6765a364b6d   173MB        0B
```

### Critères d'acceptation

- [x] L'image se construit et `docker run` répond sur `/health`.
- [x] `npm ci` est mis en cache quand seul `server.js` change (preuve ci-dessus).
- [x] Le `Dockerfile` respecte : base épinglée, ordre deps→code, `.dockerignore` présent.
- [x] L'image est publiée sur un registre (GHCR, ci-dessous) et le repo est sur GitHub.

### Image publiée

- Registre : **GHCR** — `ghcr.io/phenix-13/demo-api:1.0`
- Récupération : `docker pull ghcr.io/phenix-13/demo-api:1.0`
- Digest : `sha256:263a87c299b3247e228631ec761777930d8ebe3e411a649e678d1e822fab7d84`

