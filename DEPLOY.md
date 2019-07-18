# Prerequis

## Creation des utilisateurs
1. Connexion au serveur avec l'utilisateur `root` : 
    ```bash 
    ssh root@vps709628.ovh.net
    ```
2. Creation du user `westendorp` :
    ```bash 
    adduser westendorp
    ```
3. Ajout de l'utilisateur `westendorp` au group `sudo` :
    ```bash 
    usermod -aG sudo westendorp 
    ```
4. Verification de la connexion ssh avec le user `westendorp`
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
5. Mise de la liste des paquets disponnible (car nous avons ajoute le repository officiel de Docker)
    ```bash 
    sudo apt-get update
    ```
4. Installation de Docker CE et containerd
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```
5. Test de l'installation de Docker
   ```bash
   sudo docker run hello-world
   ```
   Un long message d'information devrait aparaitre, sur la 7 lignes, vous devriez voir `Hello from Docker!`.

## Installation de docker-compose

1. Telechargement du binaire de docker-compose
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```
2. Configuration des droits du binaire
   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

## Configuration de Traefik et Docker

1. Creation du resau docker : `web`
   ```bash
   sudo docker network create web
   ```
2. Creation des repertoires et fichier necessaire a l'utilisation de **Traefik** et **Let's Encrypt**
    ```bash
    sudo mkdir -p /opt/traefik/ && sudo touch /opt/traefik/docker-compose.yml
    sudo touch /opt/traefik/acme.json && sudo chmod 600 /opt/traefik/acme.json
    sudo touch /opt/traefik/traefik.toml
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

    logLevel = \"ERROR\"
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
    domain = \"ce-westendop.com\"
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
    sudo docker-compose up -d
    ```
## Recuperation du projet

1. Recuperation du projet depuis Github
    ```bash
    sudo git clone --single-branch --branch master --depth 1 https://github.com/WildCodeSchool/tours-0219-js-westendorp.git /opt/westendorp
    ```
2. Mise a jour des sous repertoires :
   ```bash
   cd /opt/westendorp && sudo git submodule update --init --recursive --depth 1
   ```
# Deploiement de l'UI et API
1. Se connecter en SSH sur le serveur
2. Recuperer les dernieres mise a jour des branchs `master`
   ```bash
   cd /opt/westendorp && sudo git submodule update --recursive --depth 1
   ```
3. Valorisation de la variable d'environement "SECRET", aller sur [randomkeygen](https://randomkeygen.com/), copier la valeur a la place de "monsecret" dans la commande suivante :
   ```bash
   export SECRET=monsecret
   ```
4. Executer la commande suivante pour demarrer tous les containers :
   ```bash
   cd /opt/westendorp && sudo docker-compose --build -d
   ```
5. **OPTIONNEL** creation de l'utilisateur admin (si la base de donnee est vierge par exemple) :

