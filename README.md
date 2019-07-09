#   

## Projets
1. [westendorp-api](https://github.com/WildCodeSchool/tours-0219-js-westendorp-api)
2. [westendorp-ui](https://github.com/WildCodeSchool/tours-0219-js-westendorp-ui)

## Git sub modules  
### Cloner le projet

Pour cloner le projet et ses sous-modules, entrez la commande suivante :

```sh
# With SSH
$ git clone --recurse-submodules git@github.com:WildCodeSchool/tours-0219-js-westendorp.git

# With HTTPS
$ git clone --recurse-submodules https://github.com/WildCodeSchool/tours-0219-js-westendorp.git
```

### Mise à jour des sous-modules

```sh
# Tous les sous-modules
$ git submodule update --remote

# Sous-module back
$ git submodule update --remote westendorp-api

# Sous-module front
$ git submodule update --remote westendorp-ui
```

### Pousser les modifications des sous-modules

```sh
# Si modifs sur back
$ git add westendorp-api

# Si modifs sur front
$ git add westendorp-ui

# Commit message
$ git commit -m "message explicite"

# Push
$ git push
```