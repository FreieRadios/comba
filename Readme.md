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


## Docker Compose example

    version: '2'
    services:
      comba:
      privileged: true
    image: comba/comba
    environment:
      COMBA_WEB_ADMIN: comba
      COMBA_WEB_ADMIN_PASSWORD: comba123
      CALENDERURL "http://my.real.jimtwawl.calender.tld/index.php?option=com_jimtawl&view=calendar&format=json&from=#datefrom#&to=#dateto#"
    ulimits:
      nofile:
        hard: 40000
        soft: 20000
    volumes:
    - ./etc/comba:/etc/comba
    - ./sqlite:/opt/comba/var/sqlite
    - ./mongo-data:/data
    - ./redis-data:/var/redis/data/
    - /srv/comba/audio:/var/audio
    - ./plugins-custom:/opt/comba/plugins/custom    
    ports:
    - 8088:8000/tcp
