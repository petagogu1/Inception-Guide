# Inception — Complete Guide

## What the project actually wants

Deploy a working WordPress website accessible at `yourlogin.42.fr` via HTTPS, running entirely inside Docker containers on your VM, where:
- NGINX handles all incoming traffic (port 443, TLS only)
- WordPress runs the actual website
- MariaDB stores the website data

---

## Theory

**What is NGINX?**
Software that acts as a web server, reverse proxy, load balancer and HTTP cache.
- Web server: serves static files (HTML, images, CSS)
- Load balancer: if you have multiple services running, NGINX splits the internet traffic evenly
- Caching: can store copies of frequently visited pages
- Reverse proxy: sits between users and backend servers

**What is WordPress?**
Allows you to build and manage websites without needing to know code.
- The core software: handles user accounts, basic settings, and content storage
- Themes: dictate the look and layout of your website
- Plugins: add specialized features like contact forms, shopping carts, etc.

**What is MariaDB?**
A program that stores data — everything that happens on your site gets saved here. For example, when someone creates an account, writes a post, logs in, changes settings, or uploads comments — WordPress saves that information inside MariaDB.

**What is Docker?**
Software that packages an application with everything it needs so it runs the same anywhere.

**What is Docker Compose?**
A tool to define and run multi-container Docker applications, allowing you to launch and manage multiple connected containers at once using a single configuration file (`docker-compose.yml`) and one command.

---

## Step 1 — Add your user to sudoers

```bash
su -
#Then enter root pass
usermod -aG sudo yourusername
reboot
```

---

## Step 2 — Install Docker and Docker Compose

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Add your user to the docker group so you don't need sudo every time:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify everything is working:
```bash
docker --version
docker compose version
```

Also install make:
```bash
sudo apt-get install -y make
```

---

## Step 3 — Set up the project folder structure

```bash
mkdir -p ~/inception/srcs/requirements/nginx/conf
mkdir -p ~/inception/srcs/requirements/nginx/tools
mkdir -p ~/inception/srcs/requirements/wordpress/conf
mkdir -p ~/inception/srcs/requirements/wordpress/tools
mkdir -p ~/inception/srcs/requirements/mariadb/conf
mkdir -p ~/inception/srcs/requirements/mariadb/tools
mkdir -p ~/inception/secrets
touch ~/inception/Makefile
touch ~/inception/README.md
touch ~/inception/USER_DOC.md
touch ~/inception/DEV_DOC.md
touch ~/inception/srcs/docker-compose.yml
touch ~/inception/srcs/.env
touch ~/inception/secrets/db_password.txt
touch ~/inception/secrets/db_root_password.txt
touch ~/inception/secrets/credentials.txt
```

The structure the project wants:
```
inception/
├── Makefile
├── README.md
├── USER_DOC.md
├── DEV_DOC.md
├── secrets/
│   ├── db_password.txt
│   ├── db_root_password.txt
│   └── credentials.txt
└── srcs/
    ├── docker-compose.yml
    ├── .env
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile
        │   ├── conf/
        │   │   └── nginx.conf
        │   └── tools/
        ├── wordpress/
        │   ├── Dockerfile
        │   └── tools/
        │       └── wordpress_setup.sh
        └── mariadb/
            ├── Dockerfile
            └── tools/
                └── db_setup.sh
```

---

## Step 4 — Write the 3 Dockerfiles

### NGINX Dockerfile
>**So what you do now is copy each path given in each step and paste the code do not forget to change the login if the script has it**
|
|
--->`nano ~/inception/srcs/requirements/nginx/Dockerfile`

```dockerfile
FROM debian:bookworm

RUN apt-get update && apt-get install -y \
    nginx openssl && \
    rm -rf /var/lib/apt/lists/*

RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nginx.key \
    -out /etc/ssl/certs/nginx.crt \
    -subj "/C=FR/ST=IDF/L=Paris/O=42/CN=yourlogin.42.fr"

COPY conf/nginx.conf /etc/nginx/sites-available/default

EXPOSE 443

CMD ["nginx", "-g", "daemon off;"]
```

> Replace `yourlogin` with your actual 42 login.

### MariaDB Dockerfile
`nano ~/inception/srcs/requirements/mariadb/Dockerfile`

