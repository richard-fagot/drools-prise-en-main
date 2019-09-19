Ce tutoriel présente les étapes permettant d'installer et de configurer un environnement fonctionnel de **drools** en version **7.26.0.Final**.

# Prérequis
L'installation décrite ci-après a été réalisée sous **Windows 10 Entreprise**.

# Préparatifs
Télécharger les archives suivantes :
* [OpenJDK 11](http://jdk.java.net/archive/) ;
* [Wildfly 14.0.1.Final](https://wildfly.org/downloads/) ;
* [Drools KIE Server 7.15.0.Final](https://download.jboss.org/drools/release/) récupérer l'archive `kie-server-distribution-7.15.0.Final.zip` ;
* [Drools Business Central 7.15.0.Final](https://download.jboss.org/drools/release/) récupérer l'archive `business-central-7.15.0.Final-wildfly14.war`.

## Problèmes avec les versions suivantes
Avec un *JDK 12*, un *Wildfly 17* et un *Drools 7.26* on rencontre des problèmes tels que ceux décrit ci-après et je n'ai pas encore trouvé de solution pour les résoudre.

- Ne pas utiliser *Wildfly 14* comme suggéré dans la documentation de *drools* car au déploiement du Business Central cela génère une erreur `java.lang.NoSuchFieldException: override`.

- Ne pas utiliser le *JDK 12* car dans *Business Central*, la création d'asset devient impossible (`An error occurred while executing the Local Source Config stage..` et une stack qui contient un `Illegal character in path at index 1: ${input.root-path}`).

# Installation
1. Installer le *JDK* ;
1. Créer la variable d'environnement **JAVA_HOME** pointant vers le répertoire d'installation du *JDK* (par exemple : `C:\javafiles\jdk-11`) ;
1. Dézipper *Wildfly* et déplacer le répertoire ainsi créé dans votre espace préféré. On appellera **WILDFLY_HOME** le répertoire d'installation de *Wildfly* ;
1. Dézipper l'archive du *KIE Server* dans un répertoire temporaire ;
1. Renommer le fichier `kie-server-7.15.0.Final-ee7.war` en `kie-server.war` et le déplacer dans le répertoire de déploiement de *Wildfly* `WILDFLY_HOME\standalone\deployments` ;
1. Revenir au fichier *WAR* du *Business Central Workbench* `business-central-7.15.0.Final-wildfly14.war` et le renommer en `kie-wb.war` ;
1. Déplacer ce fichier dans le répertoire de déploiement de *Wildfly* `WILDFLY_HOME\standalone\deployments` ;
1. Créer les utilisateurs suivants en lançant le script `WILDFLY_HOME\bin\add-user.bat` ([tuto][3]) ;

``` bash
add-user -a -u kieserver -p kieserver1! -g kie-server
add-user -a -u workbench -p workbench! -g admin,kie-server
```

Dans le tableau ci-dessous on précise le *type d'utilisateur*. Il est à choisir si on passe par àdd-user`sans arguments. Mais je ne sais pas si cela a un impact.

|Nom        | Groupes   |Type d'utilisateur|Source  |
|-----------|-----------|------------------|--------|
|kieserver  |kieserver1!|Management User   |[doc][1]|
|workbench  |workbench! |Application User  |[doc][2]|


# Configuration
Le *Business Central Workbench* demande pas mal de mémoire : ouvrir le fichier `WILDFLY_HOME\bin\standalone.conf.bat` (apparemment spécifique à [*Windows*](https://stackoverflow.com/questions/24959128/how-to-increase-heap-memory-for-wildfly)) et positionner la valeur de `JAVA_OPTS` ainsi : `Xms256m -Xmx4G -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=1024m`

Dans un [Dockerfile](https://hub.docker.com/r/jboss/business-central-workbench/dockerfile) fournit par *JBoss* on trouve d'autres valeurs mais je ne les ai pas testées.

# Démarrage du serveur
Exécuter la commande suivante dans `WILDFLY_HOME\bin` :
``` bash
standalone --server-config=standalone-full.xml -Dorg.kie.server.id=wildfly-kieserver -Dorg.kie.server.location=http://localhost:8080/kie-server/services/rest/server -Dorg.kie.server.controller=http://localhost:8080/kie-wb/rest/controller
```

Prendre soin notamment de la variable `org.kie.server.id` qui permet d'identifier plus facilement le serveur d'exécution des règles.

Si on se contente du fichier `standalone.xml` cela provoque une erreur du type *Required services that are not installed* ([ref](https://stackoverflow.com/questions/29126598/deployment-jbpm-console-war-from-eclipse-service-service-jboss-ejb-default-reso)).

# Références
[1]: https://docs.jboss.org/drools/release/7.26.0.Final/drools-docs/html_single/#_installing_the_kie_server
[2]: https://docs.jboss.org/drools/release/7.26.0.Final/drools-docs/html_single/#_wb.usermanagement
[3]: http://www.mastertheboss.com/jboss-jbpm/drools/getting-started-with-business-central-workbench


