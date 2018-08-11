# A Prometheus & Grafana docker-compose stack for Raspberry Pi


Emerging trends like microservices & containerization on IoT devices like Raspberry Pi, BananaPi, ZeroPi etc., all keep pushing the 
boundaries of what constitutes DevOps Monitoring. We have customers using Prometheus that are monitoring and controlling their IoT 
infrastructure , application and database instances, and containers—providing a consolidated view or single pane of glass solution to 
their DevOps monitoring needs. Dockerworx Ready Solution for Prometheus Stack comprises of ready-to-use Prometheus, Grafana, cAdvisor, 
Grafana, Zipkin & AlertManager all integrated with Slack channel for real-time alerts and management.

## What’s unique about Prometheus?
 
Prometheus is a self-hosted set of tools which collectively provide metrics storage, aggregation, visualization and alerting. Usually most of the available monitoring tools are push based, i.e. agents on the monitored servers talk to a central server (or set of servers) and send out their metrics. Prometheus is little different on this aspect – It is a pull based server which expects monitored servers to provide a web interface from which it can scrape data. This is a unique characteristic of Prometheus.

## Main Features of Prometheus

- Decentralized architecture
- Scalable data collection
- Powerful Query language (leverages the data model for meaningful alerting (including easy silencing) and graphing (for dashboards and for ad-hoc exploration).
- A multi-dimensional data model.( data can be sliced and diced at will, along dimensions like instance, service, endpoint, and method)
- No reliance on distributed storage; single server nodes are autonomous.
- Time series collection happens via a pull model over HTTP.
- Pushing time series is supported via an intermediary gateway.
- Targets are discovered via service discovery or static configuration.
- Multiple modes of graphing and dashboard support.

You can deep dive more into its architecture at https://prometheus.io/docs/introduction/overview/#architecture.

## Pre-requisite:

## Hardware:

- Raspberry Pi 3 ( You can order it from Amazon in case you are in India for $35)
- Micro-SD card reader ( I got it from here )
- Any Windows or Linux Desktop or Laptop
- HDMI cable ( I used the HDMI cable of my plasma TV)
- Internet Connectivity(Wifi/Broadband/Tethering using Mobile) – to download Docker 1.12.1 package
- Keyboard & mouse connected to Pi’s USB ports


## Software:

- SD-Formatter – to format microSD card
- Win32 disk imager(in case you have Windows OS running on your laptop) – to burn Raspbian Jessie ISO into microSD card

## Steps:

Format the microSD card using SD Formatter. Insert the microSD card into your Pi box. Now connect the HDMI cable  from one end of Pi’s HDMI slot to your TV or display unit and mobile charger(recommended 5.1V@1.5A).

## Installing Docker

Use the below command to bring up Docker

```
$curl -sSL https://get.docker.com/ | sh
```

## Verify if Docker is installed or not using the below command:

```
$docker version
```

## Installing docker-compose

```
apt-get install -qy python-pip --no-install-recommends
pip install pip --upgrade
pip install docker-compose
pip install --upgrade --force-reinstall docker-compose
```

Under this blog post, we will see how to bring Dockerworx Ready Solution for Prometheus Stack which for an ARM monitoring stack containing [Prometheus](http://prometheus.io/), Grafana, cAdvisor and Node scraper to monitor your Docker infrastructure. The images used are multi-arch for ARM32 and ARM64 and correctly fetched from DockerHub using metadata from your archtecture.




# Installation & Configuration

Clone the project locally to your Docker host.

```
git clone https://github.com/carlosedp/arm-monitoring.git
cd arm-monitoring
```

This deployment uses XXXXXX to provide external accessible dashboards thru Traefik, in case external access is not needed, remove all references to Traefik from the `docker-compose.yml` file:

From `networks:` section, remove:

  traefik:
    external:
      name: containermgmt_traefik

And from the Grafana and Portainer containers, remove the traefik labels and remove from `networks:` section:

    - traefik

If you would like to change which targets should be monitored or make configuration changes edit the `/prometheus/prometheus.yml` file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yml` file.

Once configurations are done let's start it up. From the cloned project directory run the following command:

    docker-compose up -d

To check the containers:

    docker-compose ps

or in case of using Docker Swarm (not tested)

    $ HOSTNAME=$(hostname) docker stack deploy -c docker-compose.yml prom

That's it the `docker stack deploy` command deploys the entire stack automagically. By default cAdvisor and node-exporter are set to Global deployment which means they will propogate to every docker host attached to the Swarm.

In order to check the status of the newly created stack:

Docker-compose:

    docker-compose ps

Swarm:

    $ docker stack ps rpi-monitoring

View running services:

    $ docker ps

View logs for a specific service

    $ docker-compose logs rpimonitoring_<service_name>

## Portainer

[Portainer](https://portainer.io/) is also installed as part of the stack. You can access it using the URL `http://<Host IP Address>:9000` for example http://192.168.10.1:9000, Portainer will ask to create a new user. For install details, check the [official site](https://portainer.io/install.html).


## Grafana

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000

    username - admin
    password - admin (Password is stored in the `config.monitoring` env file)

## Post Configuration
Create the Prometheus Datasource in order to connect Grafana to Prometheus
* Click the `Grafana` Menu at the top left corner (looks like a fireball)
* Click `Data Sources`
* Click the green button `Add Data Source`.

## Install Dashboard
There area a couple of dashboards for monitoring the infrastructure and Docker in the repository. Simply select from the Grafana menu -> Dashboards -> Import or import te default ones from Grafana website using the ID.

* [Docker Dashboard](https://grafana.net/dashboards/179).
* [Docker and System Monitoring](https://grafana.net/dashboards/893).
* [Node monitoring - Hardware](https://grafana.net/dashboards/1860).

## Alerting - Currently disabled
Alerting has been added to the stack with Slack integration. 2 Alerts have been added and are managed.

Alerts              - `prometheus/alert.rules`
Slack configuration - `alertmanager/config.yml`

The Slack configuration requires to build a custom integration.
* Open your slack team in your browser `https://<your-slack-team>.slack.com/apps`
* Click build in the upper right corner
* Choose Incoming Web Hooks link under Send Messages
* Click on the "incoming webhook integration" link
* Select which channel
* Click on Add Incoming WebHooks integration
* Copy the Webhook URL into the `alertmanager/config.yml` URL section
* Fill in Slack username and channel

View Prometheus alerts `http://<Host IP Address>:9090/alerts`
View Alert Manager `http://<Host IP Address>:9093`

### Test Alerts
A quick test for your alerts is to stop a service. Stop the node_exporter container and you should notice shortly the alert arrive in Slack. Also check the alerts in both the Alert Manager and Prometheus Alerts just to understand how they flow through the system.

High load test alert - `docker run --rm -it busybox sh -c "while true; do :; done"`

Let this run for a few minutes and you will notice the load alert appear. Then Ctrl+C to stop this container.

# Backup and Restore

The repository includes two scripts to launch a docker container to backup and restore your docker volumes. Adjust the destination in the variable in the backup script.

The backup script will backup all your docker volumes or the volume specified. To launch:

    ./backup_docker_volumes.sh -a    # To backup all volumes

    ./backup_docker_volumes.sh volume_name

The restore will take the .tar.gz file and restore it to an existing volume (asking to replace files) or create a new one if it does not exist:

    ./restore_docker_volume.sh docker_volume-20180117_175449.tar.gz

# Security Considerations
This project is intended to be a quick-start to get up and running with Docker and Prometheus. Security has not been implemented in this project. It is the users responsability to implement Firewall/IpTables and SSL.

Since this is a template to get started Prometheus and Alerting services are exposing their ports to allow for easy troubleshooting and understanding of how the stack works.

## Production Security:
Here are just a couple security considerations for this stack to help you get started.
* Remove the published ports from Prometheus and Alerting servicesi and only allow Grafana to be accessed
* Enable SSL for Grafana with a Proxy such as [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) or [Traefik](https://traefik.io/) with Let's Encrypt
* Add user authentication via a Reverse Proxy [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) or [Traefik](https://traefik.io/) for services cAdvisor, Prometheus, & Alerting as they don't support user authenticaiton
* Terminate all services/containers via HTTPS/SSL/TLS
