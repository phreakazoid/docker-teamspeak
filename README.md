[![](https://images.microbadger.com/badges/image/phreakazoid/docker-teamspeak:alpine.svg)](https://microbadger.com/images/phreakazoid/docker-teamspeak:alpine "Get your own image badge on microbadger.com")[![](https://images.microbadger.com/badges/version/phreakazoid/docker-teamspeak:alpine.svg)](https://microbadger.com/images/phreakazoid/docker-teamspeak:alpine "Get your own version badge on microbadger.com")[![Docker Pulls](https://img.shields.io/docker/pulls/phreakazoid/docker-teamspeak.svg)](https://hub.docker.com/r/phreakazoid/docker-teamspeak/)[![GitHub issues](https://img.shields.io/github/issues/phreakazoid/docker-teamspeak.svg)](https://github.com/phreakazoid/docker-teamspeak/issues)

## docker-teamspeak:alpine

Alpine+glibc with TS3 Server.

### Summary
* Alpine+glibc + Teamspeak 3 Server
* Some files can be injected to host:
  * query_ip_whitelist.txt
  * query_ip_blacklist.txt
  * logs
  * files (Not yet)
  * ts3server.sqlitedb 
  * licence (Maybe; Dont have one)
  * ts3server.ini (Not tested)

### Update Notice
I made bigger updates to the Dockerfile to simplify and streamline the whole process. Please read the following infos carefully!
The old image used root to run the ts3-server and there all created files had root-permission. The new image uses a dedicated user (ts3) with a default UID of 1000 which can be overridden with an ENV-variable (TS3_UID). So in case you want to use old data you most probably need to chown the old files to the new user/uid.
Also the volumes inside the container have changed, the strcture itself should be the same - so the docker run command will differ slightly from the old version.

### Usage v2
I recommend to use the new 'docker volume' command in conjunction with this TS3-Container, but its obviously up to you.

#### docker volume create (Since docker-engine 1.9 - RECOMMENDED) 
```
docker volume create --name ts3-data
docker run --name=ts3 -p 9987:9987/udp -p 30033:30033 -p 10011:10011 -v ts3-data:/home/ts3/data phreakazoid/docker-teamspeak:alpine
```

#### data container

```
# create the data container
docker run --name=ts3-data --entrypoint /bin/true phreakazoid/docker-teamspeak:alpine
# Now start the actual TS3-Server
docker run --name=ts3 -d --volumes-from ts3-data -p 9987:9987/udp -p 30033:30033 -p 10011:10011 phreakazoid/docker-teamspeak:alpine
```

The data-container does not need to be running for this to work.

#### Mounted Host-directory
```
docker run --name ts3 -d -p 9987:9987/udp -p 30033:30033 -p 10011:10011 -v {FOLDER}:/home/ts3/data phreakazoid/docker-teamspeak:alpine
```

If you experience permission problems, especially after an upgrade, you can use the TS3_UID-env to set the user for the teamspeak-server process (inside the container). When using an mounted host-directory, the owner of the files will be the UID of this internal user (default is 1000)

```
docker run --name ts3 -d -p -e TS3_UID=1001 9987:9987/udp -p 30033:30033 -p 10011:10011 -v {FOLDER}:/home/ts3/data phreakazoid/docker-teamspeak:alpine
```
This would change the internal user to an UID of 1001.  


#### MariaDB

This is still WIP.
    
### Admin Secret
After starting the container you probably want to get the Admin secret with:
`sudo docker logs ts3` 
    
### Upgrading
Just stop and remove the old container, then start again at "Creating container". You may have to pull the image again if its not updating.
CAUTION: Didnt test if all files are really persisted or if the TS3 process overwrites some files. So make sure you have a backup. 

### SELinux
If your host uses SELinux it may be necessary to use the **:z** option:
```
docker run --name ts3 -d -p 9987:9987/udp -p 30033:30033 -p 10011:10011 -v /data/teamspeak:/home/ts3/data:z phreakazoid/docker-teamspeak:alpine
