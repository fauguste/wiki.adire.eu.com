---

layout: default
permalink: /aws/website_jekyll_cloudfront_s3/
tags: jekyll cloudfront S3 travis
description: Création d'un site web statique en utilisant jekyll, S3 et cloud Front en utilisant travici pour publier le site.

title: Jekyll, cloudfront et S3.
menus: aws
---
# Création d'un site web statique en utilisant jekyll, S3 et cloud Front.

## Introduction

[Jekyll](https://jekyllrb.com/) est logiciel permettant de transformer du texte en un site web site.
Cet article décrit comment héberger un site web statique généré par jekyll sur AWS avec l'architecture ci-dessous :

<img src="/images/jekyll_s3_cloudfront.png" width="512" class="img-fluid border rounded" alt="jekyll_s3_cloudfront">

## Mise en oeuvre S3

Créer un bucket S3 avec la visibilité publique et les options suivantes :
- _Hébergement de site Web statique_ doit être activé. Le point de terminaison doit être noté car il sera utilisé pour la mise en œuvre de cloudfront.
- La _stratégie de compartiment_ (onglet autorisation) doit être configuré avec l’option ci-dessous (remplacer BUCKETNAME par le nom de votre bucket) :
````
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKETNAME/*"
        }
    ]
}
````

Pour terminer la configuration de S3, il faut activer l'option `Static website hosting` / `Use this bucket to host a website`.
Il faut noter le `endpoint` présent dans cette option qui sera utilisé dans la configuration de cloud front.
## Créer un compte IAM permettant à travis de publier le site dans s3

Il faut d'abord créer une policy dans la console IAM permettant de lire et écrire dans le bucket.

Il faut ensuite ajouter la clé générée dans la configuration de travis CI :

<img src="/images/jekyll_s3_config_travis.png" width="512" class="img-fluid border rounded" alt="jekyll_s3_travisconfig">

## Configuration de travis

Le déploiement  utilise l'utilitaire [s3_website](https://www.npmjs.com/package/s3-website).

Voici le fichier `.travis.yml` à configurer pour que le site soit publié dans un bucket S3 à chaque tag posé.
````
language: ruby
rvm:
  - 2.4.1
jdk:
  - openjdk7

before_script:
  - chmod +x ./script/cibuild
  - chmod +x deploy

script: ./script/cibuild

env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer

addons:
  apt:
    packages:
    - libcurl4-openssl-dev

sudo: false # route your build to the container-based infrastructure for a faster build

cache: bundler # caching bundler gem packages will speed up build

# Optional: disable email notifications about the outcome of your builds
notifications:
  email: false

deploy:
  provider: script
  script:  bash deploy
  skip_cleanup: true
  on:
    tags: true
````

Le fichier `script/cibuild` :
````
#!/usr/bin/env bash
set -e # halt script on error

gem install s3_website

bundle exec jekyll build
````

Ainsi que le fichier `deploy`

````
gem install s3_website
s3_website push
````

et le fichier `s3_website.yml`
````
s3_id: <%= ENV['S3_ACCESS_KEY_ID'] %>
s3_secret: <%= ENV['S3_SECRET_KEY'] %>
s3_bucket: BUCKETNAME

max_age: 3000
cache_control: public, no-transform, max-age=1200, s-maxage=1200

ignore_on_server:
 - Readme.md
 - script
 - deploy
````
## Mise en oeuvre CloudFront

*Prérequis* : Disposer d'un certificat HTTPS valide dans AWS certificate manager.

Il faut créer avec distribution Web CloudFront avec les caractéristiques suivantes :
- `Origin Domain Name` : endpoint de S3 (Cf. configuration de S3)
- `Viewer Protocol Policy` : Redirect HTTP to HTTPS
- `Alternate Domain Names` : Mettre le nom de domaine du site.
- `SSL Certificate` : Choisir le certificat HTTPS correspond au domaine.

Laissez les autres options par défaut.

Pour terminer, il faut renseigner ajouter l'enregistrement DNS :
````
domaine CNAME XXXXXXXXXXX.cloudfront.net.
````

Patienter quelques minutes et le site sera accessible.
