# Docker with WSL 2 and Ubuntu: Complete Practical Tutorial

**Environment:** Windows 10/11, WSL 2, Ubuntu, Linux containers  
**Level:** Beginner to intermediate  
**Updated:** October 2026

This tutorial shows you how to use Docker from an Ubuntu distribution running under Windows Subsystem for Linux (WSL 2). It starts with installation and finishes with a practical multi-container PHP, MySQL, and Adminer project.

By the end, you will know how to:

- Connect Docker Desktop to Ubuntu under WSL 2
- Alternatively, install Docker Engine directly inside Ubuntu
- Run, inspect, stop, and remove containers
- Build images using a `Dockerfile`
- Publish container ports to Windows
- Persist data with volumes
- share source code with bind mounts
- Connect containers through Docker networks
- Manage a multi-container project with Docker Compose
- Use Docker from Visual Studio Code
- Back up a containerized MySQL database
- Diagnose the most common WSL and Docker problems

## 1. Understand the architecture

There are two valid ways to use Docker with WSL Ubuntu.

| Setup | Where the Docker daemon runs | Best for |
|---|---|---|
| **A. Docker Desktop with WSL integration** | Docker Desktop's WSL 2 backend | Most Windows developers and beginners |
| **B. Docker Engine inside Ubuntu** | Directly inside your Ubuntu distribution | Linux-like administration, labs, or avoiding Docker Desktop |

### Recommended setup

For normal Windows development, use **Docker Desktop with WSL integration**. It provides:

- Straightforward installation and updates
- Docker Engine, Docker CLI, Buildx, and Compose
- Integration with Windows and WSL distributions
- A graphical interface for containers, images, volumes, and logs
- Easy access to container ports through `localhost`

### Important: choose only one Docker daemon

Do not install Docker Engine directly inside Ubuntu if you intend to use Docker Desktop's WSL integration. Docker explicitly recommends removing a Docker Engine or Docker CLI installation from the WSL distribution before enabling Docker Desktop integration, because parallel installations can conflict.

If you already installed Docker inside Ubuntu, first decide which setup you want. Do not blindly uninstall anything if it contains important containers or volumes; export or back up the data first.

## 2. Key Docker concepts

Before installing anything, understand these terms:

| Term | Meaning |
|---|---|
| **Image** | A read-only template containing an application and its dependencies |
| **Container** | A running or stopped instance created from an image |
| **Dockerfile** | Instructions for building an image |
| **Registry** | A service that stores images, such as Docker Hub |
| **Volume** | Docker-managed persistent storage |
| **Bind mount** | A host file or directory mounted inside a container |
| **Network** | A virtual network through which containers communicate |
| **Compose** | A tool and YAML format for managing multi-container applications |
| **Docker daemon** | The background service that creates and manages Docker objects |
| **Docker CLI** | The `docker` command used to communicate with the daemon |

An image is similar to a reusable blueprint. A container is a particular instance created from that blueprint.

Deleting a container does not delete its image. Data stored only in a container's writable layer is lost when that container is removed. Important data should live in a volume, bind mount, database backup, or external service.

## 3. Verify WSL and Ubuntu

Open **PowerShell** and run:

```powershell
wsl --version
wsl --status
wsl --list --verbose
```

You should see Ubuntu using WSL version 2:

```text
NAME      STATE      VERSION
Ubuntu    Running    2
```

If Ubuntu uses WSL 1, convert it:

```powershell
wsl --set-version Ubuntu 2
```

Set WSL 2 as the default for future distributions:

```powershell
wsl --set-default-version 2
```

Update WSL:

```powershell
wsl --update
```

Start Ubuntu:

```powershell
wsl -d Ubuntu
```

Inside Ubuntu, update its packages:

```bash
sudo apt update
sudo apt upgrade
```

Check the Ubuntu release:

```bash
cat /etc/os-release
```

## 4. Installation path A: Docker Desktop with WSL integration

This is the recommended path for most Windows development computers.

### 4.1 Check for an existing native installation

Inside Ubuntu, run:

```bash
command -v docker
dpkg -l | grep -E 'docker|containerd'
```

The presence of a `docker` command does not always mean a native daemon is installed; Docker Desktop also exposes its CLI inside an integrated WSL distribution. If you previously installed Docker Engine manually, back up its important data before removing or replacing it.

### 4.2 Install Docker Desktop on Windows

