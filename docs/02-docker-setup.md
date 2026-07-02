# 2. Docker Setup

[← Back to overview](../README.md) · Previous: [1. Server Setup](01-server-setup.md)

The whole setup is done as the `sebserver` user.

## Clone the SEB Server setup repository

```bash
su sebserver
cd ~

git clone https://github.com/SafeExamBrowser/seb-server-setup
cd seb-server-setup
git checkout rel-1.5

cd docker/prod/bundled/
```

## Adjust the Docker files

Replace the content of `docker-compose.yml` with the following (uses environment variables from a `.env` file, see below):

```yaml
version: '3.5'
services:
  mariadb:
    image: "mariadb:10.5"
    container_name: seb-server-mariadb
    environment:
        # Set your DB root password here.
        # Note: If you don't want to use the DB root account for connection you have to configure this within the DB
        #       first and then set also the user in seb-server service below with spring_datasource_username
        - MYSQL_ROOT_PASSWORD=${DB_SA_PWD}
    ports:
        - 3306:3306
    volumes:
        - seb-server-config:/etc/mysql/conf.d
        - seb-server-mariadb:/var/lib/mysql
        - ./config/mariadb/config.cnf:/etc/mysql/conf.d/config.cnf
    networks:
        - seb-server-network
    restart: unless-stopped

  seb-server:
      image: "anhefti/seb-server:v2.0-stable"
      container_name: seb-server
      volumes:
        - seb-server-config:/sebserver/config
        - ./config/spring:/sebserver/config/spring
      environment:
        # Service runtime settings
        - JAVA_HEAP_MIN=1G
        - JAVA_HEAP_MAX=8G
        - spring_profiles_active=gui,ws,prod-ws,prod-gui,prod

        # SEB Server password, internally used to en-/decrypt sensitive data secure internal communication
        # NOTE: These must match with the other sebserver_password set on other services
        - sebserver_password=${SEBSERVER_PWD}

        # Data Base settings
        # NOTE: these must match with the DB connection settings on mariadb service
        - spring_datasource_username=root
        - sebserver_mariadb_password=${DB_SA_PWD}
        - datastore_mariadb_server_address=seb-server-mariadb
        - datastore_mariadb_server_port=3306

        # Set the below URL component settings from where the web-service is externally available
        # NOTE: This must be the address from that the service is externally available (not network internal address)
        - sebserver_webservice_http_external_scheme=https
        - sebserver_webservice_http_external_servername=${DNS_NAME}
        - sebserver_webservice_http_external_port=${BASE_PORT}
        - sebserver_webservice_autologin_url=https://${DNS_NAME}:${BASE_PORT}

        # Screen Proctoring settings and passwords
        # NOTE: these must match with the respective equivalents on sps-webservice
        - sebserver_feature_exam_seb_screenProctoring_bundled_url=https://${DNS_NAME}:${SPS_WEB_PORT}
        - sps_sebserver_client_secret=${SEBSERVER_PWD}
        - sps_sebserver_password=${SEBSERVER_PWD}

        # JMX monitoring settings. To enable set to true, and uncomment port mapping below and add password in config/jmx directory
        - MONITORING_MODE=false
        # - JMX_PORT=9090
        #ports:
        #  - 9090:9090
      logging:
        driver: "json-file"
        options:
            mode: "non-blocking"
            max-size: "200k"
            max-file: "10"
      networks:
          - seb-server-network
      depends_on:
          - "mariadb"
      restart: unless-stopped

  sps-webservice:
    image: "anhefti/seb-sps:v1.0-stable"
    container_name: sps-webservice
    environment:
      # Service runtime settings
      - JAVA_HEAP_MIN=1G
      - JAVA_HEAP_MAX=6G
      - SERVER_PORT=8090
      - spring_profiles_active=prod

      # SEB Server password, internally used to en-/decrypt sensitive data secure internal communication
      # NOTE: These must match with the other sebserver_password set on other services
      - sebserver_password=${SEBSERVER_PWD}

      # Data Base settings
      # NOTE: these must match with the DB connection settings on mariadb service
      - spring_datasource_username=root
      - spring_datasource_password=${DB_SA_PWD}
      - datastore_mariadb_server_port=3306
      - datastore_mariadb_server_address=seb-server-mariadb
      - sps_data_store_adapter=FULL_RDBMS

      # Set the below URL component settings from where the screen proctoring web-service is externally available
      # NOTE: This must be the address from that the service is externally available (not network internal address)
      - sps_webservice_http_external_scheme=https
      - sps_webservice_http_external_servername=${DNS_NAME}
      - sps_webservice_http_external_port=${SPS_WEB_PORT}

      # SEB Server client connection detail settings
      - sebserver_client_secret=${SEBSERVER_PWD}
      - sps_gui_redirect_url=https://${DNS_NAME}:${SPS_GUI_PORT}
      - sps_webservice_sebserver_bundle=true

      # Screen Proctoring GUI client connection detail settings
      - spsgui_client_secret=${SEBSERVER_PWD}
      - sps_init_sebserveraccount_password=${SEBSERVER_PWD}

    volumes:
      - .:/sebsps/config/spring
    networks:
      - seb-server-network
    depends_on:
      - "mariadb"
    restart: unless-stopped

  sps-guiservice:
    image: naritter/seb-sps-gui:v1.0-stable
    container_name: sps-guiservice
    environment:
      # Service runtime settings
      - NODE_ENV=prod
      - SERVER_PORT=3000

      # VITE server settings
      - VITE_SERVER_URL=https://${DNS_NAME}
      - VITE_SERVER_PORT=${SPS_GUI_PORT}

      # Screen Proctoring webservice connection settings
      - PROCTOR_SERVER_URL=https://${DNS_NAME}
      - PROCTOR_SERVER_PORT=${SPS_WEB_PORT}
      - PROCTOR_DEFAULT_URL=/admin-api/v1
      - PROCTOR_SERVER_USERNAME=spsGuiClient
      - PROCTOR_SERVER_PASSWORD=${SEBSERVER_PWD}

      # Screen Proctoring SEB Server integration mode. NOTE: always true for this setup
      - SEB_SERVER_INTEGRATED_MODE=true
    networks:
      - seb-server-network
    restart: unless-stopped

  reverse-proxy:
    image: "nginx:latest"
    container_name: seb-server-proxy
    volumes:
      - ./config/nginx:/etc/nginx/conf.d
      - ./config/nginx/certs:/sebserver/config/certs
    ports:
      - ${BASE_PORT}:${BASE_PORT}
      - ${SPS_WEB_PORT}:${SPS_WEB_PORT}
      - ${SPS_GUI_PORT}:${SPS_GUI_PORT}
    networks:
      - seb-server-network
    depends_on:
      - "mariadb"
      - "seb-server"
    restart: unless-stopped

networks:
  seb-server-network:
    name: seb-server-network

volumes:
  seb-server-config:
    name: seb-server-config
  seb-server-mariadb:
    name: seb-server-mariadb
```

