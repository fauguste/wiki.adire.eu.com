---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /solr/documents-enfant-et-parent.html
tags: solr
description: Gestion des documents enfants et parents dans solr.
title: Documents enfant et parent
menus: solr
---
# Solr

## Documents enfant et parent
[Solr](http://lucene.apache.org/solr/) permet de gérer des documents enfant et parent.

Ceci peut être pratique pour modéliser des listes de documents.

La documentation de solr est [ici](https://cwiki.apache.org/confluence/display/solr/Other+Parsers#OtherParsers-BlockJoinQueryParsers).

### Création d’un document
La syntaxe permet de déclarer des enfants est relativement simple, il faut insérer une balise doc dans la balise docdu parent comme ceci :
````
<add>
  <doc>
    <field name="id">parent1</field>
    <field name="title">parent 1</field>
    <field name="content_type">parentDocument</field>
    <doc>
      <field name="id">enfant1</field>
      <field name="comments">enfant multiple 1</field>
      <field name="content_type">childDocument</field>
    </doc>
    <doc>
      <field name="id">enfant2</field>
      <field name="comments">enfant multiple 1</field>
      <field name="content_type">childDocument</field>
    </doc>
  </doc>
  <doc>
    <field name="id">parent2</field>
    <field name="title">parent 2</field>
    <field name="content_type">parentDocument</field>
    <doc>
      <field name="id">enfant1</field>
      <field name="comments">enfant unique</field>
      <field name="content_type">childDocument</field>
    </doc>
  </doc>
</add>
````

### Recherche

#### Recherche sur les parents

Cette requête retourne tous les parents qui ont un enfant qui contient unique le champ comments :

````
{!parent which="content_type:parentDocument"} comments:unique
````

Cette requête retourne tous les parents qui ont un enfant qui contient enfant le champ comments :
````
{!parent which="content_type:parentDocument"} comments:enfant
````
