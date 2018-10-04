---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /ubuntu/securisation.html
tags: git
description: Sécurisation des serveurs Ubuntu

title: Sécurisation
menus: ubuntu
---
# Sécurisation serveur ubuntu
Cette article liste les composants à installer et/ou configurer pour sécuriser un serveur Ubuntu 14.04

## Fail2ban

Un module qui bannit les adresses IP des clients en fonction de règles prédéfinies : https://help.ubuntu.com/community/Fail2ban Le client est banni via des iptables sur un serveur au bout de N essai (maxretry) pour une durée paramétrable (bantime). On peut définir une liste blanche d’IP (ignore). Il y a un ensemble de filtre disponible sur ubuntu dans le répertoire :
````
/etc/fail2ban/filter.d
````
### Création d’une règle customisée
Vous pouvez créer des règles customisées pour interdire une adresse IP qui fait des tentatives infructueuses de login.
Pour cela, les tentatives infructueuses doivent être loguée dans l’application.

## Mise à jour automatique des patchs de sécurité
Pour sécuriser un serveur, il faut appliquer les patchs de sécurité [diffusés régulièrement par Ubuntu](http://www.ubuntu.com/usn/). Pour ceci, il faut suivre la procédure qui est décrite [ici](https://help.ubuntu.com/community/AutomaticSecurityUpdates). Certains patch affecte le noyau, pour qu’ils soient pris ne compte, il faut redémarrer le serveur. De la même manière qu’il faut automatiser les patchs, il faut automatiser des reboot régulier des serveurs.

## Gestion des clé d’accès au serveur
L’accès aux serveurs DOIT s’effectuer via des clés SSH, il ne faut jamais utiliser les login / mot de passe. La clé privée SSH NE DOIT PAS être échangée et DOIT être protégée par un mot de passe. Pour interdire les connexions ssh via mot de passe, il faut s’assurer que la directive ci dessous soit renseignée dans le fichier _sshd_config_
````
PasswordAuthentication no
````
L’utilisateur Root ne doit pas pouvoir se loguer à distance.
````
PermitRootLogin no
````
## Génération d’une clé SSH sous windows
Il faut utilisé l’outil [puttyGen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html). Il faut lancer la programme et aller dans Key / « Generate Pair Key » (attention, il faut agiter la souris du PC pour générer de l’aléatoire). Dans la partie commentaire, il faut mettre un commentaire permettant d’identifier la clé, il faut aussi saisir un mot de passe pour protéger la clé.
<img src="/images/puttygen-1.png" class="img-fluid" alt="putty">


Enfin, il faut :
1. sauvegarder la clé privée (save private key) et ne l’envoyer à personne.
2. Envoyer le contenu de la fenêtre qui commence par ssh-rsa AAA…… à l’administrateur du serveur. Ce contenu correspond à la clé publique.
3. L’administrateur du serveur doit ajouter dans le fichier .ssh/authorized_keys de l’utilisateur unix avec lequel la clé pourra être utilisée.

## Reboot
Il faut configurer un reboot régulier du serveur.

## Ntpd
Permet de garder le serveur à l’heure : https://help.ubuntu.com/14.04/serverguide/NTP.html

## Suppression des noyaux non utilisés
Les mises à jour de sécurité mettent à jour le noyau et laisse sur le serveur des noyaux non utilisé, pour les purger, il faut :
````
/usr/bin/dpkg -l 'linux-*' | /bin/sed '/^ii/!d;/'"$(uname -r | /bin/sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | /usr/bin/xargs sudo /usr/bin/apt-get -y purge
````
## Apache
Ce chapitre décrit les différents paramètres à corriger dans la configuration apache suite à son installation.

### Redirection des pages d’erreur
L’objectif de ce point est d’afficher des pages d’erreur propre aux utilisateurs et d’éviter d’afficher des pages d’erreur contenant le numéro de version du serveur ou tout autres informations.
Pour cela, il faut écrire une page d’erreur et la mettre dans le DocRoot de apache.
Fichier _/etc/apache2/conf-enabled/localized-error-pages.conf_
````
ErrorDocument 500 /error.html
ErrorDocument 404 /error.html
ErrorDocument 404 /error.html
ErrorDocument 402 /error.html
````
### HTTPS
Supprimer les mécanismes HTTPs obsolète :
````
SSLHonorCipherOrder on
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite ALL:!DH:!EXPORT:!RC4:+HIGH:+MEDIUM:!LOW:!aNULL:!eNULL
````
### Signature du serveur
L’objectif est de faire en sorte que le serveur retourne le moins d’information possible.

Editer le fichier /etc/apache2/conf-available/security.conf et ajuster les variables ci dessous :
````
ServerSignature Off
ServerTokens Prod
````
Installer le mod-security2 d’apache :
````
apt-get install libapache2-mod-security2
````
Créer le fichier _/etc/modsecurity/modsecurity.conf_ et y mettre le contenu ci-dessous :
````
SecServerSignature " "
````
### Clickjacking
Empêcher le Clickjacking.
````
<ifModule mod_headers.c>
    Header set X-XSS-Protection "1; mode=block"
    Header always append X-Frame-Options SAMEORIGIN
    Header set X-Content-Type-Options: "nosniff”
</ifModule>
````
### Interdire de lister les répertoires
L’objectif est d’empecher des lires les répertoires présents sur le serveur et d’exposer des fichiers secret par accident.

Supprimer toutes les options Indexes de la conf apache.

## Php
Ce chapitre décrit les différents paramètres à corriger dans la configuration php suite à son installation.
### Sécurisation des cookies
Fichier _/etc/php5/apache2/php.ini_
````
session.cookie_secure = On (si mise en place du HTTPS)
session.cookie_httponly = On
````
Si vous utilisez la fonction setcookie, il faut mette les paramètres secure et httponly à true.
````
setcookie($_SERVER["cookie-name"],$result->access_token, time()+3600*24*365*10, "/", $_SERVER['HTTP_HOST'], true, true);
````
### Suppression des headers
Dans le fichier _/etc/php5/apache2/php.ini_ mettre la variable expose_php à off
## AWS
Ce chapitre décrit comment profiter d’AWS pour augmenter la sécurité.
### Utilisateurs AWS
Chaque utilisateurs doit disposer un compte individuel d’accès à AWS.
### Groupe de sécurité
Chaque serveur doit être placé dans un groupe de séucité correspondant à son rôle. Seul les ports nécessaires doivent être autorisés par le groupe de sécurité.
### Logs des actions AWS
CloudTrail doit être activé sur la console AWS.
### Centralisation des logs
Les logs doivent être centralisés dans CloudWatch.

Pour cela, il faut créer un clé dans IAM avec les droits ci dessous :
````
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:*"
      ],
      "Resource": [
        "arn:aws:logs:*:*:*"
      ]
    }
  ]
}
````
Puis lancer les commandes ci-dessous :
````
wget https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py
sudo python ./awslogs-agent-setup.py --region eu-west-1
````
Renseigner les points demandés.

Le fichier de configuration se trouve _/var/awslogs/etc/awslogs.conf_.

Voici un exemple d’un bloc de configuration qu’il faut afficher par fichier :
````
[/var/log/syslog]
datetime_format = %Y-%m-%d %H:%M:%S
file = /var/log/syslog
buffer_duration = 5000
log_stream_name = /var/log/syslog
initial_position = start_of_file
log_group_name = AppliName
````
### Création d’alarme sur les logs
Vous pouvez ajouter des alarmes sur des paterns présents dans les logs. Par exemple, 10 tentatives de connexion infructueuse en 5 minutes.

## Outils de contrôle de sécurité

1. OpenVAS
2. Nmap
3. Skipfish
4. Wapiti
5. Wikto
6. Lyris / lynis
7. Chkrootkit
