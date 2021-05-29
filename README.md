<!-- ABOUT THE PROJECT -->
## About The Project

This project is the docker composer project which setup MQTT Broker, InfluxDB, Grafana, Telegraf and Set Domain to Grafana Dashboard with SSL configuration.
You can test mqtt broker with MQTT Broker and show datas in Grafana Dashboard.

### Built With

This project will be installed with Docker Images with these:
* [Eclipse Mosquitto Docker Image](https://hub.docker.com/_/eclipse-mosquitto)
* [Influxdb Docker Image](https://hub.docker.com/_/influxdb)
* [Telegraf Docker Image](https://hub.docker.com/_/telegraf)
* [Grafana Docker Image](https://hub.docker.com/r/grafana/grafana)
* [Nginx Docker Image](https://hub.docker.com/_/nginx)
* [Certbot Docker Image](https://hub.docker.com/r/certbot/certbot)



<!-- GETTING STARTED -->
## Getting Started

This docker composer includes multiple containers which is working with the latest docker images, especially you can check the infludb with UI because we are using influxdb v2.0 and all datas can be shown in the bucket also.

This can be installed in local (Windows, Mac, Ubuntu) or VPS Server and AWS. (Verified in Local Windows, Mac OS, Ubuntu, VPS Server)


### Prerequisites
To work this, Server should be installed docker and docker compose file.
Please refer to install the docker compose : [Installing Docker and Docker Composer](https://docs.docker.com/compose/install/)

### Installation (Automatically)

1. Clone the repo
   ```sh
   git clone https://github.com/your_username_/Project-Name.git
   ```
2. Install Docker composer...
   ```sh
   docker-compose up -d
   ```
### Installation (Manually && Reference in Linux or VPS) 

1. Install MQTT Broker
    - Install MQTT Broker Container
    ```sh
        docker run -v /home/cosmo/volumes/mosquitto/config:/mosquitto/config -v /home/cosmo/volumes/mosquitto/data:/mosquitto/data -v /home/cosmo/volumes/mosquitto/log:/mosquitto/log -p 1883:1883 -p 9001:9001 --name mqtt eclipse-mosquitto
    ```

    - Create Username and Password
        ```sh
        docker exec -it exec mosquitto /bin/bash
        
        mosquitto_passwd /mosquitto/config/pwfile admin
        ```
    - Set MQTT Broker Configure in the mosquitto.conf
        Set listener to 1883 port

2. Verify MQTT Broker with Test file. 
    - Install Go 
    ```sh
        sudo apt install golang 
    ```
    - Run the golang file to verify MQTT Broker
        (e.g https://gabrieltanner.org/blog/grafana-sensor-visualization: step 4)
3. Install Influx DB 
    - Install Influx DB Container
    ```sh
        docker run -v /home/cosmo/volumes/influxdb/:/var/lib/influxdb2 -p 8086 ?name influxdb influxdb:2.0.6
    ```
    - Set Initial Configuration of Influxdb
    ```sh
        docker exec -it influxdb influx setup
    ```
    - Check 6 for setting telegraf and influxdb
4. Install Telegraf
    - Use the Telegraf Configuration from Repo and Set Influxdb2 db username and password, token, organization and bucket from initial configuration of InfluxDB
    - Set MQTT Broker Consumer in telegraf.conf
    - Install Telegraf Container
    ```sh
        docker run ?name telegraf -v /home/cosmo/volumes/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf :ro telegraf
    ```
5. Install Grafana
    - Install Grafana Docker Container
    ```sh
        docker run -it --name=grafana -p 3000:3000 grafana/grafana
    ```
6. Configuration MQTT, Telegraf, InfluxDB and Grafana
    - Set Telegraf in Influxdb
        Create the Telegraf configuration with bucket and get token
    - Set Influxdb in Telegraf
        Set token of influxdb which is generated while setting telegraf in influxdb in telegraf.conf
    - Set the [inputs.mqtt.consumer] in telegraf.conf and set mqtt username and password and telegraf server url and data_format
    - Set the Query code in Grafana
        Create Datasource as Influxdb and set influxdb setting in grafana
        Add Query code in Influxdb Data like this: 
        ```
            from(bucket:"sensor-bucket")
            |> range(start:-1h)
            |> filter(fn:(r) =>
                r._measurement == "weather" and
                
            )
            |> aggregateWindow(every: 1m, fn: mean
        ```
    - To confirm above settings, please check all configuration files in repos.
7. Set Nginx and Domain, SSL
    - Install Nginx Docker Container
        docker run -itd ?name nginx -p 80:80 nginx
    - Install Certbot Container
        docker run -it --name certbot certbot/certbot certonly --webroot --webroot-path=/var/www/certbot --email your-email@domain.com --agree-tos --no-eff-email -d domain.com -d www.domain.com
    - Set domain in nginx/conf.d/default.conf
8. Verify
    You can check all containers using docker portainers and check errors if you face some errors while installing.
     * [Docker Portainer](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/)
    