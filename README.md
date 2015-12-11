Log.io - Real-time log monitoring in your browser
=================================================

Doc d'origine: https://github.com/NarrativeScience/Log.io 

## Pré-requis installations

- nodejs

ajouter dans l'ENC des VM concernées: 

      package_manager:
        actions:
          to_install:
            rpms:
              - vsc_nodejs-0.12.2

puis lancer un refresh puppet agent: 

    sudo /usr/bin/puppet agent -t

## Installation

    wget "http://nexus/service/local/artifact/maven/content?g=org.log.io&a=log.io&v=RELEASE&c=vsct&r=thirdparty&e=tar.gz" -O log.io-vsct.tar.gz
    tar -xzf log.io-vsct.tar.gz
    mv package/ logio

Url d'accès par défaut: http://main.host.server:54007/

## Configuration

Voir le fichier de configuration dans conf/harvester.conf pour exemple, le supprimer par la suite


### Serveur principal

Récupérer et modifier à souhait les fichiers [BOOT](http://gitlab.socrate.vsct.fr/dvs_tools/Log.io/raw/master/bin/vsct/BOOT) & [SHUT](http://gitlab.socrate.vsct.fr/dvs_tools/Log.io/raw/master/bin/vsct/SHUT) qui sont par défaut à mettre dans **$HOME**


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


### Serveur(s) secondaire(s)

Dans le cas de fichiers où les fichiers à monitorer sont sur plusieurs instances, il est important de raccrocher les watchers au serveur de monitoring.

Récupérer et modifier à souhait les fichiers [BOOT](http://gitlab.socrate.vsct.fr/dvs_tools/Log.io/raw/master/bin/vsct/BOOT) & [SHUT](http://gitlab.socrate.vsct.fr/dvs_tools/Log.io/raw/master/bin/vsct/SHUT) qui sont par défaut à mettre dans **$HOME**

- Modifier le fichier BOOT: 

**Supprimer** la partie qui démarre le serveur principal, laisser uniquement le démarrage des watchers harvesters

- Fichier de conf dans logio/conf/harvester-**nom_instance**.conf: 


        exports.config = {
          nodeName: "WAS-SERVER-INSTANCE",
          logStreams: {
            stream1: [
        	"/appl/wasblabla/logs/logfile.txt"
            ],
          },
          server: {
*nom du host sur lequel se connecter*

            host: 'host_du_serveur_principal_de_monitoring',
            port: 54006
          }
        }


### Cron

    #every working day at 06:00
    00 06 * * 1-5 $HOME/BOOT