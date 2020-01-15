---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /aws/aelb_startssl/
tags: git
description: Utilisation des certificats startssl avec un ELB.

title: ELB et startssl
menus: aws
---
# ELB et startssl
[ELB](http://aws.amazon.com/elasticloadbalancing/) est le load balancer managé mis à disposition par AWS.

[StartSsl](https://www.startssl.com/) permet de fabriquer des certificats HTTPs gratuitement reconnu par tous les navigateurs.

## Génération du certificat avec start SSL
````
sudo openssl genrsa -out test.exemple.com.key 2048
sudo openssl req -new -key test.exemple.com.key -out test.exemple.com.csr
````
La clé privé est test.exemple.com.key, il faut mettre le contenu du fichier test.exemple.com.csr dans l’IHM de startSSL.

## Configuration de ELB
Ci dessous les 3 noms de fichier à renseigner dans ELB pour installer les certificats SSL sur le load balancer
<img src="/images/Amazon-ELB.png" class="img-fluid" alt="putty">
<img src="/images/startSSLcertificat.png" class="img-fluid" alt="putty">
