# Prerequis

## Creation des utilisateurs
1. Connexion au serveur avec l'utilisateur `root` : 
    ```bash 
    ssh root@vps709628.ovh.net
    ```
2. Création du user `westendorp` :
    ```bash 
    adduser westendorp
    ```
3. Ajout de l'utilisateur `westendorp` au group `sudo` :
    ```bash 
    usermod -aG sudo westendorp 
    ```
4. Vérification de la connexion ssh avec le user `westendorp`
    ```bash 
    ssh westendorp@vps709628.ovh.net 
    ```
## Installation de docker


1. Mise de la liste des paquets disponnible
    ```bash 
    sudo apt-get update
    ```
2. Installation des paquets necessaire pour l'installation de docker
    ```bash
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    ```
3. Ajout de la clef GPG de Docker
    ```bash 
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
4. Ajout du repository officiel de Docker
    ```bash 
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```
5. Mise de la liste des paquets disponnible
    ```bash 
    sudo apt-get update
    ```
6. Installation de Docker CE et containerd
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```
7. Test de l'installation de Docker
   ```bash
   sudo docker run hello-world
   ```

## Installation de docker-compose

1. Téléchargement du binaire de docker-compose
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```
2. Configuration des droits du binaire
   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

## Configuration de Traefik et Docker

1. Création du réseau docker : `web`
   ```bash
   sudo docker network create web
   ```
2. Création des repertoires et fichier necessaire a l'utilisation de **Traefik** et **Let's Encrypt**
    ```bash
    sudo mkdir -p /opt/traefik/ && sudo touch /opt/traefik/docker-compose.yml && \
    sudo touch /opt/traefik/acme.json && sudo chmod 600 /opt/traefik/acme.json && \
    sudo touch /opt/traefik/traefik.toml && \
    sudo mkdir -p /opt/westendorp/
    ```
3. Ajout de la configuration du **container** de **Traefik** dans le fichier `/opt/traefik/docker-compose.yml`
    ```bash
    sudo bash -c  "echo \"version: '3.6'

    services:
        traefik:
            image: traefik:1.7.11
            restart: always
            ports:
            - 80:80
            - 443:443
            networks:
            - web
            volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /opt/traefik/traefik.toml:/traefik.toml
            - /opt/traefik/acme.json:/acme.json
            container_name: traefik

    networks:
        web:
            external: true\" > /opt/traefik/docker-compose.yml"
    ```
4. Ajout de la configuration de **Traefik** et **Let's Encrypt** dans le ficher `/opt/traefik/traefik.toml`
    ```bash
    sudo bash -c  'echo "debug = false

    logLevel = \"DEBUG\"
    defaultEntryPoints = [\"https\",\"http\"]

    [entryPoints]
        [entryPoints.http]
        address = \":80\"
            [entryPoints.http.redirect]
            entryPoint = \"https\"
        [entryPoints.https]
        address = \":443\"
        [entryPoints.https.tls]

    [retry]

    [docker]
    endpoint = \"unix:///var/run/docker.sock\"
    domain = \"ce-westendorp.com\"
    watch = true
    exposedByDefault = false

    [acme]
    email = \"michaele.verger@ce-westendorp.com\"
    storage = \"acme.json\"
    entryPoint = \"https\"
    onHostRule = true
    [acme.httpChallenge]
    entryPoint = \"http\"" > /opt/traefik/traefik.toml'
    ```
5. Lancement de **Traefik**
    ```bash
    cd /opt/traefik && sudo docker-compose up -d
    ```
6. Création du `docker-compose.yml` de Westendorp :
    ```bash 
    sudo bash -c  "echo \"version: '3.3'

    services:
    db:
        image: mongo:4.0.1
        restart: always
        networks:
        - web
        volumes:
        - /opt/mongo-volume:/data/db

    api:
        build:
        context: ./tours-0219-js-westendorp-api
        restart: always 
        environment:
        - UPLOAD_PATH=/uploads
        - NODE_ENV=production
        - PORT=3000
        - SECRET_KEY=${SECRET}
        - DBURI=mongodb://db/Westendorp?retryWrites=true
        networks:
        - web
        volumes:
        - /opt/uploads:/uploads
        depends_on:
        - db
        labels:
        - 'traefik.enable=true'
        - 'traefik.frontend.rule=Host:www.ce-westendorp.com,ce-westendorp.com;PathPrefix:/api,/images'
        - 'traefik.port=3000'
        - 'traefik.docker.network=web'

    ui:
        build:
        context: ./tours-0219-js-westendorp-ui
        restart: always
        environment:
        - NODE_ENV=production
        - PORT=80
        networks:
        - web
        depends_on:
        - api
        labels:
        - 'traefik.enable=true'
        - 'traefik.frontend.rule=Host:www.ce-westendorp.com,ce-westendorp.com'
        - 'traefik.port=80'
        - 'traefik.docker.network=web'

    networks:
    web:
        external: true
    volumes:
    mongo-volume:
        driver: 'local'
    upload-volume:
        driver: 'local'\" > /opt/westendorp/docker-compose.yml"
    ```

# Deploiement de l'UI et API

1. Se connecter en SSH sur le serveur
    ```bash 
    ssh westendorp@vps709628.ovh.net 
    ```
4. Récuperer et installer les dernières versions de l'UI et de l'API (remplacer v1.0.1 par la version souhaité)
    ```bash
    cd /opt/westendorp && \
    sudo rm -rf tours-0219-js-westendorp-ui tours-0219-js-westendorp-api && \
    sudo mkdir tours-0219-js-westendorp-ui tours-0219-js-westendorp-api && \
    sudo wget -O- https://github.com/WildCodeSchool/tours-0219-js-westendorp-ui/archive/v1.0.1.tar.gz | sudo tar -xvz --strip 1 -C ./tours-0219-js-westendorp-ui && \
    sudo wget -O- https://github.com/WildCodeSchool/tours-0219-js-westendorp-api/archive/v1.0.1.tar.gz | sudo tar -xvz --strip 1 -C ./tours-0219-js-westendorp-api && \
    sudo docker-compose up --build -d
    ```