Download Docker Desktop from the [official Docker Desktop for Windows page](https://docs.docker.com/desktop/setup/install/windows-install/).

During installation:

1. Use the WSL 2 backend when offered.
2. Complete the installer.
3. Start Docker Desktop from the Windows Start menu.
4. Accept the terms when prompted.
5. Wait until Docker Desktop reports that the engine is running.

### 4.3 Enable the WSL 2 engine

In Docker Desktop:

1. Open **Settings**.
2. Select **General**.
3. Enable **Use the WSL 2 based engine** if that option is visible.
4. Select **Apply & restart**.

On supported installations, the WSL 2 engine may already be enabled and the setting may not be shown.

### 4.4 Enable Ubuntu integration

In Docker Desktop:

1. Open **Settings → Resources → WSL Integration**.
2. Enable integration with the default WSL distribution, or enable the switch beside **Ubuntu**.
3. Select **Apply & restart**.

If the WSL Integration page is missing, Docker Desktop may be using Windows containers. From Docker's taskbar menu, switch to **Linux containers**.

### 4.5 Verify Docker inside Ubuntu

Open Ubuntu and run:

```bash
docker version
docker info
docker compose version
```

`docker version` should display both **Client** and **Server** sections. If only the client appears, the CLI cannot reach the Docker daemon.

Run the test container:

```bash
docker run --rm hello-world
```

Docker downloads the `hello-world` image, creates a container, prints a confirmation, and automatically removes the stopped container because of `--rm`.

You do not normally use `sudo docker` with Docker Desktop's WSL integration.

After completing this section, continue at **Section 6**. Do not also install the native Engine from Section 5.

## 5. Installation path B: Docker Engine directly inside Ubuntu

Use this path only if you do not want to use Docker Desktop. Docker will run as a Linux service inside the Ubuntu WSL distribution.

### 5.1 Ensure systemd works

Current Ubuntu installations created by `wsl --install` normally use systemd. Check it:

```bash
systemctl status
```

If systemd is not active, edit the WSL configuration:

```bash
sudo nano /etc/wsl.conf
```

Add:

```ini
[boot]
systemd=true
```

Save and exit Nano:

- Press `Ctrl+O`
- Press `Enter`
- Press `Ctrl+X`

From PowerShell—not Ubuntu—restart WSL:

```powershell
wsl --shutdown
```

Open Ubuntu again and verify:

```bash
systemctl status
```

### 5.2 Remove conflicting Ubuntu packages

The official Docker packages may conflict with unofficial or distribution-provided packages. The following command removes only the named packages if they are installed:

```bash
sudo apt remove $(dpkg --get-selections \
  docker.io docker-compose docker-compose-v2 docker-doc docker-buildx \
  podman-docker containerd runc 2>/dev/null | cut -f1)
```

It is normal for `apt` to report that some or all packages are not installed.

Removing packages does not automatically remove existing Docker data under `/var/lib/docker`. Do not delete that directory unless you deliberately want to erase all native Docker images, containers, and volumes.

### 5.3 Add Docker's official apt repository

Install the prerequisites and Docker signing key:

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker's repository:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Refresh the package index:

```bash
sudo apt update
```

### 5.4 Install Docker Engine and Compose

```bash
sudo apt install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

Check the service:

```bash
sudo systemctl status docker
```

If it is not running:

```bash
sudo systemctl start docker
```

Test the installation:

```bash
sudo docker run --rm hello-world
```

### 5.5 Run Docker commands without `sudo`

Add your Linux user to the `docker` group:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Test it:

```bash
docker run --rm hello-world
```

> **Security warning:** Membership in the `docker` group effectively grants root-level control over the Linux environment. Do not add untrusted users to this group.

To start Docker automatically when the Ubuntu distribution starts:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### 5.6 Verify all components

```bash
docker version
docker info
docker compose version
docker buildx version
```

## 6. Store projects in the Linux filesystem

Create Docker projects inside your Ubuntu home directory:

```bash
mkdir -p ~/projects
cd ~/projects
```

Recommended location:

```text
/home/andreas/projects/my-project
```

Avoid using this for active Linux/Docker development:

```text
/mnt/c/Users/Andreas/projects/my-project
```

Projects under `/home/...` normally have much faster file access, dependency installation, builds, and file watching than projects under `/mnt/c/...`.

Open the current Linux directory in Windows File Explorer:

```bash
explorer.exe .
```

Open it in Visual Studio Code:

```bash
code .
```

## 7. Run your first useful container

Start an Nginx web server:

```bash
docker run --name my-nginx -d -p 8080:80 nginx:alpine
```

The options mean:

| Option | Meaning |
|---|---|
| `--name my-nginx` | Give the container a memorable name |
| `-d` | Run in detached/background mode |
| `-p 8080:80` | Map host port 8080 to container port 80 |
| `nginx:alpine` | Use the Alpine-based Nginx image |

Open this address in your Windows browser:

```text
http://localhost:8080
```

List running containers:

```bash
docker ps
```

List running and stopped containers:

```bash
docker ps -a
```

Read the logs:

```bash
docker logs my-nginx
```

Follow new log messages:

```bash
docker logs -f my-nginx
```

Press `Ctrl+C` to stop following the logs; this does not stop the container.

Open a shell inside the container:

```bash
docker exec -it my-nginx sh
```

Inside the container, try:

```sh
hostname
cat /etc/os-release
ls -la /usr/share/nginx/html
exit
```

Stop, start, and restart the container:

```bash
docker stop my-nginx
docker start my-nginx
docker restart my-nginx
```

Remove it:

```bash
docker stop my-nginx
docker rm my-nginx
```

Force-removing a running container is possible, but use it deliberately:

```bash
docker rm -f my-nginx
```

## 8. Work with images

Download an image without starting a container:

```bash
docker pull ubuntu:24.04
```

List local images:

```bash
docker image ls
```

Inspect image metadata:

```bash
docker image inspect ubuntu:24.04
```

Run a temporary Ubuntu shell:

```bash
docker run --rm -it ubuntu:24.04 bash
```

Inside the container:

```bash
cat /etc/os-release
exit
```

Remove an unused image:

```bash
docker image rm ubuntu:24.04
```

Docker cannot normally remove an image while existing containers still depend on it.

## 9. Build your first image

Create a project:

```bash
mkdir -p ~/projects/docker-web-demo
cd ~/projects/docker-web-demo
```

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Docker on WSL</title>
</head>
<body>
    <h1>Hello from Docker on WSL 2</h1>
    <p>This page is served by an Nginx container.</p>
</body>
</html>
```

Create a file named `Dockerfile` with no extension:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Create `.dockerignore`:

```text
.git
.gitignore
README.md
```

Build the image:

```bash
docker build -t wsl-web-demo:1.0 .
```

The final period means that the current directory is the build context.

Run it:

```bash
docker run --name wsl-web-demo -d -p 8080:80 wsl-web-demo:1.0
```

Visit:

```text
http://localhost:8080
```

Inspect the container:

```bash
docker inspect wsl-web-demo
```

View its resource use:

```bash
docker stats wsl-web-demo
```

Stop and remove it:

```bash
docker rm -f wsl-web-demo
```

## 10. Bind mounts and volumes

Docker has two main storage mechanisms.

### Bind mounts

A bind mount maps a known file or directory from Ubuntu into a container. It is useful for source code and configuration during development.

From `~/projects/docker-web-demo`, run:

```bash
docker run --name live-nginx -d \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html,readonly \
  nginx:alpine
```

Edit `index.html` in Ubuntu, refresh `http://localhost:8080`, and the change should appear without rebuilding the image.

The `readonly` option prevents the container from modifying the mounted project directory.

Remove the container:

```bash
docker rm -f live-nginx
```

### Named volumes

A named volume is stored and managed by Docker. Volumes are normally the best choice for database data.

Create a volume:

```bash
docker volume create demo-data
```

List volumes:

```bash
docker volume ls
```

Inspect it:

```bash
docker volume inspect demo-data
```

Write a file into it using a temporary container:

```bash
docker run --rm \
  --mount source=demo-data,target=/data \
  alpine sh -c 'echo "Persistent Docker data" > /data/message.txt'
```

Read it using another temporary container:

```bash
docker run --rm \
  --mount source=demo-data,target=/data,readonly \
  alpine cat /data/message.txt
```

The data remains even though both containers were removed.

Remove the volume only when its data is no longer needed:

```bash
docker volume rm demo-data
```

## 11. Docker networking

Create a user-defined network:

```bash
docker network create demo-network
```

Start Nginx on that network without publishing a host port:

```bash
docker run --name internal-web -d \
  --network demo-network \
  nginx:alpine
```

Use a temporary Curl container on the same network:

```bash
docker run --rm \
  --network demo-network \
  curlimages/curl http://internal-web
```

Docker's internal DNS resolves the container name `internal-web` to its container address.

Inspect the network:

```bash
docker network inspect demo-network
```

Clean up:

```bash
docker rm -f internal-web
docker network rm demo-network
```

Containers in the same Compose project receive a project network automatically, so you rarely need to create networks manually for a normal Compose application.

## 12. Docker Compose fundamentals

Docker Compose describes a multi-container application in a YAML file, normally `compose.yaml`.

The modern command is:

```bash
docker compose
```

The older standalone form:

```bash
docker-compose
```

is not used in this tutorial.

Common Compose commands:

```bash
docker compose up
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose exec SERVICE COMMAND
```

## 13. Practical project: PHP, MySQL, and Adminer

This local development stack contains:

- A PHP 8.4 Apache application
- A MySQL 8.4 database
- Adminer for browser-based database administration
- A bind mount for live PHP source-code changes
- A named volume for persistent MySQL data
- A health check so the application waits for MySQL

### 13.1 Create the project

```bash
mkdir -p ~/projects/php-docker-demo/src
cd ~/projects/php-docker-demo
```

The final structure will be:

```text
php-docker-demo/
├── .env
├── .gitignore
├── compose.yaml
├── Dockerfile
└── src/
    └── index.php
```

### 13.2 Create the PHP Dockerfile

Create `Dockerfile`:

```dockerfile
FROM php:8.4-apache

RUN docker-php-ext-install pdo pdo_mysql

WORKDIR /var/www/html
```

This extends the official PHP Apache image and installs the PHP PDO MySQL driver.

### 13.3 Create the environment file

Create `.env`:

```dotenv
MYSQL_DATABASE=demo
MYSQL_USER=demo_user
MYSQL_PASSWORD=replace-with-a-strong-user-password
MYSQL_ROOT_PASSWORD=replace-with-a-strong-root-password
```

Use different passwords from those shown here.

Create `.gitignore` so secrets and local data are not committed:

```gitignore
.env
```

For a real project, commit a safe `.env.example` file containing variable names but no real secrets.

### 13.4 Create the Compose file

Create `compose.yaml`:

```yaml
services:
  app:
    build:
      context: .
    ports:
      - "127.0.0.1:8080:80"
    environment:
      DB_HOST: db
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./src:/var/www/html
    depends_on:
      db:
        condition: service_healthy

  db:
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test:
        - CMD-SHELL
        - mysqladmin ping -h localhost -u root -p"$${MYSQL_ROOT_PASSWORD}"
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 20s

  adminer:
    image: adminer:latest
    ports:
      - "127.0.0.1:8081:8080"
    depends_on:
      db:
        condition: service_healthy

volumes:
  db_data:
```

The database has no published host port. Only the application and Adminer can access it over the private Compose network. This is safer and avoids conflicts with a MySQL server already using Windows or WSL port 3306.

### 13.5 Create the PHP application

Create `src/index.php`:

```php
<?php
declare(strict_types=1);

$dsn = sprintf(
    'mysql:host=%s;dbname=%s;charset=utf8mb4',
    getenv('DB_HOST'),
    getenv('DB_NAME')
);

try {
    $pdo = new PDO(
        $dsn,
        getenv('DB_USER'),
        getenv('DB_PASSWORD'),
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );

    $pdo->exec(
        'CREATE TABLE IF NOT EXISTS visits (
            id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            visited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )'
    );

    $pdo->exec('INSERT INTO visits () VALUES ()');
    $visits = (int) $pdo->query('SELECT COUNT(*) FROM visits')->fetchColumn();
} catch (Throwable $exception) {
    http_response_code(500);
    $error = $exception->getMessage();
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Docker Compose Demo</title>
</head>
<body>
    <h1>Docker Compose on WSL 2</h1>

    <?php if (isset($error)): ?>
        <p>Database error: <?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
    <?php else: ?>
        <p>The PHP application successfully connected to MySQL.</p>
        <p>Recorded visits: <?= $visits ?></p>
    <?php endif; ?>
</body>
</html>
```

### 13.6 Validate and start the stack

Render and validate the Compose configuration:

```bash
docker compose config
```

Build the PHP image and start all services:

```bash
docker compose up -d --build
```

View their state:

```bash
docker compose ps
```

The database may take several seconds to become healthy during its first startup.

Open the PHP application:

```text
http://localhost:8080
```

Refresh the page several times. The visit counter should increase.

Open Adminer:

```text
http://localhost:8081
```

Use:

| Field | Value |
|---|---|
| System | MySQL |
| Server | `db` |
| Username | Value of `MYSQL_USER` |
| Password | Value of `MYSQL_PASSWORD` |
| Database | Value of `MYSQL_DATABASE` |

Do not use `localhost` as the database server in Adminer. Inside the Compose network, the MySQL service is named `db`.

### 13.7 Inspect and debug the stack

View logs from every service:

```bash
docker compose logs
```

Follow logs continuously:

```bash
docker compose logs -f
```

Follow only the application logs:

```bash
docker compose logs -f app
```

Open a shell in the PHP container:

```bash
docker compose exec app bash
```

Check PHP and its extensions:

```bash
php -v
php -m | grep -E 'PDO|pdo_mysql'
exit
```

Open the MySQL client:

```bash
docker compose exec db mysql \
  -u demo_user -p demo
```

Enter the password from `.env`, then try:

```sql
SHOW TABLES;
SELECT * FROM visits ORDER BY id DESC LIMIT 10;
EXIT;
```

### 13.8 Stop or remove the stack

Stop the containers without removing them:

```bash
docker compose stop
```

Start them again:

```bash
docker compose start
```

Stop and remove the containers and project network:

```bash
docker compose down
```

The MySQL data remains because `db_data` is a named volume.

To also delete the database volume:

```bash
docker compose down --volumes
```

> **Destructive warning:** `--volumes` permanently removes the Compose-managed database volume and its data. Back up anything important first.

## 14. Back up and restore the MySQL database

Run these commands from the directory containing `compose.yaml`.

Create a backup directory:

```bash
mkdir -p backups
```

Export the database using the credentials already present inside the database container:

```bash
docker compose exec -T db sh -c \
  'mysqldump -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE"' \
  > backups/demo.sql
```

Check that the file contains SQL:

```bash
ls -lh backups/demo.sql
head -n 10 backups/demo.sql
```

Restore it:

```bash
docker compose exec -T db sh -c \
  'mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE"' \
  < backups/demo.sql
```

Treat database backups as sensitive because they can contain customer data, password hashes, API keys, or personal information.

## 15. Daily Docker workflow

For a typical Compose project:

```bash
cd ~/projects/php-docker-demo
docker compose up -d
docker compose ps
docker compose logs -f app
```

After changing a Dockerfile or dependency definition:

```bash
docker compose up -d --build
```

After changing only a bind-mounted source file, a rebuild is normally unnecessary.

When finished:

```bash
docker compose down
```

Useful container commands:

```bash
docker ps
docker ps -a
docker logs CONTAINER
docker exec -it CONTAINER sh
docker inspect CONTAINER
docker stats
```

Useful image and storage commands:

```bash
docker image ls
docker volume ls
docker network ls
docker system df
```

## 16. Use Docker with Visual Studio Code

Install these Windows VS Code extensions:

- **WSL**
- **Docker**
- **Dev Containers** if you want the editor itself to run inside a development container

From Ubuntu:

```bash
cd ~/projects/php-docker-demo
code .
```

Check the lower-left corner of VS Code. It should indicate that the folder is open through WSL Ubuntu.

Open the integrated terminal and verify:

```bash
pwd
docker compose ps
```

The path should begin with `/home/...`, not `C:\...` or `/mnt/c/...`.

### Optional: Dev Containers

A Dev Container lets VS Code use a container as the complete development environment. The configuration lives in `.devcontainer/devcontainer.json` and can be committed to Git so that team members receive consistent tools and extensions.

In VS Code:

1. Press `F1`.
2. Select **Dev Containers: Add Dev Container Configuration Files**.
3. Select a suitable base environment.
4. Select **Dev Containers: Reopen in Container**.

This is separate from merely running your application containers with Compose.

## 17. Ports and access from Windows

This Compose mapping:

```yaml
ports:
  - "127.0.0.1:8080:80"
```

means:

- `127.0.0.1`: listen only on the local host
- `8080`: port used by Windows and WSL
- `80`: port used inside the container

You access it from Windows at:

```text
http://localhost:8080
```

This shorter mapping:

```yaml
ports:
  - "8080:80"
```

normally binds on all host interfaces. That may make the service reachable from other devices, depending on the Docker/WSL networking mode and Windows Firewall rules.

For local development, binding to `127.0.0.1` reduces unnecessary exposure.

Avoid publishing database ports unless a host application genuinely needs them. Containers in the same Compose project can connect using the service name and internal port, such as `db:3306`.

## 18. Resource management

Docker Desktop uses the WSL 2 virtualized environment. Configure its limits through Docker Desktop settings or WSL settings, depending on your current Docker Desktop version.

A global WSL configuration file can be placed at:

```text
C:\Users\<WindowsUser>\.wslconfig
```

Example:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Apply changes from PowerShell:

```powershell
wsl --shutdown
```

Do not allocate all system RAM or CPU cores to WSL. Windows and other applications also need resources.

Inspect live container use:

```bash
docker stats
```

Add Compose resource constraints only after measuring actual requirements.

## 19. Disk use and cleanup

See Docker's disk usage:

```bash
docker system df
```

Remove stopped containers:

```bash
docker container prune
```

Remove unused images that are not referenced by containers:

```bash
docker image prune
```

Remove unused networks:

```bash
docker network prune
```

Remove unused volumes:

```bash
docker volume prune
```

> **Warning:** An unused volume may still contain valuable database or application data. Inspect and back up volumes before pruning them.

The broad cleanup command is:

```bash
docker system prune
```

Read the displayed list carefully before confirming. Do not routinely add `--volumes` unless you intend to remove unused persistent data.

## 20. Security essentials

Docker is a development and deployment tool, not an automatic security boundary. Follow these rules:

1. Do not expose the Docker daemon's TCP socket to the network without strong authentication and a clear need.
2. Treat access to `/var/run/docker.sock` as root-equivalent access.
3. Do not mount the Docker socket into untrusted containers.
4. Avoid `--privileged` unless you understand exactly why it is required.
5. Bind development ports to `127.0.0.1` when only the local computer needs access.
6. Do not store passwords or API keys in Dockerfiles or images.
7. Keep `.env` files out of Git.
8. Prefer official or trusted images and review their tags, update policy, and Dockerfiles.
9. Use specific version tags for important environments instead of relying entirely on `latest`.
10. Keep Docker Desktop, WSL, Ubuntu packages, base images, and application dependencies updated.
11. Run applications as a non-root container user where practical.
12. Do not publish MySQL, PostgreSQL, Redis, or administration tools to external interfaces without access controls.

The PHP/MySQL example in this tutorial is a local learning environment, not a production deployment blueprint.

## 21. Troubleshooting

### `docker: command not found`

For Docker Desktop:

1. Confirm Docker Desktop is installed and running.
2. Open **Settings → Resources → WSL Integration**.
3. Enable Ubuntu.
4. Select **Apply & restart**.
5. Close and reopen the Ubuntu terminal.

Check:

```bash
which docker
docker version
```

For the native Engine, verify that `docker-ce-cli` was installed.

### `Cannot connect to the Docker daemon`

With Docker Desktop:

- Start Docker Desktop.
- Wait for the engine to finish starting.
- Check Ubuntu integration.
- From PowerShell, try:

```powershell
wsl --shutdown
```

Then restart Docker Desktop and Ubuntu.

With native Docker Engine:

```bash
sudo systemctl status docker
sudo systemctl start docker
journalctl -u docker --no-pager -n 100
```

### Permission denied on `/var/run/docker.sock`

This normally applies to the native Engine setup:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
docker run --rm hello-world
```

Remember that the `docker` group grants root-equivalent capabilities.

### Port is already allocated

If port 8080 is occupied, find the process or container:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
sudo ss -ltnp | grep ':8080'
```

From PowerShell:

```powershell
Get-NetTCPConnection -LocalPort 8080
```

Either stop the conflicting service or change the host side of the mapping:

```yaml
ports:
  - "127.0.0.1:8082:80"
```

Then use `http://localhost:8082`.

### Container name already exists

List the existing container:

```bash
docker ps -a --filter name=my-nginx
```

Remove it if no longer needed:

```bash
docker rm my-nginx
```

Or choose a different `--name`.

### Compose variables are blank or invalid

Make sure `.env` is in the same directory as `compose.yaml`, then inspect the rendered configuration:

```bash
docker compose config
```

Do not share the output publicly if it contains resolved secrets.

### A service exits immediately

List all services and read its logs:

```bash
docker compose ps -a
docker compose logs SERVICE_NAME
```

Inspect its exit code:

```bash
docker inspect CONTAINER_NAME \
  --format '{{.State.ExitCode}} {{.State.Error}}'
```

### Changes to a Dockerfile have no effect

Rebuild the image:

```bash
docker compose up -d --build
```

If the build cache is hiding a diagnostic change:

```bash
docker compose build --no-cache
docker compose up -d
```

Do not routinely disable the cache; it makes builds slower.

### Docker builds or file watching are slow

Check the project path:

```bash
pwd
```

If it begins with `/mnt/c`, move or clone the project under:

```text
/home/<your-linux-user>/projects/
```

Then open it using `code .` from Ubuntu.

### MySQL remains unhealthy

Read its logs:

```bash
docker compose logs db
```

Common causes include:

- An invalid environment variable
- A damaged or incompatible old volume
- Insufficient disk space
- Starting with one set of credentials and later changing `.env`

MySQL initialization environment variables are applied only when the data directory is first created. Changing `.env` does not automatically rewrite accounts inside an existing database volume.

For a disposable learning database only, you can recreate it:

```bash
docker compose down --volumes
docker compose up -d --build
```

This permanently deletes the existing Compose database volume.

### Windows browser cannot reach the container

Check:

```bash
docker compose ps
docker compose logs app
curl -I http://localhost:8080
```

Confirm that the port appears in `docker compose ps`. Also check Windows Firewall, VPN software, endpoint security, and whether another application owns the port.

## 22. Docker command cheat sheet

| Task | Command |
|---|---|
| Test Docker | `docker run --rm hello-world` |
| List running containers | `docker ps` |
| List all containers | `docker ps -a` |
| List images | `docker image ls` |
| Pull an image | `docker pull IMAGE:TAG` |
| Run in background | `docker run -d IMAGE` |
| Publish a port | `docker run -p 8080:80 IMAGE` |
| Read logs | `docker logs CONTAINER` |
| Follow logs | `docker logs -f CONTAINER` |
| Open a shell | `docker exec -it CONTAINER sh` |
| Stop a container | `docker stop CONTAINER` |
| Remove a container | `docker rm CONTAINER` |
| Build an image | `docker build -t NAME:TAG .` |
| List volumes | `docker volume ls` |
| List networks | `docker network ls` |
| Start Compose project | `docker compose up -d` |
| Build and start | `docker compose up -d --build` |
| Show Compose services | `docker compose ps` |
| Follow Compose logs | `docker compose logs -f` |
| Stop and remove project | `docker compose down` |
| Show disk usage | `docker system df` |

## 23. Recommended next exercises

After completing the tutorial, try these projects:

1. Replace the PHP demo with a Yii2 application.
2. Add Nginx as a reverse proxy in front of PHP-FPM.
3. Add Redis and connect to it using the Compose service name.
4. Add a MySQL migration command to the application workflow.
5. Create separate development and production Compose override files.
6. Add a health check to the application service.
7. Create a Dev Container configuration for VS Code.
8. Push your own image to a private container registry.
9. Add automated image building with GitHub Actions.
10. Deploy the Compose application to a disposable Linux virtual machine after replacing development settings and secrets.

## 24. Essential rules to remember

- Use either Docker Desktop integration or a native Docker Engine—not both.
- Keep Linux/Docker projects under `/home/...`, not `/mnt/c/...`.
- Use `docker compose`, not the obsolete `docker-compose` workflow.
- Use bind mounts for editable source code and named volumes for database data.
- Container names and Compose service names provide internal DNS resolution.
- Publishing a port makes a container service reachable from the host; internal container ports alone do not.
- A container is disposable; important state must live elsewhere.
- `docker compose down` preserves named volumes, while `docker compose down --volumes` deletes them.
- Docker group and Docker socket access are effectively root-level access.
- Never assume a local development Compose file is production-ready.

## Official references

- [Docker Desktop WSL 2 backend](https://docs.docker.com/desktop/features/wsl/)
- [Develop with Docker Desktop and WSL 2](https://docs.docker.com/desktop/features/wsl/use-wsl/)
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
- [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Docker port publishing](https://docs.docker.com/engine/network/port-publishing/)
- [Microsoft WSL filesystem guidance](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- [Microsoft systemd on WSL](https://learn.microsoft.com/en-us/windows/wsl/systemd)
