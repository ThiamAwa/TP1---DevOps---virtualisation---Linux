# TP : Déploiement d'une Application Web Java avec Vagrant & Tomcat 9

> **Objectif :** Créer une machine virtuelle avec Vagrant, installer les JDKs (8, 11, 17), déployer Tomcat 9 et une application Web Java, puis automatiser le cycle de vie du serveur via un script de déploiement interactif.

---

##  Table des Matières

1. [Prérequis](#-prérequis)
2. [Partie 1 – Création de la VM avec Vagrant](#-partie-1--création-de-la-vm-avec-vagrant)
3. [Partie 2 – Connexion SSH à la VM](#-partie-2--connexion-ssh-à-la-vm)
4. [Partie 3 – Installation des JDKs](#-partie-3--installation-des-jdks)
5. [Partie 4 – Installation de Tomcat 9](#-partie-4--installation-de-tomcat-9)
6. [Partie 5 – Déploiement de l'application Web Java](#-partie-5--déploiement-de-lapplication-web-java)
7. [Partie 6 – Script deploy.sh](#-partie-6--script-deploysh)
8. [Résultat Final](#-résultat-final)

---

##  Prérequis

Avant de commencer, assurez-vous d'avoir installé sur votre machine hôte :

| Outil | Version minimale | Lien |
|-------|-----------------|------|
| [VirtualBox](https://www.virtualbox.org/) | 6.x+ | https://www.virtualbox.org/wiki/Downloads |
| [Vagrant](https://www.vagrantup.com/) | 2.x+ | https://developer.hashicorp.com/vagrant/downloads |
| Terminal / PowerShell | — | — |

Vérification des installations :

```bash
vagrant --version
# Vagrant 2.x.x

VBoxManage --version
# 6.x.x
```

---

## 🏗️ Partie 1 – Création de la VM avec Vagrant

### 1.1 – Initialisation du projet

Créez un dossier de travail et initialisez Vagrant :

```bash
mkdir tp-vagrant-tomcat
cd tp-vagrant-tomcat
vagrant init
```

### 1.2 – Contenu du Vagrantfile

Remplacez le contenu du fichier `Vagrantfile` généré par la configuration suivante :

```ruby
Vagrant.configure("2") do |config|

  config.vm.define "srv-web" do |web|
    # Box Ubuntu 20.04 LTS
    web.vm.box = "ubuntu/focal64"

    # Nom d'hôte de la VM
    web.vm.hostname = "srv-web"

    # Réseau privé avec IP fixe
    web.vm.network "private_network", ip: "192.168.56.10"

    # Redirection de port : accès Tomcat depuis l'hôte
    web.vm.network "forwarded_port", guest: 8080, host: 8080

    # Configuration VirtualBox
    web.vm.provider "virtualbox" do |vb|
      vb.name   = "srv-web"
      vb.memory = "2048"   # 2 Go RAM
      vb.cpus   = 2
    end

    # Message d'affichage au démarrage
    web.vm.post_up_message = "La VM srv-web est prête ! IP : 192.168.56.10"
  end

end
```

### 1.3 – Démarrage de la VM

```bash
vagrant up
```

> 📸 **Capture 1 :** `vagrant up` — démarrage de la VM

```
Bringing machine 'srv-web' up with 'virtualbox' provider...
==> srv-web: Importing base box 'ubuntu/focal64'...
==> srv-web: Matching MAC address for NAT networking...
==> srv-web: Setting the name of the VM: srv-web
==> srv-web: Booting VM...
==> srv-web: Waiting for machine to boot...
    srv-web: SSH address: 127.0.0.1:2222
    srv-web: SSH username: vagrant
    srv-web: SSH auth method: private key
==> srv-web: Machine booted and ready!
==> srv-web: La VM srv-web est prête ! IP : 192.168.56.10
```

![Capture 1 - vagrant up](./screenshots/01_vagrant_up.png)

### 1.4 – Vérification du statut

```bash
vagrant status
```

```
Current machine states:

srv-web                   running (virtualbox)
```

![Capture 2 - vagrant status](./screenshots/02_vagrant_status.png)

---

## 🔌 Partie 2 – Connexion SSH à la VM

### 2.1 – Connexion avec la commande Vagrant

```bash
vagrant ssh srv-web
```

Vous devriez voir le prompt Ubuntu de la VM :

```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-182-generic x86_64)

vagrant@srv-web:~$
```

> 📸 **Capture 3 :** Connexion SSH réussie

![Capture 3 - SSH](./screenshots/03_ssh_connexion.png)

### 2.2 – Mise à jour du système

Une fois connecté, mettez à jour les paquets :

```bash
sudo apt update && sudo apt upgrade -y
```

![Capture 4 - apt update](./screenshots/04_apt_update.png)

---

## ☕ Partie 3 – Installation des JDKs

### 3.1 – Installation de JDK 8

```bash
sudo apt install -y openjdk-8-jdk
```

Vérification :

```bash
java -version
# openjdk version "1.8.0_xxx"
```

> 📸 **Capture 5 :** Installation JDK 8

![Capture 5 - JDK 8](./screenshots/05_jdk8_install.png)

---

### 3.2 – Installation de JDK 11

```bash
sudo apt install -y openjdk-11-jdk
```

Vérification :

```bash
/usr/lib/jvm/java-11-openjdk-amd64/bin/java -version
# openjdk version "11.x.xx"
```

> 📸 **Capture 6 :** Installation JDK 11

![Capture 6 - JDK 11](./screenshots/06_jdk11_install.png)

---

### 3.3 – Installation de JDK 17

```bash
sudo apt install -y openjdk-17-jdk
```

Vérification :

```bash
/usr/lib/jvm/java-17-openjdk-amd64/bin/java -version
# openjdk version "17.x.x"
```

> 📸 **Capture 7 :** Installation JDK 17

![Capture 7 - JDK 17](./screenshots/07_jdk17_install.png)

---

### 3.4 – Gestion des versions avec `update-alternatives`

Enregistrez les trois JDKs dans le système :

```bash
sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 1

sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-11-openjdk-amd64/bin/java 2

sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-17-openjdk-amd64/bin/java 3
```

Pour switcher entre versions :

```bash
sudo update-alternatives --config java
```

```
There are 3 choices for the alternative java.

  Selection    Path                                          Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-17-openjdk-amd64/bin/java   3         auto mode
  1            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java  1       manual mode
  2            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   2         manual mode
  3            /usr/lib/jvm/java-17-openjdk-amd64/bin/java   3         manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

> 📸 **Capture 8 :** Sélection de la version Java active

![Capture 8 - update-alternatives](./screenshots/08_java_alternatives.png)

Définissez la variable `JAVA_HOME` pour Tomcat (JDK 11 recommandé) :

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

echo $JAVA_HOME
# /usr/lib/jvm/java-11-openjdk-amd64
```

---

## 🐱 Partie 4 – Installation de Tomcat 9

### 4.1 – Création d'un utilisateur dédié

```bash
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
```

### 4.2 – Téléchargement de Tomcat 9

```bash
cd /tmp
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz
```

> 📸 **Capture 9 :** Téléchargement de Tomcat 9

![Capture 9 - wget tomcat](./screenshots/09_tomcat_download.png)

### 4.3 – Extraction et installation

```bash
sudo tar -xzf apache-tomcat-9.0.89.tar.gz -C /opt/tomcat/
sudo ln -s /opt/tomcat/apache-tomcat-9.0.89 /opt/tomcat/latest
sudo chown -R tomcat: /opt/tomcat
sudo chmod +x /opt/tomcat/latest/bin/*.sh
```

### 4.4 – Création du service systemd

```bash
sudo nano /etc/systemd/system/tomcat.service
```

Contenu du fichier :

```ini
[Unit]
Description=Apache Tomcat 9 Web Application Server
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

### 4.5 – Activation et démarrage du service

```bash
sudo systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat
sudo systemctl status tomcat
```

> 📸 **Capture 10 :** Service Tomcat actif (running)

![Capture 10 - tomcat status](./screenshots/10_tomcat_status.png)

### 4.6 – Vérification dans le navigateur

Depuis votre machine hôte, ouvrez :

```
http://localhost:8080
```

> 📸 **Capture 11 :** Page d'accueil Tomcat dans le navigateur

![Capture 11 - tomcat browser](./screenshots/11_tomcat_browser.png)

### 4.7 – Configuration du Manager (optionnel)

```bash
sudo nano /opt/tomcat/latest/conf/tomcat-users.xml
```

Ajoutez avant `</tomcat-users>` :

```xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="Admin@1234" roles="manager-gui,admin-gui"/>
```

```bash
# Autoriser l'accès au Manager depuis toutes les IPs
sudo nano /opt/tomcat/latest/webapps/manager/META-INF/context.xml
# Commenter la ligne <Valve> ... RemoteAddrValve ...
sudo systemctl restart tomcat
```

---

## 🚀 Partie 5 – Déploiement de l'Application Web Java

### 5.1 – Préparer un fichier WAR

#### Option A – Utiliser un WAR de démonstration

```bash
# Téléchargement d'un WAR sample (sample.war de Apache)
wget https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/sample.war \
  -O /tmp/myapp.war
```

#### Option B – Créer un WAR simple depuis les sources

```bash
# Structure minimale d'une appli Java Web
mkdir -p ~/myapp/WEB-INF
cat > ~/myapp/index.jsp << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Mon App Java</title></head>
<body>
  <h1>✅ Application Java déployée avec succès !</h1>
  <p>Serveur : <%= application.getServerInfo() %></p>
  <p>Java    : <%= System.getProperty("java.version") %></p>
</body>
</html>
EOF

cat > ~/myapp/WEB-INF/web.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
  <display-name>MonApplication</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>
EOF

# Installation de l'outil jar (fourni avec JDK)
cd ~/myapp
jar -cvf /tmp/myapp.war .
```

### 5.2 – Déploiement du WAR dans Tomcat

```bash
sudo cp /tmp/myapp.war /opt/tomcat/latest/webapps/myapp.war
sudo chown tomcat:tomcat /opt/tomcat/latest/webapps/myapp.war
```

Tomcat détecte automatiquement le WAR et le déploie (hot deploy). Vérifiez :

```bash
ls /opt/tomcat/latest/webapps/
# myapp/   myapp.war   ROOT/   ...
```

> 📸 **Capture 12 :** Répertoire webapps après déploiement

![Capture 12 - webapps](./screenshots/12_webapps_deploy.png)

### 5.3 – Test de l'application

```bash
curl http://localhost:8080/myapp/
# Ou depuis le navigateur hôte :
# http://localhost:8080/myapp/
```

> 📸 **Capture 13 :** Application Java accessible dans le navigateur

![Capture 13 - app browser](./screenshots/13_app_browser.png)

---

## 📜 Partie 6 – Script deploy.sh

### 6.1 – Création du script

```bash
sudo nano /opt/tomcat/deploy.sh
```

Contenu complet du script :

```bash
#!/bin/bash
# ============================================================
#  deploy.sh — Gestion de Tomcat 9 (menu interactif)
#  Auteur  : Équipe TP DevOps
#  Date    : $(date +%Y-%m-%d)
# ============================================================

# --- Couleurs ---
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
BOLD='\033[1m'
RESET='\033[0m'

# --- Variables ---
TOMCAT_HOME="/opt/tomcat/latest"
WEBAPPS="$TOMCAT_HOME/webapps"
LOGS="$TOMCAT_HOME/logs/catalina.out"
SERVICE="tomcat"

# --- Fonctions utilitaires ---

print_header() {
  clear
  echo -e "${BLUE}${BOLD}"
  echo "  ╔══════════════════════════════════════════╗"
  echo "  ║      🐱  GESTION TOMCAT 9 — MENU         ║"
  echo "  ║         srv-web · 192.168.56.10           ║"
  echo "  ╚══════════════════════════════════════════╝"
  echo -e "${RESET}"
}

get_status() {
  if systemctl is-active --quiet $SERVICE; then
    echo -e "  État Tomcat : ${GREEN}●  EN COURS D'EXÉCUTION${RESET}"
  else
    echo -e "  État Tomcat : ${RED}●  ARRÊTÉ${RESET}"
  fi
  echo ""
}

# --- Actions ---

start_tomcat() {
  echo -e "${YELLOW}⏳  Démarrage de Tomcat...${RESET}"
  sudo systemctl start $SERVICE
  sleep 2
  if systemctl is-active --quiet $SERVICE; then
    echo -e "${GREEN}✅  Tomcat démarré avec succès.${RESET}"
  else
    echo -e "${RED}❌  Échec du démarrage. Consultez les logs.${RESET}"
  fi
}

stop_tomcat() {
  echo -e "${YELLOW}⏳  Arrêt de Tomcat...${RESET}"
  sudo systemctl stop $SERVICE
  sleep 2
  echo -e "${GREEN}✅  Tomcat arrêté.${RESET}"
}

restart_tomcat() {
  echo -e "${YELLOW}⏳  Redémarrage de Tomcat...${RESET}"
  sudo systemctl restart $SERVICE
  sleep 2
  if systemctl is-active --quiet $SERVICE; then
    echo -e "${GREEN}✅  Tomcat redémarré avec succès.${RESET}"
  else
    echo -e "${RED}❌  Échec du redémarrage.${RESET}"
  fi
}

status_tomcat() {
  echo -e "${CYAN}📊  Statut du service Tomcat :${RESET}"
  sudo systemctl status $SERVICE --no-pager
}

deploy_war() {
  echo -e "${CYAN}📦  Déploiement d'un fichier WAR${RESET}"
  read -rp "  Chemin du fichier WAR : " WAR_PATH
  if [ ! -f "$WAR_PATH" ]; then
    echo -e "${RED}❌  Fichier introuvable : $WAR_PATH${RESET}"
    return
  fi
  WAR_NAME=$(basename "$WAR_PATH")
  APP_NAME="${WAR_NAME%.war}"
  sudo cp "$WAR_PATH" "$WEBAPPS/"
  sudo chown tomcat:tomcat "$WEBAPPS/$WAR_NAME"
  echo -e "${GREEN}✅  WAR déployé : http://localhost:8080/$APP_NAME${RESET}"
}

undeploy_app() {
  echo -e "${CYAN}🗑️   Applications déployées :${RESET}"
  ls "$WEBAPPS" | grep -v 'ROOT\|manager\|host-manager\|examples\|docs'
  echo ""
  read -rp "  Nom de l'application à supprimer : " APP_NAME
  if [ -d "$WEBAPPS/$APP_NAME" ] || [ -f "$WEBAPPS/${APP_NAME}.war" ]; then
    sudo rm -rf "$WEBAPPS/$APP_NAME" "$WEBAPPS/${APP_NAME}.war"
    echo -e "${GREEN}✅  Application '$APP_NAME' supprimée.${RESET}"
  else
    echo -e "${RED}❌  Application '$APP_NAME' introuvable.${RESET}"
  fi
}

show_logs() {
  echo -e "${CYAN}📄  Dernières lignes du log Catalina :${RESET}"
  sudo tail -n 50 "$LOGS"
}

list_apps() {
  echo -e "${CYAN}📂  Applications déployées dans webapps :${RESET}"
  echo ""
  for dir in "$WEBAPPS"/*/; do
    app=$(basename "$dir")
    echo -e "  ${GREEN}▶${RESET}  $app  →  http://192.168.56.10:8080/$app"
  done
  echo ""
}

# ============================================================
# MENU PRINCIPAL
# ============================================================

while true; do
  print_header
  get_status

  echo -e "  ${BOLD}Choisissez une action :${RESET}"
  echo ""
  echo -e "  ${GREEN}1)${RESET}  Démarrer Tomcat"
  echo -e "  ${RED}2)${RESET}  Arrêter Tomcat"
  echo -e "  ${YELLOW}3)${RESET}  Redémarrer Tomcat"
  echo -e "  ${CYAN}4)${RESET}  Voir le statut"
  echo -e "  ${CYAN}5)${RESET}  Déployer un fichier WAR"
  echo -e "  ${CYAN}6)${RESET}  Supprimer une application"
  echo -e "  ${CYAN}7)${RESET}  Afficher les logs"
  echo -e "  ${CYAN}8)${RESET}  Lister les applications"
  echo -e "  ${RED}0)${RESET}  Quitter"
  echo ""
  read -rp "  Votre choix [0-8] : " CHOICE
  echo ""

  case $CHOICE in
    1) start_tomcat ;;
    2) stop_tomcat ;;
    3) restart_tomcat ;;
    4) status_tomcat ;;
    5) deploy_war ;;
    6) undeploy_app ;;
    7) show_logs ;;
    8) list_apps ;;
    0) echo -e "${GREEN}Au revoir !${RESET}"; exit 0 ;;
    *) echo -e "${RED}❌  Option invalide, réessayez.${RESET}" ;;
  esac

  echo ""
  read -rp "  Appuyez sur [Entrée] pour continuer..."
done
```

### 6.2 – Rendre le script exécutable

```bash
sudo chmod +x /opt/tomcat/deploy.sh
```

### 6.3 – Exécution du script

```bash
sudo /opt/tomcat/deploy.sh
```

> 📸 **Capture 14 :** Menu principal du script deploy.sh

![Capture 14 - deploy.sh menu](./screenshots/14_deploy_menu.png)

### 6.4 – Démonstration des options

#### Arrêt du serveur (option 2)

```
  Votre choix [0-8] : 2

⏳  Arrêt de Tomcat...
✅  Tomcat arrêté.
```

> 📸 **Capture 15 :** Arrêt de Tomcat via le script

![Capture 15 - stop tomcat](./screenshots/15_stop_tomcat.png)

#### Redémarrage du serveur (option 3)

```
  Votre choix [0-8] : 3

⏳  Redémarrage de Tomcat...
✅  Tomcat redémarré avec succès.
```

> 📸 **Capture 16 :** Redémarrage de Tomcat via le script

![Capture 16 - restart tomcat](./screenshots/16_restart_tomcat.png)

#### Déploiement d'un WAR (option 5)

```
  Votre choix [0-8] : 5

📦  Déploiement d'un fichier WAR
  Chemin du fichier WAR : /tmp/myapp.war
✅  WAR déployé : http://localhost:8080/myapp
```

> 📸 **Capture 17 :** Déploiement d'un WAR via le script

![Capture 17 - deploy war](./screenshots/17_deploy_war.png)

---

## ✅ Résultat Final

### Récapitulatif de l'environnement déployé

| Composant | Valeur |
|-----------|--------|
| VM | `srv-web` – Ubuntu 20.04 LTS |
| IP | `192.168.56.10` |
| RAM | 2 Go / 2 vCPUs |
| JDK 8 | `/usr/lib/jvm/java-8-openjdk-amd64` |
| JDK 11 | `/usr/lib/jvm/java-11-openjdk-amd64` ✅ (actif) |
| JDK 17 | `/usr/lib/jvm/java-17-openjdk-amd64` |
| Tomcat | v9.0.89 → `/opt/tomcat/latest` |
| Application | `http://192.168.56.10:8080/myapp` |
| Script | `/opt/tomcat/deploy.sh` |

### Commandes de nettoyage

```bash
# Depuis la machine hôte
vagrant halt        # Éteindre la VM
vagrant destroy -f  # Supprimer définitivement la VM
```

---

## 📁 Structure du Projet

```
tp-vagrant-tomcat/
├── Vagrantfile           ← Configuration de la VM
├── README.md             ← Ce fichier
├── deploy.sh             ← Script de gestion Tomcat
├── myapp/
│   ├── index.jsp         ← Page de l'application
│   └── WEB-INF/
│       └── web.xml       ← Descripteur de déploiement
└── screenshots/
    ├── 01_vagrant_up.png
    ├── 02_vagrant_status.png
    ├── 03_ssh_connexion.png
    ├── 04_apt_update.png
    ├── 05_jdk8_install.png
    ├── 06_jdk11_install.png
    ├── 07_jdk17_install.png
    ├── 08_java_alternatives.png
    ├── 09_tomcat_download.png
    ├── 10_tomcat_status.png
    ├── 11_tomcat_browser.png
    ├── 12_webapps_deploy.png
    ├── 13_app_browser.png
    ├── 14_deploy_menu.png
    ├── 15_stop_tomcat.png
    ├── 16_restart_tomcat.png
    └── 17_deploy_war.png
```

---

> 💡 **Conseil :** Pour prendre les captures d'écran, utilisez `scrot` (Linux), `Snipping Tool` (Windows) ou `Cmd+Shift+4` (macOS). Placez-les dans le dossier `screenshots/` avec les noms indiqués ci-dessus.

---

*TP réalisé dans le cadre du cours DevOps / Administration Système*
