#   

## Projets
1. [smart-borne-api](https://github.com/WildCodeSchool/tours-0219-js-smart-borne-api)
2. [smart-borne-ui](https://github.com/WildCodeSchool/tours-0219-js-smart-borne-ui)

## Git sub modules  
### Cloner le projet

Pour cloner le projet et ses sous-modules, entrez la commande suivante :

```sh
# With SSH
$ git clone --recurse-submodules git@github.com:WildCodeSchool/tours-0219-js-smart-borne.git

# With HTTPS
$ git clone --recurse-submodules https://github.com/WildCodeSchool/tours-0219-js-smart-borne.git
```

### Mise à jour des sous-modules

```sh
# Tous les sous-modules
$ git submodule update --remote

# Sous-module back
$ git submodule update --remote smart-borne-api

# Sous-module front
$ git submodule update --remote smart-borne-ui
```

### Pousser les modifications des sous-modules

```sh
# Si modifs sur back
$ git add smart-borne-api

# Si modifs sur front
$ git add smart-borne-ui

# Commit message
$ git commit -m "message explicite"

# Push
$ git push
```