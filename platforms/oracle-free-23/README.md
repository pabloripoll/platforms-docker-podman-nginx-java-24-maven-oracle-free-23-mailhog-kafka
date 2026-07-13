<div id="top-header" style="with:100%;height:auto;text-align:right;"></div>

# ORACLE FREE 23

- [./back](../../README.md)
- [Features](#features)
- [Configuration](#configuration)
- [Management](#management)
<br>

## <a id="features"></a>Features

![Oracle](https://custom-icon-badges.demolab.com/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=fff)
![SQL](https://img.shields.io/badge/SQL-003B57?style=for-the-badge&labelColor=black&logo=sqlite&logoColor=white)

Content:
- Oracle Linux Server 8.10
- Oracle Free 23 *(Requires 1.33G Disk and 2G RAM)*
<br><br>

Sources:
- https://hub.docker.com/r/gvenzl/oracle-free
<br><br>

## <a id="configuration"></a>Service Configuration

There is a dedicated GNU Make file and the main Docker directory with the required scripts to build the required platform configuration.

This service platform is designed to be managed by a parent `./../../.env` but, as it feeds `./docker/.env` through GNU Make, it can be edited manually.

If no GNU Make is in used, create a copy from the example `./docker/.env.example` into `./docker/.env`.

The service container can be easily manage by the `./docker/docker-compose.yml`.

Require environment variables at `./docker/.env` *(all are customizable)*:
```bash
COMPOSE_PROJECT_LEAD="myproj"                                   # <- lead abbreviation or acronym as part of related containers naming rule ------> #
COMPOSE_PROJECT_CNET="mp-dev"                                   # <- useful for networking to connect between containers -------------------------> #
COMPOSE_PROJECT_IMGK="gvenzl/oracle-free:23-slim-faststart"     # <- real main image keys to manage automations for sharing resources ------------> #
COMPOSE_PROJECT_NAME="mp-oracle-dev"                            # <- container name - it is important to set the environment in this variable ----> #
COMPOSE_PROJECT_HOST=127.0.0.1                                  # <- machine hostname referrer - not necessary for this project ------------------> #
COMPOSE_PROJECT_PORT=7710                                       # <- local machine port opened for container service -----------------------------> #
COMPOSE_PROJECT_CPUS=2.00                                       # <- container's maximum CPUs usage - leave it empty for full usage --------------> #
COMPOSE_PROJECT_MEM=512M                                        # <- container's maximum RAM usage to apply by docker-compose --------------------> #
COMPOSE_PROJECT_SWAP=1G                                         # <- container's RAM swap space in storage executed by automation command --------> #
ORACLE_PLUGGABLE_DB_NAME=dev_local                              # <- database pluggable name -----------------------------------------------------> #
ORACLE_DATABASE_PASSWORD="dev-password"                         # <- database password -----------------------------------------------------------> #
```
<br>

### Containers Access Modes

- Set the required environment values in `./docker/.env` from `./docker/.env.example` if no GNU Make will be applied.
- Set the required configuration files by coping and updating them depending on your project requirements.
- Container availability by building the container with `docker-composer.yml` in separated configuration layers
    - Stand-alone
        - The container is intended to be published directly and accessed from the host network, typically via `0.0.0.0:<port>`. It does not require a shared Docker network. It is a common setting for local development.
    - Inside a Custom Network
        - The container is attached to a custom Docker network and is intended to be accessed through a reverse proxy or other containers on the same network.
        - This network setting is useful for isolating services while still allowing container-to-container communication.
        - It is a strongly recommended setting for remote deployment to avoid exposing the localhost port in used and protect by firewall.
        - <b>Connect from one container to another inside the custom network, by container name and its own exposed port</b>.
    - Host-Gateway
        - The container can reach services running on the host machine using the Docker host gateway mapping. This is useful when the container must access local services on the VPS/host, while public access is still handled through a reverse proxy. It is a recommended setting for remote deployment too.
    - Public exposure is controlled by the `ports` mapping.
    - `0.0.0.0:<port>` means externally accessible.
    - `127.0.0.1:<port>` means local-only access on the host and requires a reverse proxy, e.g. NGINX.
    - Docker network attachment controls container-to-container communication.
    - Host-gateway controls container-to-host communication.

build: ## builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
	cd docker && $(DOCKER_COMPOSE) -f docker-compose.yml -f docker-compose.public.yml up --build -d

network: ## starts up into an existing custom network for container-to-container communication, runnning in detached mode
	cd docker && $(DOCKER_COMPOSE) -f docker-compose.yml -f docker-compose.network.yml up -d
<br><br>

## <a id="management"></a>Service Management

To manage the container, run the GNU Make recipes
```bash
$ make help
Usage: $ make [target]
Targets:
$ make help                           shows this Makefile help message
$ make port-check                     shows this project port availability on local machine
$ make env                            checks if docker .env file exists
$ make env-set                        sets docker .env file
$ make info                           shows container information
$ make ssh                            enters the container shell
$ make up                             starts and recreates containers if compose config or image changed, runnning in detached mode
$ make build                          builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
$ make network                        starts up into an existing custom network for container-to-container communication, runnning in detached mode
$ make start                          starts the container and put on running from latest configuration
$ make stop                           stops the running container but data will not be destroyed
$ make restart                        restarts the running container
$ make clear                          removes container from Docker running containers
$ make destroy                        delete container image from Docker cache
$ make dev                            sets a development enviroment
$ make test-up                        creates a side database for tests
$ make test-down                      drops the side database for tests
$ make sql-install                    imports [file-name].sql file into database container as an initial project database located in /[dir-path]/
$ make sql-install-clear              clears up the [file-name].sql file as an initial project database located in /[dir-path]/
$ make sql-replace                    replaces the database container with the located in /[dir-path]/[file-name].sql file
$ make sql-backup                     exports into [file-name].sql file the project database backup into /[dir-path]/
$ make sql-backup-clear               clears up the project database backup [file-name].sql file located in /[dir-path]/
$ make sql-drop                       drops the current database and creates an empty new one inside the container
$ make remote-host-copy               copies referenced remote database by tcp connection into /[dir-path]/[file-name].sql for local development
$ make remote-file-copy               copies referenced remote database file into /[dir-path]/[file-name].sql for local development
```
<br>

See database version
```bash
$ make db-ssh

> cat /etc/os-release # see linux alpine version

> sqlplus -V # see oracle version

> sqlplus -S / as sysdba @- <<<'SELECT banner FROM v$$version;'
```
<br><br>


## Contributing

Contributions are very welcome! Please open issues or submit PRs for improvements, new features, or bug fixes.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/YourFeature`)
3. Commit your changes (`git commit -am 'feat: Add new feature'`)
4. Push to the branch (`git push origin feature/YourFeature`)
5. Create a new Pull Request
<br><br>

## License

This project is open-sourced under the [MIT license](LICENSE).

<!-- FOOTER -->
<br>

---

<br>

- [GO TOP ⮙](#top-header)

<div style="with:100%;height:auto;text-align:right;">
    <img src="./../../resources/docs/images/pr-banner-long.png">
</div>