Create a `.env` file in the same directory and store your own passwords and settings in it (use strong, unique passwords):

```bash
SEBSERVER_PWD=<strong-password>
DB_SA_PWD=<strong-password>
DNS_NAME=seb.example.org
BASE_PORT=443
SPS_WEB_PORT=4431
SPS_GUI_PORT=4432
```

Adjust the nginx configuration in `config/nginx/app.conf` to use your SSL certificates:

```nginx
ssl_session_cache   shared:SSL:10m;
ssl_session_timeout 10m;


server {
    listen              443 ssl;
    charset utf-8;
    access_log off;
    keepalive_timeout   70;

    server_name         localhost;
    ssl_certificate     /sebserver/config/certs/seb.example.org_chained.crt;
    ssl_certificate_key /sebserver/config/certs/seb.example.org.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://seb-server:8080;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen              4431 ssl;
    charset utf-8;
    access_log off;
    keepalive_timeout   70;

    server_name         localhost;
    ssl_certificate     /sebserver/config/certs/seb.example.org_chained.crt;
    ssl_certificate_key /sebserver/config/certs/seb.example.org.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://sps-webservice:8090;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen              4432 ssl;
    charset utf-8;
    access_log off;
    keepalive_timeout   70;

    server_name         localhost;
    ssl_certificate     /sebserver/config/certs/seb.example.org_chained.crt;
    ssl_certificate_key /sebserver/config/certs/seb.example.org.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://sps-guiservice:3000;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

## Add SSL certificates

Copy your certificate and key into the certs directory and create a chained certificate (server certificate + intermediate CA):

```bash
mkdir -p ~/seb-server-setup/docker/prod/bundled/config/nginx/certs
cd ~/seb-server-setup/docker/prod/bundled/config/nginx/certs

# copy seb.example.org.crt, seb.example.org.key and the CA certificate here, then:
cat seb.example.org.crt intermediate_ca.crt > seb.example.org_chained.crt
```

## Start Docker

```bash
docker-compose up -d
```

On the first start, an initial password for the `sebserver-admin` user is generated and printed to the console/log:

```bash
docker-compose logs seb-server
```

Log in to the web UI (`https://seb.example.org`) with `sebserver-admin` and **change this password immediately**. Afterwards the services can be restarted normally:

```bash
docker-compose start
```

## Disable self-registration

By default, anyone can register an account on the SEB Server login page. To disable this, add the following lines to `config/spring/application-prod.properties`:

```properties
# disable user registration
sebserver.gui.registering:false
```

Alternatively, set it as an environment variable in `docker-compose.yml` (dots replaced by underscores): `sebserver_gui_registering=false`. Restart the services afterwards.

## Restarting

```bash
su sebserver
cd ~/seb-server-setup/docker/prod/bundled/

docker-compose stop
docker-compose start
```

## Database access (optional)

```bash
mysql -h localhost -P 3306 -u root -p
```

---

Next: [3. Moodle Setup →](03-moodle-setup.md)
