---
layout: default
permalink: /solr/index-documents-with-tika/
tags: solr tika
description: Indexation de documents présent dans un répertoire dans solr à l'aide de tika.
title: Tika
menus: solr
---
# Indexation de documents avec solr et tika.

Cet article décrit comment indexer dans solr des documents (word, pdf, excel, ...) présents dans un répertoire.

L'extraction des données et metadonnées présentes dans un document est prise en charge par [Tika](https://tika.apache.org/) qui est intégré à solr.

Les tests ont été effectués avec [solr 7.5.0](https://lucene.apache.org/solr/guide/7_5/installing-solr.html) sur le conteneur docker [solr:7](https://hub.docker.com/_/solr/).

## Configuration de solr.

Ajout dans le fichier `solrconfig.xml` des librairies nécessaires à l'import des fichiers :
````
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
````

Déclarer l'url d'import dans le fichier `solrconfig.xml`  :
````
<requestHandler name="/dataimport" class="solr.DataImportHandler">
  <lst name="defaults">
    <str name="config">data-config.xml</str>
  </lst>
</requestHandler>
````

Il faut ensuite déclarer les champs utilisés dans le fichier `managed-schema` :
````
<field name="id" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="category" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="fileAbsolutePath" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="fileSize" type="plong" indexed="true" stored="true" multiValued="false"/>
<field name="fileLastModified" type="pdate" indexed="true" stored="true" multiValued="false"/>
<field name="file" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="author" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="text" type="text_general" indexed="true" stored="true" multiValued="false"/>
````

Il faut créer un fichier `data-config.xml` avec le contenu suivant (voir les commentaires):
````
<dataConfig>
  <!--
     Data source de type fichier :
     https://wiki.apache.org/solr/DataImportHandler#FileListEntityProcessor
    -->
  <dataSource type="BinFileDataSource"/>
  <document>
    <!--
       base dir est à adapter.
       Utilisation des [transformer](https://wiki.apache.org/solr/DataImportHandler#Transformer) pour modifier les données extraite.
    -->
    <entity name="files" dataSource="null" rootEntity="false" processor="FileListEntityProcessor" baseDir="/opt/solr/server/solr/mycores/documents2" fileName=".*" recursive="true" onError="skip" transformer="TemplateTransformer,RegexTransformer">
      <!-- Suppression du chemin relatifs à l'aide RegexTransformer -->
      <field column="id" regex="/opt/solr/server/solr/mycores/documents2/(.*)" replaceWith="$1" sourceColName="fileAbsolutePath"/>
      <field column="fileSize" name="size"/>
      <field column="fileLastModified" name="lastModified"/>
      <!-- Ajout d'une variable statique au document à l'aide de TemplateTransformer -->
      <field column="category" template="documents"/>
      <entity name="documentImport" processor="TikaEntityProcessor" url="${files.fileAbsolutePath}" format="text" >
        <field column="file" name="fileName"/>
        <field column="author" name="author" meta="true"/>
        <field column="title" name="title" meta="true"/>
        <field column="text" name="text"/>
      </entity>
    </entity>
  </document>
</dataConfig>
````

## Lancement de l'indexation

Le lancement de l'indexation s'effectue via l'IHM d'administration de solr : http://localhost:8983/solr/#/<CORE_NAME>/dataimport//dataimport

<img src="/images/solr_datahandle_index.png" width="512" class="img-fluid border rounded" alt="jekyll_s3_cloudfront">
