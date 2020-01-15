---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /git/rebase/
tags: git rebase
description: Utilisation de la fonction rebase de git
title: Rebase
menus: git
---

# Fusion de plusieurs commit

Le dernier chiffre correspond aux nombres de commits qui doivent être retravaillé :

    git rebase -i HEAD~4

Changer les marqueurs pour indiquer ce qu’il y a à faire pour chaque commit, le plus ancien est en haut.

    git rebase --continue

Traiter les conflits s’il y a des conflits.

    git push origin BRANCH_NAME -f
