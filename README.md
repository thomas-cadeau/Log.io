Log.io - Real-time log monitoring in your browser
=================================================

Doc d'origine: https://github.com/NarrativeScience/Log.io 

## Pré-requis installations
node

## Installation
wget "http://nexus/service/local/artifact/maven/content?g=org.log.io&a=log.io&v=RELEASE&c=vsct&r=thirdparty&e=tar.gz" -O log.io-vsct.tar.gz
tar -xzf log.io-vsct.tar.gz
mv package/ logio

## Configuration

Voir le fichier de configuration dans conf/harvester.conf pour exemple.

# Serveur principal
- Exemple de fichier BOOT: 

    export PATH=$PATH:/export/product/nodejs/0.12.2/usr/bin
    # permet de démarrer un serveur de monitoring de logs accessible par les clients web
    $HOME/logio/bin/log.io-server &
    # permet de démarrer un watcher de logs rattaché au serveur de monitoring
    $HOME/logio/bin/log.io-harvester &

- Fichier de conf dans logio/conf/harvester-**nom_instance**.conf: 


    exports.config = {
      nodeName: "WAS-SERVER-INSTANCE",
      logStreams: {
        stream1: [
    	"/appl/wasblabla/logs/logfile.txt"
        ],
      },
      server: {
        host: 'localhost',
        port: 54006
      }
    }


# Serveur(s) secondaire(s)

Dans le cas de fichiers où les fichiers à monitorer sont sur plusieurs instances, il est important de raccrocher des watchers au serveur de monitoring.

Exemple de fichier BOOT: 

    export PATH=$PATH:/export/product/nodejs/0.12.2/usr/bin
    # pas besoin ici de déclarer un serveur de monitoring: on va simplement s'y connecter au travers de la conf des watchers
    # permet de démarrer un watcher de logs rattaché au serveur de monitoring
    $HOME/logio/bin/log.io-harvester &

- Fichier de conf dans logio/conf/harvester-**nom_instance**.conf: 


    exports.config = {
      nodeName: "WAS-SERVER-INSTANCE",
      logStreams: {
        stream1: [
    	"/appl/wasblabla/logs/logfile.txt"
        ],
      },
      server: {
        host: 'host_du_serveur_principal_de_monitoring',
        port: 54006
      }
    }