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

## Configuration

Voir le fichier de configuration dans conf/harvester.conf pour exemple, le supprimer par la suite

### Serveur principal

- Exemple de fichier BOOT: 


        #!/bin/sh
        
        LOCK_FILE=$HOME/logio.lockboot
        LOG_DIR=$HOME/logio/logs
        PID_FILE=$HOME/logio.pid
        
        if [ ! -f "$LOCK_FILE" ]; then
        	mkdir -p $LOG_DIR || (touch $LOCK_FILE;exit 1)
        	echo "export nodejs to PATH"
        	export PATH=$PATH:/export/product/nodejs/0.12.2/usr/bin || (touch $LOCK_FILE;exit 1)
        	echo "export nodejs [done]"

*permet de démarrer un serveur de monitoring de logs accessible par les clients web*

        	# starting server
        	$HOME/logio/bin/log.io-server > $LOG_DIR/server.log &
        	# get pid number
        	PID=`echo $!`
        	echo $PID > $PID_FILE
        	ps -p $PID || (touch $LOCK_FILE;exit 1)
        	echo "server started"
        
        	sleep 5

*permet de démarrer un watcher de logs rattaché au serveur de monitoring*

        	# starting harvesters
        	$HOME/logio/bin/log.io-harvester >> $LOG_DIR/harvester.log &
        	PID=`echo $!`
        	echo $PID >> $PID_FILE
        	ps -p $PID || (touch $LOCK_FILE;exit 1)
        	echo "watchers harvester started"
        
        	echo "pid file: $PID_FILE"
        else
        	echo "there was an issue, please have a look to the launchers and remove the lockboot file $LOCK_FILE"
        fi



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

- Exemple de fichier BOOT: 


        #!/bin/sh
        
        LOCK_FILE=$HOME/logio.lockboot
        LOG_DIR=$HOME/logio/logs
        PID_FILE=$HOME/logio.pid
        
        if [ ! -f "$LOCK_FILE" ]; then
        	mkdir -p $LOG_DIR || (touch $LOCK_FILE;exit 1)
        	echo "export nodejs to PATH"
        	export PATH=$PATH:/export/product/nodejs/0.12.2/usr/bin || (touch $LOCK_FILE;exit 1)
        	echo "export nodejs [done]"

**pas besoin ici de déclarer un serveur de monitoring: on va simplement s'y connecter au travers de la conf des watchers**

*permet de démarrer un watcher de logs rattaché au serveur de monitoring*

        	# starting harvesters
        	$HOME/logio/bin/log.io-harvester >> $LOG_DIR/harvester.log &
        	PID=`echo $!`
        	echo $PID >> $PID_FILE
        	ps -p $PID || (touch $LOCK_FILE;exit 1)
        	echo "watchers harvester started"
        
        	echo "pid file: $PID_FILE"
        else
        	echo "there was an issue, please have a look to the launchers and remove the lockboot file $LOCK_FILE"
        fi


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