```dockerfile
FROM debian:bookworm

RUN apt-get update && apt-get install -y \
    mariadb-server && \
    rm -rf /var/lib/apt/lists/*

COPY tools/db_setup.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

EXPOSE 3306

ENTRYPOINT ["/docker-entrypoint.sh"]
```

### WordPress Dockerfile
`nano ~/inception/srcs/requirements/wordpress/Dockerfile`

```dockerfile
FROM debian:bookworm

RUN apt-get update && apt-get install -y \
    php-fpm php-mysql curl default-mysql-client && \
    rm -rf /var/lib/apt/lists/*

RUN curl -L https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -o /usr/local/bin/wp \
    && chmod +x /usr/local/bin/wp

COPY tools/wordpress_setup.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

WORKDIR /var/www/html

EXPOSE 9000

ENTRYPOINT ["/docker-entrypoint.sh"]
```

**How Dockerfiles work (explained simply):**

`FROM debian:bookworm` — Start from a clean Debian OS. Like a fresh Linux install with nothing on it.

`RUN apt-get update && apt-get install -y mariadb-server` — Install software inside that clean Debian. Same as typing apt-get install in your terminal, but inside the container. The `rm -rf /var/lib/apt/lists/*` cleans up the package cache to keep the image small.

`COPY tools/db_setup.sh /docker-entrypoint.sh` — Take a script from your computer and put it inside the container.

`RUN chmod +x /docker-entrypoint.sh` — Make that script executable.

`EXPOSE 3306` — Tell Docker that this container communicates on port 3306. This is just documentation — it doesn't actually open the port.

`ENTRYPOINT ["/docker-entrypoint.sh"]` — When the container starts, run this script.

Flow: Start with Debian → Install software → Copy setup script → Make it executable → When container starts, run the script.

---

## Step 5 — Write the docker-compose.yml

`nano ~/inception/srcs/docker-compose.yml`

```yaml
services:

  mariadb:
    image: mariadb
    build: ./requirements/mariadb
    container_name: mariadb
    env_file: .env
    secrets:
      - db_password
      - db_root_password
    volumes:
      - mariadb_db:/var/lib/mysql
    networks:
      - inception
    restart: unless-stopped

  wordpress:
    image: wordpress
    build: ./requirements/wordpress
    container_name: wordpress
    env_file: .env
    secrets:
      - db_password
      - credentials
    volumes:
      - wordpress:/var/www/html
    networks:
      - inception
    depends_on:
      - mariadb
    restart: unless-stopped

  nginx:
    image: nginx
    build: ./requirements/nginx
    container_name: nginx
    ports:
      - "443:443"
    volumes:
      - wordpress:/var/www/html
    networks:
      - inception
    depends_on:
      - wordpress
    restart: unless-stopped

volumes:
  wordpress:
    driver: local
    driver_opts:
      type: none
      device: /home/${USER}/data/wordpress
      o: bind
  mariadb_db:
    driver: local
    driver_opts:
      type: none
      device: /home/${USER}/data/mariadb
      o: bind

networks:
  inception:
    driver: bridge

secrets:
  db_password:
    file: ../secrets/db_password.txt
  db_root_password:
    file: ../secrets/db_root_password.txt
  credentials:
    file: ../secrets/credentials.txt
```

**What each part does:**
- `services` — defines your 3 containers
- `build` — tells Docker where the Dockerfile is for each service
- `env_file` — loads variables from your .env file into the container
- `secrets` — passes sensitive files (passwords) securely into the container
- `volumes` — connects persistent storage to the container
- `networks` — puts all containers on the same private network so they can talk to each other
- `depends_on` — makes sure mariadb starts before wordpress, and wordpress before nginx
- `restart: unless-stopped` — automatically restarts the container if it crashes

---

## Step 6 — Configure the services

### NGINX config
`nano ~/inception/srcs/requirements/nginx/conf/nginx.conf`

```nginx
server {
    listen 443 ssl;
    server_name yourlogin.42.fr;

    ssl_certificate     /etc/ssl/certs/nginx.crt;
    ssl_certificate_key /etc/ssl/private/nginx.key;
    ssl_protocols       TLSv1.2 TLSv1.3;

    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass wordpress:9000;
    }
}
```

