<div id="top-header" style="with:100%;height:auto;text-align:right;">
    <img src="./../../resources/docs/images/pr-banner-long.png">
</div>

# KAFKA 3.9

- [./main](../../README.md)
- [Features](#features)
- [Configuration](#configuration)
- [Management](#management)
<br>

## <a id="features"></a>Features

![Alpine Linux](https://shields.io/badge/Alpine_Linux-%230D597F.svg?style=for-the-badge&logo=alpine-linux&logoColor=white)
![Apache Kafka](https://shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white)

Content:
- Linux Alpine 3.23
- Apache Kafka
    - Docker Compose with Kafka - KRaft Mode (No Zookeeper — lightest setup)
    - Bitnami's Kafka image (or apache/kafka since Kafka 3.3+ supports KRaft mode, eliminating the need for Zookeeper entirely)
    - Using the official apache/kafka image in KRaft mode removes the Zookeeper dependency, saving significant memory and CPU.
- Kafka UI by Provectus
    - Kafka does NOT come with a built-in UI as it is purely CLI/API-based by default.
    - Here are the most popular choices, ranked by ease of use + lightweight:
        - **Kafka UI** (by Provectus): General purpose, most popular.
        - **Redpanda Console**: Clean UI - easy setup.
        - **AKHQ**:	JVM-based - Advanced management.
        - **Confluent Control Center**: By suscription - Enterprise only.
<br><br>

Sources:
- https://kafka.apache.org/
- https://github.com/provectus/kafka-ui
<br><br>

## Kafka Ports

In Apache Kafka, port 9092 is the default port used for plaintext (non-TLS) client communication, while port 9093 is commonly designated for SSL/TLS encrypted communication or Control-Plane traffic.

For comprehensive documentation on configuring your listeners and protocols, review the official Apache [Kafka Listener Configuration](https://kafka.apache.org/39/security/listener-configuration/) guide.

### Container Port 9092: Plaintext Client Traffic

- <b>Primary Role</b>: The standard, out-of-the-box port for Kafka clients (producers and consumers) to connect to the broker, publish data, and read streams.

- <b>Data Transmission</b>: Data sent across this port is in cleartext. It does not provide encryption or secure authentication, making it suitable for development environments or trusted, isolated internal networks.

- <b>Usage</b>: Typically configured as the `PLAINTEXT://` listener in your `server.properties` file.

### Container Port 9093: SSL/TLS or Control-Plane Traffic

- <b>Encryption/Security</b>: In standard client-broker setups, 9093 is conventionally configured as the secure listener for <b>SSL/TLS encryption</b> (often defined as `SSL://`). This ensures that data in transit across the network is securely encrypted.

- <b>Internal Control Plane</b>: In modern Kafka architectures utilizing KRaft (Kafka Raft) without ZooKeeper, <b>port 9093 is often designated for Control-Plane traffic</b>. This handles communication between the KRaft controllers (which manage cluster metadata) and the brokers.

### Network & Port Configuration

This container uses fixed internal ports for its services, but you can freely customize the local (host) ports on your machine to avoid conflicts or hiding ports.

How Port Mapping Works? The containerized applications always listen on their standard, hardcoded internal ports. When starting the container, you map your preferred local ports to these internal destination ports (`-p <local_port>:<container_port>`).

#### Connecting to Kafka from your Local Machine

When writing your local application code or configuring your producers/consumers, you must target your custom local ports, not the container ports:
```sh
# Use your custom local port in your application code:
bootstrap.servers=localhost:[your-custom-port]
```
<br><br>

## <a id="configuration"></a>Service Configuration

There is a `./Makefile` to manege Docker container/s from outside `./docker` directory. Inside `./docker` there are the required scripts to build the platform service.

This service platform is also developed to be managed by a parent `Makefile` by its own `./../../.env` that feeds this `./docker/.env` and others for the infrastructure platforms. Repository directories structure overview:
```sh
.
├── platforms                   # infrastructure platforms
│   │
│   ├── kafka-3.9               # this platform
│   │   ├── docker
│   │   │   ├── .env            # this platform environment
│   │   │   ├── docker-compose.yml
│   │   │   └── ...etc
│   .   └── Makefile            # this platform automation
│
├── .env                        # infrastructure platforms environments
├── Makefile                    # infrastructure platforms automations
.
```

If no GNU Make is in used, create a copy from the example `./docker/.env.example` into `./docker/.env`.

Required environment variables from `./docker/.env.example` for Docker Compose:
```bash
COMPOSE_LEAD=myproj                                 # <- lead abbreviation or acronym as part of related containers naming rule -------------------------> #
COMPOSE_CNET=mp-dev                                 # <- real main image keys to manage automations for sharing resources -------------------------------> #
COMPOSE_IMGK=apache/kafka:3.9.2                     # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_HOST=127.0.0.1                              # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_PORT_PTEXT=9092                             # <- local machine port opened for container service - plaintext client traffic ---------------------> #
COMPOSE_PORT_PCTRL=9093                             # <- local machine port opened for container service - ssl/tls or control-plane traffic -------------> #
COMPOSE_DATA="./data"                               # <- broker platform data storage path in local disk ------------------------------------------------> #
COMPOSE_LOGS="./logs"                               # <- broker platform logs storage path in local disk ------------------------------------------------> #
COMPOSE_NAME=mp-kafka-dev                           # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_CPUS=2.00                                   # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_MEMO=256M                                   # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_SWAP=512M                                   # <- container's RAM swap space in storage executed by automation command ---------------------------> #
COMPOSE_CLIENT_IMGK=provectuslabs/kafka-ui:latest   # <- container property for the database client for pulling and caching image into docker -----------> #
COMPOSE_CLIENT_CPUS=1.00                            # <- client's container maximum CPUs usage to apply by docker-compose - empty for full usage --------> #
COMPOSE_CLIENT_MEMO=128M                            # <- client's container maximum RAM usage to apply by docker-compose --------------------------------> #
COMPOSE_CLIENT_SWAP=256M                            # <- client's container RAM swap space in storage executed by automation command --------------------> #
COMPOSE_CLIENT_PORT=7712                            # <- local machine port opened for client's container service client --------------------------------> #
COMPOSE_CLIENT_USERNAME=admin.user                  # <- database client first user account -------------------------------------------------------------> #
COMPOSE_CLIENT_PASSWORD="admin-pass"                # <- database client first user password ------------------------------------------------------------> #
COMPOSE_IGNORE_ORPHANS=true                         # <- ignore docker warnings messages when client is in used -----------------------------------------> #
```

* **Ignore Orphans**:<br>
This Mongo DB platform has been developed for optionally using it with a database client like Mongo Express. By avoiding the usage of `--remove-orphans` flag while building the container, it prevents to remove Mongo DB container but terminal will show a warning message. To prevent that known message, the `.env` file has a constant value set to true `COMPOSE_IGNORE_ORPHANS=true` by default if using `./Makefile` for automation

### Containers Access Modes
COMPOSE_LEAD="myproj"                               # <- lead abbreviation or acronym as part of related containers naming rule -------------------------> #
COMPOSE_CNET="mp-dev"                               # <- real main image keys to COMPOSE_LEAD="myproj"                               # <- lead abbreviation or acronym as part of related containers naming rule -------------------------> #
COMPOSE_CNET="mp-dev"                               # <- real main image keys to manage automations for sharing resources -------------------------------> #
COMPOSE_IMGK="apache/kafka:3.9.2"                   # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_HOST=127.0.0.1                              # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_PORT_PTEXT=9092                             # <- local machine port opened for container service - plaintext client traffic ---------------------> #
COMPOSE_PORT_PCTRL=9093                             # <- local machine port opened for container service - ssl/tls or control-plane traffic -------------> #
COMPOSE_PATH="../data"                              # <- platform broker data storage in local ----------------------------------------------------------> #
COMPOSE_NAME="mp-kafka-dev"                         # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_CPUS=2.00                                   # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_MEM=256M                                    # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_SWAP=512M                                   # <- container's RAM swap space in storage executed by automation command ---------------------------> #
COMPOSE_CLIENT_IMGK="provectuslabs/kafka-ui:latest" # <- container property for the database client for pulling and caching image into docker -----------> #
COMPOSE_CLIENT_CPUS=1.00                            # <- client's container maximum CPUs usage to apply by docker-compose - empty for full usage --------> #
COMPOSE_CLIENT_MEMO=128M                            # <- client's container maximum RAM usage to apply by docker-compose --------------------------------> #
COMPOSE_CLIENT_SWAP=256M                            # <- client's container RAM swap space in storage executed by automation command --------------------> #
COMPOSE_CLIENT_PORT=7712                            # <- local machine port opened for client's container service client --------------------------------> #
COMPOSE_CLIENT_USERNAME=admin.user                  # <- database client first user account -------------------------------------------------------------> #
COMPOSE_CLIENT_PASSWORD="admin-pass"                # <- database client first user password ------------------------------------------------------------> #
COMPOSE_IGNORE_ORPHANS=true                         # <- ignore docker warnings messages when client is in used -----------------------------------------> #manage automations for sharing resources -------------------------------> #
COMPOSE_IMGK="apache/kafka:3.9.2"                   # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_HOST=127.0.0.1                              # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_PORT_PTEXT=9092                             # <- local machine port opened for container service - plaintext client traffic ---------------------> #
COMPOSE_PORT_PCTRL=9093                             # <- local machine port opened for container service - ssl/tls or control-plane traffic -------------> #
COMPOSE_PATH="../data"                              # <- platform broker data storage in local ----------------------------------------------------------> #
COMPOSE_NAME="mp-kafka-dev"                         # <- container name to build the service - important to set the environment in this variable --------> #
COMPOSE_CPUS=2.00                                   # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_MEM=256M                                    # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_SWAP=512M                                   # <- container's RAM swap space in storage executed by automation command ---------------------------> #
COMPOSE_CLIENT_IMGK="provectuslabs/kafka-ui:latest" # <- container property for the database client for pulling and caching image into docker -----------> #
COMPOSE_CLIENT_CPUS=1.00                            # <- client's container maximum CPUs usage to apply by docker-compose - empty for full usage --------> #
COMPOSE_CLIENT_MEMO=128M                            # <- client's container maximum RAM usage to apply by docker-compose --------------------------------> #
COMPOSE_CLIENT_SWAP=256M                            # <- client's container RAM swap space in storage executed by automation command --------------------> #
COMPOSE_CLIENT_PORT=7712                            # <- local machine port opened for client's container service client --------------------------------> #
COMPOSE_CLIENT_USERNAME=admin.user                  # <- database client first user account -------------------------------------------------------------> #
COMPOSE_CLIENT_PASSWORD="admin-pass"                # <- database client first user password ------------------------------------------------------------> #
COMPOSE_IGNORE_ORPHANS=true                         # <- ignore docker warnings messages when client is in used -----------------------------------------> #
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
<br><br>

## <a id="management"></a>Service Management

To manage the container, run the GNU Make recipes
```bash
$ make help
Usage: $ make [target]
Targets:
$ make help                           shows this Makefile help message
$ make ports-check                    shows this project ports availability on local machine
$ make env                            checks if docker .env file exists
$ make env-set                        sets docker .env file
# -------------------------------------------------------------------------------------------------
#  Kafka
# -------------------------------------------------------------------------------------------------
$ make info                           shows container information
$ make ssh                            enters the container shell
$ make up                             starts and recreates containers if compose config or image changed, runnning in detached mode
$ make build                          builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
$ make network                        starts up into an existing custom network for container-to-container communication, runnning in detached mode
$ make start                          starts the container and put on running
$ make stop                           stops the running container but data will not be destroyed
$ make restart                        restarts the running container
$ make clear                          removes container from Docker running containers
$ make destroy                        delete container image from Docker cache
$ make dev                            sets a development enviroment
# -------------------------------------------------------------------------------------------------
#  Kafka Client
# -------------------------------------------------------------------------------------------------
$ make info-client                    shows client container information
$ make ssh-client                     enters the client container shell
$ make up-client                      starts and recreates client containers if compose config or image changed, runnning in detached mode
$ make build-client                   builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
$ make network-client                 starts up into an existing custom network for container-to-container communication, runnning in detached mode
$ make start-client                   starts the client container and put on running
$ make stop-client                    stops the running client container but data will not be destroyed
$ make restart-client                 restarts the running client container
$ make clear-client                   removes client container from Docker running containers
$ make destroy-client                 delete client container image from Docker images cache
$ make dev-client                     sets a development enviroment
```
<br><br>

---

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