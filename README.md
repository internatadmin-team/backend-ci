# backend-ci — InternatAdmin

Backend monolithique (Spring Boot) de l'application InternatAdmin : gestion
d'un Daara (internat) — eleves, personnel, paiements et comptabilite.

## Presentation

- Java 21, Spring Boot, Maven (wrapper `mvnw`).
- Persistance PostgreSQL, schema gere par Flyway (`classpath:db/migration`) ;
  Hibernate reste en `validate` et ne modifie jamais le schema lui-meme.
- Authentification par JWT (HS256).
- Documentation API auto-generee : Swagger UI (`/swagger-ui.html`) et OpenAPI
  (`/v3/api-docs`).
- Supervision : endpoint de sante `/health` (Spring Boot Actuator, sondes
  liveness/readiness activees).

## Prerequis

- JDK 21
- Docker Desktop (pour les tests d'integration via Testcontainers et pour le
  build de l'image)
- Une base PostgreSQL accessible en local (ou via `docker compose`, voir le
  depot `deployment-cd`)

## Variables d'environnement

Aucune valeur secrete n'est commitee dans ce depot. En local, cree un fichier
`.env` (non versionne, voir `.gitignore`) ou exporte ces variables dans ton
shell avant de lancer l'application :

| Variable | Obligatoire | Valeur par defaut | Role |
|---|---|---|---|
| `DB_URL` | oui | — | URL JDBC PostgreSQL (ex. `jdbc:postgresql://localhost:5432/internatadmin`) |
| `DB_USERNAME` | oui | — | Utilisateur PostgreSQL |
| `DB_PASSWORD` | oui | — | Mot de passe PostgreSQL |
| `JWT_SECRET` | oui | — | Cle HS256 encodee en base64, 256 bits minimum. Le demarrage echoue si elle est absente ou trop courte |
| `JWT_EXPIRATION_MINUTES` | non | `480` | Duree de validite des jetons JWT |
| `CORS_ALLOWED_ORIGINS` | non | `http://localhost:4200` | Origine(s) autorisee(s) a appeler l'API (frontend Angular) |
| `SERVER_PORT` | non | `8080` | Port d'ecoute du serveur |
| `LOGO_DIR` | non | `uploads/logos` | Dossier de stockage des logos uploades |
| `ADMIN_INITIAL_PASSWORD_HASH` | non | — | Hash bcrypt utilise uniquement au tout premier demarrage, si aucun compte ADMIN n'existe encore |
| `PAIEMENT_LOCK_TIMEOUT` | non | `5s` | Delai de verrouillage applique lors du traitement d'un paiement |

Aucune valeur reelle de `JWT_SECRET`, `DB_PASSWORD` ou `ADMIN_INITIAL_PASSWORD_HASH`
n'est partagee ici ; demande-les a un membre de l'equipe ou genere les tiennes
en local.

## Lancement local

```bash
export DB_URL=jdbc:postgresql://localhost:5432/internatadmin
export DB_USERNAME=internatadmin
export DB_PASSWORD=change-moi
export JWT_SECRET=$(openssl rand -base64 32)

./mvnw spring-boot:run
```

L'API est alors disponible sur `http://localhost:8080`, la doc interactive sur
`http://localhost:8080/swagger-ui.html`.

## Tests

```bash
./mvnw clean verify -DskipITs          # tests unitaires seulement
./mvnw verify -Dtest=NONE -Dsurefire.failIfNoSpecifiedTests=false   # tests d'integration (Testcontainers, necessite Docker)
```

## Pipeline Jenkins (`Jenkinsfile`)

Le pipeline CI s'execute automatiquement a chaque push sur ce depot (webhook
GitHub ou scrutation toutes les minutes en secours) et enchaine :

1. Build et tests unitaires, tests d'integration (Testcontainers).
2. Analyse des dependances OWASP Dependency-Check (bloque si une CVE atteint
   un score CVSS >= 9).
3. Build de l'image Docker (multi-etapes, utilisateur non root).
4. Scan de l'image avec Trivy (bloque sur toute vulnerabilite CRITICAL).
5. Publication de l'image sur Docker Hub (uniquement sur la branche `main`).
6. Declenchement automatique du job `deployment-cd` pour mettre a jour
   l'application deployee.
7. Notification du resultat du build dans Slack.

Un build incremental est applique : si rien dans `src/`, `pom.xml`,
`Dockerfile`, `.dockerignore`, `.mvn/` ou `mvnw` n'a change depuis le dernier
build reussi, les etapes de build/test/scan/publication sont sautees (le
parametre `BUILD_ALL` permet de forcer un build complet).

Voir `docs/securite.md` dans le depot `deployment-cd` pour le detail des
resultats de scan et de la politique d'exceptions.

## Image Docker Hub

Image publiee : [`abdoulaye940/daara-backend`](https://hub.docker.com/r/abdoulaye940/daara-backend)
— tags `latest` (dernier build reussi sur `main`) et `1.<numero de build>`
(version figee de chaque publication).