> Replace `yourlogin` with your 42 login.

### MariaDB setup script
`nano ~/inception/srcs/requirements/mariadb/tools/db_setup.sh`

```bash
#!/bin/bash
set -e

DB_PASS=$(cat /run/secrets/db_password)
ROOT_PASS=$(cat /run/secrets/db_root_password)

# Initialize database if first time
if [ ! -d /var/lib/mysql/mysql ]; then
    mysql_install_db --user=mysql --datadir=/var/lib/mysql
fi

# Start MariaDB temporarily to set up users
# --skip-grant-tables allows passwordless root access during setup
mysqld_safe --skip-networking --skip-grant-tables &
sleep 8

# Create database and users
mysql -u root <<EOF
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '${ROOT_PASS}';
CREATE DATABASE IF NOT EXISTS ${MYSQL_DATABASE};
CREATE USER IF NOT EXISTS '${MYSQL_USER}'@'%' IDENTIFIED BY '${DB_PASS}';
GRANT ALL PRIVILEGES ON ${MYSQL_DATABASE}.* TO '${MYSQL_USER}'@'%';
FLUSH PRIVILEGES;
EOF

# Shut down temp instance
mysqladmin -u root -p"${ROOT_PASS}" shutdown
sleep 3

# Start properly bound to all interfaces so WordPress can reach it over the network
exec mysqld_safe --bind-address=0.0.0.0
```

### WordPress setup script
`nano ~/inception/srcs/requirements/wordpress/tools/wordpress_setup.sh`

```bash
#!/bin/bash
set -e

DB_PASS=$(cat /run/secrets/db_password)
WP_ADMIN_PASS=$(cat /run/secrets/credentials)

# Wait for MariaDB to be actually ready
# Uses mysql auth check — more reliable than mysqladmin ping
# because it tests actual user authentication, not just whether the socket is open
until mysql -h mariadb -u "${MYSQL_USER}" -p"${DB_PASS}" -e "SELECT 1;" 2>/dev/null; do
    echo "Waiting for MariaDB..."
    sleep 3
done

if [ ! -f /var/www/html/wp-config.php ]; then
    wp core download --allow-root

    wp config create --allow-root \
        --dbname=${MYSQL_DATABASE} \
        --dbuser=${MYSQL_USER} \
        --dbpass=${DB_PASS} \
        --dbhost=mariadb:3306

    wp core install --allow-root \
        --url=https://${DOMAIN_NAME} \
        --title="${WP_TITLE}" \
        --admin_user=${WP_ADMIN_USER} \
        --admin_password=${WP_ADMIN_PASS} \
        --admin_email=${WP_ADMIN_EMAIL}

    wp user create --allow-root \
        ${WP_USER} ${WP_USER_EMAIL} \
        --role=author \
        --user_pass=${WP_USER_PASS}
fi

# Fix php-fpm to listen on TCP port 9000 instead of unix socket
# By default Debian's php-fpm listens on /run/php/php8.2-fpm.sock
# but NGINX connects to wordpress:9000 over the Docker network — they must match
sed -i 's|listen = /run/php/php8.2-fpm.sock|listen = 9000|' /etc/php/8.2/fpm/pool.d/www.conf

exec php-fpm8.2 -F
```

**What each script does:**
- NGINX config — listens on port 443, uses the SSL certificate, and forwards PHP requests to WordPress on port 9000
- MariaDB script — initializes the database, creates the user with the right password, then starts MariaDB bound to all interfaces so WordPress can reach it
- WordPress script — waits for MariaDB to be truly ready, downloads WordPress, connects it to the database, installs it with your admin user, creates a second regular user, fixes php-fpm to listen on TCP, then starts PHP-FPM

---

## Step 7 — Fill in the .env file

`nano ~/inception/srcs/.env`

```env
DOMAIN_NAME=yourlogin.42.fr
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
WP_TITLE=MyInception
WP_ADMIN_USER=yourlogin42
WP_ADMIN_EMAIL=yourlogin@student.42.fr
WP_USER=regularuser
WP_USER_EMAIL=user@student.42.fr
WP_USER_PASS=somepassword123
```

