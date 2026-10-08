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
