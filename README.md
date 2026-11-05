# Keycloak Academy — lab

Un Keycloak 26 en mode développement et un OpenLDAP de test, pour les exercices de Keycloak Academy, la formation Keycloak de [Liksi](https://www.liksi.fr).

Prérequis : Docker avec Compose. Certains exercices utilisent aussi `jq` et `python3`.

## Démarrer

```bash
git clone https://github.com/liksi/keycloak-academy-lab.git
cd keycloak-academy-lab
docker compose up -d
```

Keycloak met une trentaine de secondes à démarrer. Suivre les logs avec `docker compose logs -f keycloak`.

## Accès

| Service | Depuis votre poste | Depuis le conteneur Keycloak | Identifiants |
|---|---|---|---|
| Console d'administration | http://localhost:8080/admin | — | `admin` / `admin` |
| OpenLDAP | `ldap://localhost:389` | `ldap://openldap:389` | `cn=admin,dc=academy,dc=local` / `admin` |

Base LDAP : `dc=academy,dc=local`.

## Comptes LDAP

Tous les utilisateurs (`inetOrgPerson`) sont sous `ou=people,dc=academy,dc=local`, avec le mot de passe `password`.

| uid | Nom | Groupe |
|---|---|---|
| alice | Alice Martin | agents |
| bruno | Bruno Leroy | agents |
| chloe | Chloe Bernard | agents |
| david | David Petit | administrateurs |
| emma | Emma Durand | administrateurs |

Les groupes (`groupOfNames`, attribut `member`) sont sous `ou=groups,dc=academy,dc=local`. Le seed est dans `ldap/seed.ldif`.

Vérifier l'annuaire :

```bash
docker compose exec openldap ldapsearch -x -H ldap://localhost \
  -D cn=admin,dc=academy,dc=local -w admin -b dc=academy,dc=local
```

## kcadm.sh

```bash
docker compose exec keycloak /opt/keycloak/bin/kcadm.sh config credentials \
  --server http://localhost:8080 --realm master --user admin --password admin
docker compose exec keycloak /opt/keycloak/bin/kcadm.sh get realms --fields realm
```

## Arrêter et réinitialiser

```bash
docker compose stop       # arrêter en gardant l'état
docker compose start      # reprendre
docker compose down -v    # tout supprimer : Keycloak et l'annuaire repartent de zéro
```

Le mode `start-dev` stocke les données dans le conteneur : `docker compose down -v` efface realms, clients et utilisateurs. L'annuaire est rechargé depuis `ldap/seed.ldif` au démarrage suivant (le seed n'est lu que sur un annuaire vide).
