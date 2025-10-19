
```shell
sudo apt update
sudo apt upgrade
sudo apt install docker.io
sudo apt install docker-compose
# RESTART YOUR VM
# Copy the labs to a directory in your system, then open a terminal to that directory
tar -xf peh-web-labs.tar.gz
cd labs
sudo docker-compose up
# run in background : docker-compose up -d

#The final step is to set some permissions for the webserver, this is needed for the file upload labs and the capstone challenge.
./set-permissions.sh

#check on containers running 
sudo docker ps -a

# stop docker container
sudo docker-compose stop

# remove container
sudo docker rm $(sudo docker ps -aq)
```

Browse to <http://localhost>
The first time you load the lab the database will need to be initialized, just follow the instructions in the red box by clicking the link, then coming back to the homepage.

## Reset Labs
* if you somehow break the labs, Browse to <http://localhost/ini.php> , to reset all
