# T02 — Gestion des eleves, du personnel et des parametres

Ce dossier contient uniquement ce qui correspond a la carte T02 du Trello.

## Avant de commencer

1. `git checkout develop && git pull` (recupere le socle T01 : auth, user, security, audit).
2. `git checkout -b feature/cheikh-eleves-personnel`

## Comment integrer ce dossier

Copie le contenu de ce zip (dossiers `src/main/java/com/daara/internatadmin/enfant`,
`.../personnel`, `.../parametres`, et les fichiers `src/main/resources/db/migration/V3*`,
`V5*`, `V6*`, `V14*`) directement dans ton clone de `backend-ci`, en respectant
l'arborescence (il suffit de fusionner les dossiers `src/`).

## Important : endpoint /stats retire temporairement

`EnfantController`/`EnfantService` et `EmployeController`/`EmployeService` avaient a
l'origine un endpoint `/stats` qui calcule aussi les impayes. Cette fonctionnalite a
besoin du module `finance` (paiements), qui arrive avec la tache T03 (Abdoulaye), pas
encore mergee au moment ou tu commences T02. L'endpoint a donc ete retire pour que ton
code compile seul. Abdoulaye le remettra en place quand il ajoutera le module finance
dans T03 (merge sur develop apres ton propre merge).

Tout le reste (creation, liste, recherche, mise a jour de statut, impression carte/
certificat, parametres de l'etablissement) est complet et autonome.

## Tester

```bash
./mvnw verify
./mvnw spring-boot:run
```

Puis dans Swagger (`http://localhost:8080/swagger-ui.html`) :
1. Se connecter avec le compte admin (herite du socle T01).
2. Creer un eleve via `POST /api/enfants`.
3. Creer un employe via `POST /api/employes`.
4. Verifier `GET /api/enfants`, `GET /api/employes`, `GET /api/parametres`.

## A savoir expliquer a la soutenance

Pour chaque module (`enfant`, `personnel`) : l'entite JPA, le repository, le service
(avec la logique de generation de matricule et de detection des doublons), le
controller REST, et la migration Flyway correspondante.
