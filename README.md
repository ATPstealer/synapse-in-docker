## Deploy for Ubuntu 22/24

### Preparation
* Create A and AAAA records for server domain name. I use test.aptmatrix.com
* Install [docker](https://docs.docker.com/engine/install/ubuntu/) and [docker-compose](https://docs.docker.com/compose/install/linux/)
* Create certs
  ```
  sudo apt install certbot -y
  certbot certonly --rsa-key-size 2048 --standalone --agree-tos --no-eff-email --email email@email.com -d atpmatrix.com
  ```
  
### Synapse server install
* Create dir /etc/synapse
* Copy from docker-compose.yml service synapse and network. Remove any dependencies for now
* Generate config like [here](https://github.com/matrix-org/synapse/tree/develop/contrib/docker)
  ```angular2html
  /etc/synapse# docker compose run --rm -e SYNAPSE_SERVER_NAME=test.atpmatrix.com -e SYNAPSE_REPORT_STATS=yes synapse generate
  ```
  It should be generated at /var/synapse/data/homeserver.yaml
* Try to run server 
  ```
  /etc/synapse# docker compose up
  ```
  Check that it works on 8008 port http://test.atpmatrix.com:8008/_matrix/static/ and stop server.

### Nginx install
* Create dir /etc/nginx/docker-config and copy there nginx config atpmatrix.com.conf with only one location ~ ^(/_matrix|/_synapse/client|/_synapse/admin)
* Change server_name and cert paths in config 
* Copy nginx service from example without unnecessary dependency
* Run server again and check HTTPS & HTTP via nginx https://test.atpmatrix.com/_matrix/static/

### Create Admin user
* Enter to docker container with synapse server and use [register_new_matrix_user](https://manpages.debian.org/testing/matrix-synapse/register_new_matrix_user.1.en.html)
  ```angular2html
  docker exec -it  synapse-synapse-1 bash
  register_new_matrix_user -u admin -p admin -a -c /data/homeserver.yaml
  ```

### Systemd unit install
* Stop docker compose
  ```angular2html
  /etc/synapse# docker compose down
  ```
* Copy systemd unit file synapse.service in /etc/systemd/system
* Run
  ```
  systemctl daemon-reload && systemctl enable synapse && systemctl start synapse
  ```
  
### Configure Postgres
* Copy db service part in docker-compose file, change password and enable depend_on db on synapse service.
* Change database part in homeserver.yaml like in example
* Restart server: systemctl start synapse
* Now you have a new database and can remove SQLite file: 
  ```angular2html
  /var/synapse/data# rm homeserver.db
  ```

### Configure admin
* Copy synapse-admin service to docker-compose file
* Add to nginx config part with location ~ ^/admin-url(/?)(.*)$
* Restart service
* Check https://test.atpmatrix.com/admin-url/#/login

### Add turn server
* Copy turn service to docker-compose file
* Copy turn_ directives from homeserver.yaml example
* Restart service
* Check audio/video calls in client

### Install browser client
* Copy element-web service to docker-compose file
* Add to nginx config part with location ~ ^/web(/?)(.*)$ 
* Restart service
* Check https://test.atpmatrix.com/web/