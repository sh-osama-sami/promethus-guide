# promethus-guide

# Prometheus, Node-Exporter, CAdvisor, Prom-Client, Alert-Manager, and Grafana Setup Guide

This guide provides step-by-step instructions for setting up various components of a monitoring stack including Prometheus, Node-Exporter, CAdvisor, Prom-Client for Node.js, Alert-Manager, and Grafana.

## Prerequisites

Before you begin, ensure you have the following prerequisites:

- Access to a Linux-based server (physical or virtual) 
- I chose to create an EC2 instance that has the following:   
  - Ubuntu as an operating system
  - A security group that allows
    - port 9090 for promethus
    - port 9100 for node_exporter


## 1. Installing Prometheus as a Service

1. I have written all the steps in an ansible file (to automate the execution of these configurations )
2. Then hit the instance ip on port 9090 (ensure it is allowed in the security group)
  - ![Screenshot from 2024-05-12 03-29-43](https://github.com/sh-osama-sami/promethus-guide/assets/85364511/7f6cea1d-e3a9-41d4-a560-bf31ec040b38)\
3. To allow /metrics on the prometheus service
  - edit the file in /etc/prometheus/prometheus.yaml


## 2. Installing Node-Exporter as a Service

1. Add ansible instructions to install node exporter
2. Then hit the instance on port 9100
  - ![Screenshot from 2024-05-12 20-45-50](https://github.com/sh-osama-sami/promethus-guide/assets/85364511/643392d2-5675-4cad-886d-3d4c03320a29)
3. Add localhost as a target to display it on the dashboard
  - edit the file in `/etc/prometheus/prometheus.yaml`
  - add a new job
    - ```yaml
        - job_name: 'node_exporter'
          static_configs:
          - targets: ['localhost:9100']
      ```
  - restart the service for the changes to take effect 
    - ```bash
        sudo systemctl restart prometheus
      ```
  - ![image](https://github.com/sh-osama-sami/promethus-guide/assets/85364511/dc33c42d-ebe6-4875-80b5-f943cf23bb95)

## 3. Add extra layer of security 
  - we need to configure a self signed public certificate and a private key at the node_exporter and enable prometheus to use this public certificate
    when sending requests to the node exporter this way will make communication between prometheus and node exporter secure (https)
  - The ansible of node exporter has the configuration for :
    - the creation of the public certificate
    - the creation of the private key
    - moving them to the node exporter path and granting the required permissions `/etc/node_exporter/`
    - hashing passwords 
    - adding autherised users with their hashed passwords to the `confi.yaml` 
    - attaching the certificate and the key to the service using a `config.yaml` file
  - for prometheus :
    - attaching the certificate and the credintials of the users in the `/etc/prometheus/prometheus.yaml` file (config file of prometheus)
  - the final result should look like this
    - ![image](https://github.com/sh-osama-sami/promethus-guide/assets/85364511/53107610-ac75-4a82-85f0-7ebcd9d204be)
    - here the prometheus is sending **https** requests to the node exporter with the username and password and the node exporter authenticate the password
    - and send https responses to prometheus 
      

## 4. Running CAdvisor Container

1. Install Docker on your server if not already installed.
2. Pull the CAdvisor Docker image: `docker pull google/cadvisor`.
3. Run the CAdvisor container: `docker run -d --name cadvisor --privileged -p 8080:8080 google/cadvisor:latest`.

## 5. Using Prom-Client for Node.js Code

1. Install prom-client npm package in your Node.js application: `npm install prom-client`.
2. Include prom-client in your Node.js application code and start instrumenting your code with Prometheus metrics.

## 6. Configuring Rules in Prometheus

1. Define alerting rules in the `prometheus.yml` configuration file.
2. Make sure to specify the alerting rules file in the Prometheus configuration.
3. Reload or restart Prometheus for the changes to take effect.

## 7. Installing and Configuring Alert-Manager

1. Download Alert-Manager from the official website: [Alert-Manager Downloads](https://prometheus.io/download/).
2. Extract the downloaded archive to a location on your server.
3. Modify the `alertmanager.yml` configuration file as per your requirements.
4. Run Alert-Manager as a service using systemd or another service manager.

## 8. Installing Grafana

1. Download Grafana from the official website: [Grafana Downloads](https://grafana.com/grafana/download).
2. Extract the downloaded archive to a location on your server.
3. Navigate to the Grafana directory.
4. Start Grafana by running `./bin/grafana-server`.
5. Access Grafana via your web browser using the default URL: `http://localhost:3000`.
6. Log in using the default credentials (admin/admin) and change them immediately.

## Additional Notes

- Make sure to configure firewall rules to allow access to the required ports for Prometheus, Node-Exporter, CAdvisor, and Grafana.
- Always ensure that you're using the latest versions of the software components to benefit from the latest features and security updates.

