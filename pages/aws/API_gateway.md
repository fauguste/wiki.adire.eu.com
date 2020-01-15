---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /aws/api_gateway/
tags: git
description: Note sur l'utilisation d'AWS gateway.

title: Amazon API Gateway
menus: aws
---
# Amazon API Gateway

API Gateway permet d’exposer une API sur internet de manière sécurisée.
Cette article montre comment configurer API Gateway pour exposer une API exposée en HTTPS sur internet avec un accès restreint.

## Exposition de l’API en HTTPS
API Gateway ne prend un compte qu’un certains nombre de certificat racine non listé dans la documentation.

Des tests effectués le 5 janvier 2016 montre que les certificats gratuit [Start SSL](https://www.startssl.com/) et [letsencrypt](https://letsencrypt.org/) ne sont pas supportés. Ceci se traduit par le message d’erreur suivant lors des appels à API Gateway lorsque le certificat racine n’est pas accepté :
````
{"message": "Internal server error"}
````
Les certificats [Gandi](https://www.gandi.net/) sont supportés.

## Configuration de l’authentification forte
L’authentification forte permet de s’assurer que seul API Gateway peut accèder à l’API que vous avez exposé sur internet.

Dans API Gateway, il faut :
1. Créer un certificat client
2. Associer ce certificat client lors du déploiement
3. Configurer votre serveur pour autoriser uniquement ce certificat client

## Configuration apache
Pour autoriser uniquement API gateway accéder à votre serveur apache, il faut ajouter le certificat client généré par AWS sur votre serveur (ici /etc/apache2/ssl/apigateway.pem) et mettre les lignes suivantes dans votre configuration apache :

````
SSLVerifyClient require
SSLVerifyDepth 1
SSLCACertificateFile /etc/apache2/ssl/apigateway.pem
````
## Configuration d’un domaine spécifique
API gateway permet d’être hébergé sous votre propre nom de domaine.
Pour cela, il vous faut votre certificat HTTPS ainsi que la possibilité d’ajouté un CNAME dans votre DNS.

Dans la suite des exemples, nous aurons créé le domaine https://api.exemple.com
## Sécurisation de l’accès à API gateway
API gateway met à disposition deux éléments distincts pour sécuriser l’accès : gestion d’une clé dans le header et/ou signature de la requête.
Les deux options peuvent être activées indépendamment l’une de l’autre.
## Gestion de la clé dans le header
Pour activer cette option, il faut activer l’option API Key Required dans les paramètres d’autorisation de la requête.
Les clés peuvent être générées dans le section API Key.

Les clients devront alors transmettre la clé dans le header sous cette forme :
````
x-api-key: <GENERATED KEY>
````
## Signature des requêtes
Pour activer cette option, il faut sélectionner l’option AWS_IAM dans les paramètres d’autorisation de la requête.
Les clients doivent alors signer les requêtes en respectant la [signature v4 d’aws](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html).
Les clés pour cette signature sont à gérer dans la console [IAM](https://aws.amazon.com/documentation/iam/).

L’utilisateur créé doit avoir les droits suivants :
````
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1452017440000",
            "Effect": "Allow",
            "Action": [
                "execute-api:invoke"
            ],
            "Resource": [
                "ARN de l'API créee"
            ]
        }
    ]
}
````
### Exemple de signature en PHP

````
<?php

require_once 'vendor/autoload.php';
use Aws\Signature\SignatureV4;
use Aws\Credentials\Credentials;

$signature = new SignatureV4("execute-api" , "eu-west-1");
$client = new GuzzleHttp\Client();
$request = new \GuzzleHttp\Psr7\Request('GET', 'https://api.exemple.com/prod/2');
$credentials = new Credentials("KEY", "SECRET");
$req = $signature->signRequest($request, $credentials);
$response = $client->send($req);
echo $response->getBody();
````
