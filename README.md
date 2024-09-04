### Deploy for Ubuntu 22


* Install docker and docker-compose
* Create certs
  ```
  sudo apt install certbot -y
  certbot certonly --rsa-key-size 2048 --standalone --agree-tos --no-eff-email --email email@email.com -d atpmatrix.com
  ```
* Copy nginx and docker-compose files
* Copy systemd unit in /etc/systemd/system
* Change domain and paths in configs
* Generate config like here https://github.com/matrix-org/synapse/tree/develop/contrib/docker
* Configure homeserver.yaml. Example in repo
* Run
  ```
  systemctl daemon-reload && systemctl enable synapse && systemctl start synapse
  ```
  