> `WP_ADMIN_USER` must NOT contain `admin` or `administrator` in any form — the subject forbids it.

A `.env` file stores environment variables as KEY=VALUE pairs. Instead of hardcoding values directly into your code or Docker files, you define them once here and reference them anywhere with `${KEY}`. Docker Compose loads it automatically.

---

## Step 8 — Fill in the secrets

```bash
echo "strongdbpassword" > ~/inception/secrets/db_password.txt
echo "strongrootpassword" > ~/inception/secrets/db_root_password.txt
echo "strongwpadminpass" > ~/inception/secrets/credentials.txt
```

**What are secrets?**
Docker secrets are a secure way to pass sensitive data (passwords, keys) to containers. Instead of putting passwords in `.env` (which can be exposed via `docker inspect`), secrets are stored as files and mounted inside the container at `/run/secrets/filename`. Your scripts read them with `cat /run/secrets/db_password`.

> **SECRETS ARE NOT ALLOWED IN YOUR GIT REPOSITORY.** Add `secrets/` to your `.gitignore`. They will be created manually on each machine during evaluation.
>**What you can do is also put the secret folder aside in the home and when the reviewer comes put the secret inside the cloned repository**

---

## Step 9 — Configure your domain

```bash
echo "127.0.0.1 yourlogin.42.fr" | sudo tee -a /etc/hosts
```
>**Do not forget use the command but change your login too!!!**
---

## Step 10 — Write the Makefile

`nano ~/inception/Makefile`

```makefile
NAME = inception

all:
	@mkdir -p /home/$(shell whoami)/data/wordpress
	@mkdir -p /home/$(shell whoami)/data/mariadb
	@docker compose -f srcs/docker-compose.yml --env-file srcs/.env up -d --build

down:
	@docker compose -f srcs/docker-compose.yml --env-file srcs/.env down

re: fclean all

clean:
	@docker compose -f srcs/docker-compose.yml --env-file srcs/.env down --rmi all --volumes

fclean: clean
	@sudo rm -rf /home/$(shell whoami)/data

.PHONY: all down re clean fclean
```

> The indentation on each command line must be a TAB not spaces.

---

## Step 11 — Build and run

```bash
cd ~/inception
make
```

Then check all containers are up:
```bash
docker ps
```

All three (`nginx`, `wordpress`, `mariadb`) should show status `Up`. Check WordPress finished installing:
```bash
docker logs wordpress
```

You should see at the end:
```
Success: WordPress installed successfully.
Success: Created user 2.
```

---

## Step 12 — Test the site

Open your VM's browser and go to:
```
https://yourlogin.42.fr
```

Accept the security warning about the self-signed SSL certificate (click Advanced → Accept the risk and continue).

Admin panel:
```
https://yourlogin.42.fr/wp-admin
```

Login with `WP_ADMIN_USER` and whatever password you put in `secrets/credentials.txt`.

---

## Useful commands

```bash
make            # build and start everything
make down       # stop all containers
make clean      # stop + remove images and volumes
make fclean     # full wipe including data on host
make re         # full wipe and rebuild

docker ps                    # see running containers
docker logs wordpress        # see wordpress logs
docker logs mariadb          # see mariadb logs
docker logs nginx            # see nginx logs
docker logs -f wordpress     # follow logs in real time
docker exec -it mariadb bash # open shell inside mariadb container
```

---

## Common mistakes and fixes

| Problem | Cause | Fix |
|---|---|---|
| `502 Bad Gateway` | php-fpm listening on unix socket, not TCP 9000 | The `sed -i` line in wordpress_setup.sh fixes this — make sure it's there |
| `wp: command not found` | WP-CLI not downloaded correctly | Use `curl -L` and download directly to `/usr/local/bin/wp` |
| `Waiting for MariaDB...` forever | Wrong password or MariaDB not bound to network | Make sure secrets files are not empty and db_setup.sh uses `--bind-address=0.0.0.0` |
| `Access denied for user 'root'` | Old MariaDB data from previous run with different password | Run `sudo rm -rf ~/data` then `make fclean` then `make` |
| `make: command not found` | make not installed | `sudo apt-get install -y make` |
| `docker: command not found` | Docker not installed | Follow Step 2 above |
