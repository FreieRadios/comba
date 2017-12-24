# Comba Docker

## What is Comba

Comba is a modular server-side system for automation of broadcast and rebroadcasts for community radio.

E.g. Comba can record live broadcasts from the sound card at fixed times and rebroadcast them again at defined times.    

Comba needs access to a radio programme calendar via webservice such like [Jimtawl](https://hub.docker.com/r/comba/jimtawl/). It provides a web interface for montoring and to upload audio files for broadcasting. 

## Running the docker

docker run --name comba --privileged comba/comba -d

## Environment Variables

    
    USERID ""
    
    COMBA_WEB_ADMIN "comba"
    COMBA_WEB_ADMIN_PASSWORD "comba"
    ADMINMAILTO "admin@mydomain.tld"
    ADMINMAILFROM "comba@mydomain.tld"
    
    ALSA_BUFFER "16384"
    CALENDERURL "http://your.jimtawl.calender.tld/index.php?option=com_jimtawl&view=calendar&format=json&from=#datefrom#&to=#dateto#"
    RECORDERINPUT "soundcard"
    RECORDERDEVICE "default"
    PLAYERDEVICE "default"    
    
    MAILHUB "mail.server.tld"
    MAILUSER "itsyou"
    MAILPASSWORD "secret"
    MAILUSETLS "YES"
    MAILUSESTARTTLS "YES"    

### System

* **USERID**: map host user id to user comba, to have access to files in /var/audio     

### Webinterface

* **COMBA_WEB_ADMIN**: admin username
* **COMBA_WEB_ADMIN_PASSWORD**: admin password

### Programme Calender Service

* **CALENDERURL**: Url of webservice providing your radio programme calendar - usually jimtawl

**Attention:** Comba will not work without a valid calendar web service 

### Sound Settings 

* **ALSA_BUFFER**: Optimize your alsa buffer settings according to your sound card
* **PLAYERDEVICE**: Sound card device for playback
* **RECORDERDEVICE**: Sound card device for capturing
* **RECORDERINPUT**: Usually "soundcard" - you may also record from a stream


### Mail service (via ssmtp)
 
* **MAILHUB**: your mailserver host
* **MAILUSER**: email account username  
* **MAILPASSWORD**: email account password 
* **MAILUSETLS**: does your server use tls - YES/NO
* **MAILUSESTARTTLS**: does your server use starttls - YES/NO
* **ADMINMAILTO**: Recipient of service mails
* **ADMINMAILFROM**: From address for service mails


## Making databases and audio storage persistant
  
```mkdir -p /var/comba/sqlite && mkdir -p /var/comba/mongo-data/db && mkdir -p /var/comba/redis-data && mkdir -p /var/comba/audio```

User of sqlite directory and file must be user comba inside the docker container
   
So you can chown this way:

1. Detect the container name or id, e.g ```combacontainer=$(docker ps | grep comba | awk '{ print $1 }') ```
   
2. Get comba uid from container and chown the directory /var/comba/sqlite
   
```bash
combauid=$(docker exec -it $combacontainer id comba | awk '{ print $1 }' | sed -n  "s/^uid=\(.*\)(.*/\1/p") \
&& chown -R $combauid /var/comba/sqlite
``` 

Note: To get full control of the audio storage for your local user, you can use the environment variable $USERID
      
      chown  localuser /var/comba/sqlite && chown  localuser /var/comba/audio  
 
 

## Custom plugins volume
 
```mkdir -p  /var/comba/plugins-custom```  


## Docker Compose example

    version: '2'
    services:
      comba:
        privileged: true
        image: comba/comba
        environment:
          # link to the calendar service below: 
          CALENDERURL: http://calendar/index.php?option=com_jimtawl&view=calendar&format=json&from=#datefrom#&to=#dateto#
          COMBA_WEB_ADMIN: comba-admin
          COMBA_WEB_ADMIN_PASSWORD: comba-secret
          # assumed your local user has uid 1022 and he owns /var/comba/audio and  /var/comba/sqlite on docker host
          USERID: 1022
        volumes:
        - /var/comba/sqlite:/opt/comba/var/sqlite:rw
        - /var/comba/mongo-data:/data:rw
        - /var/comba/redis-data:/var/redis/data:rw
        - /var/comba/audio:/var/audio:rw      
        - /var/comba/plugins-custom:/opt/comba/plugins/custom               
        ulimits:
          nofile:
            hard: 40000
            soft: 20000
        ports:
        - 80:8000/tcp

      calendar:
        image: comba/jimtawl
        environment:
          JOOMLA_ADMINMAIL: admin@somewhere.tld
          JOOMLA_DB_HOST: joomladb
          JOOMLA_DB_NAME: jimtawl
          JOOMLA_DB_PASSWORD: secret
          JOOMLA_DB_USER: root
          JOOMLA_ADMIN_USERNAME: combaadmin
          JOOMLA_ADMIN_PASSWORD: comba123
          LOAD_JIMTAWL_SAMPLEDATA: "Yes"          
        links:
        - joomladb
        ports:
        - 81:80/tcp
                        
      joomladb:
        image: mariadb
        environment:
          MYSQL_DATABASE: jimtawl
          MYSQL_ROOT_PASSWORD: secret
