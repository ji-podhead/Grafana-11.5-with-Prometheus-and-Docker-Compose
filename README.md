# Grafana-11.5-with-Prometheus-and-Docker-Compose
A Short Guide how to use Grafana 11.5 with Docker Compose. You will learn how to Provision Grafana so it comes with you Dashboards, Alerts, Datasources, Login, and Datasource-Configurations. 

Your team members can use Grafana with one command without going through the painful provision process.

- we will deploy grafana in combination with prometheus. We will scrape the following containers:
  - redis
  - postgres


## Grafana Container Structure
![grafana_container_structure](https://github.com/user-attachments/assets/ac9cafe7-fcb0-47e6-8d40-6119b352888e)

The default folders for provisioning are in `/etc/grafana/`
 - the config file is stored under `/etc/grafana/grafana.ini`
 - the yml files for provisioning the dashboards, alerts, etc, are located in  `/etc/grafana/provisioning`
   - so far i had no luck copiying the entire folder to the container, so i needed to do this one by one

***we will use a similar folderstructure for provisioning***
```
grafana/
|-- grafana.ini
|-- ldap.toml
|-- provisioning/
    |-- access-control/
    |   |-- default.yml
    |-- alerting/
    |   |-- default.yml
    |-- dashboards/
    |   |-- default.yml
    |-- datasources/
    |   |-- default.yml
    |-- notifiers/
        |-- default.yml
    |-- my_dashboards/
        |-- postgres.json
        |-- redis.json
```
> create the folders accordingly in your project
 ## files for provisioning

 - create your files for provisioning
 - keep the folder structure, so you dont need to rename files


### dashboards
```yaml
apiVersion: 1
providers:
- name: 'default'
  orgId: 1
  folder: ''
  folderUid: ''
  type: file
  options:
    path: /home/grafana/dashboards/postgres.json
```
> ./grafana/provisioning/dashboards/default.yml

### datasources
> we hardcoded the ip in the compose file so we can use it here and no manual steps are required after the provisioning
```yaml
apiVersion: 1
datasources:
- name: Prometheus
  type: prometheus
  url: http://200.0.0.10:9090 
  isDefault: true
  access: proxy
  editable: true
```
> ./grafana/provisioning/datasources/default.yml

## file permissions
- after you created your files you need to set permissions accordingly, or grafana container might fail
  - cd to your root project folder
```bash
sudo chmod -R +x ./grafana
sudo chmod -R 775 ./grafana
sudo chown -R 1000:1000 ./grafana
```
> you can check permissions by using `ll -R`

## docker compose file
> ***see the  file (here)[https://github.com/ji-podhead/Grafana-11.5-with-Prometheus-and-Docker-Compose/blob/main/docker-compose.yml]
### prometheus ip
- notice that we hardcode a ip to prometheus, so  we can use it in the grafana datasources provisioning file
```yml
...
      networks:
        network1:
          ipv4_address: 200.0.0.10
```
### grafana volumes
-  1. we create the required grafana volume
-  2. we copy our user dashboards over to `/home/grafana/dashboards`
   3. we overwrite the grafana.ini config file in `/etc/grafana/grafana.ini`
   4. we create the default.yml for the dashboards in `/etc/grafana/provisioning/dashboards/default.yml`
   5. we create the default.yml for the datasources with the hardcoded prometheus ip in `/etc/grafana/provisioning/datasources/default.yml`
   6. we create the default.yml for the alerting
```yml
---
      volumes:
        - grafana:/var/lib/grafana
        - ./grafana/my_dashboards:/home/grafana/dashboards
        - ./grafana/defaults.ini:/etc/grafana/grafana.ini
        - ./grafana/provisioning/dashboards/default.yml:/etc/grafana/provisioning/dashboards/default.yml
        - ./grafana/provisioning/datasources/default.yml:/etc/grafana/provisioning/datasources/default.yml
        - ./grafana/provisioning/alerting/default.yml:/etc/grafana/provisioning/alerting/default.yml
```

### grafana environment variables
- we can set certain grafana environment variables in the compose file
- the values for this can be set in the .env folder
```yml
...
      environment:
          GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
          GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH: /home/grafana/dashboards/postgres.json
          GF_USERS_ALLOW_SIGN_UP: false
          GF_PATHS_CONFIG: /etc/grafana/grafana.ini
```
> here we can set a few values like the default config path etc
- add a .env file in your docker compose project directory to use values like  `${GRAFANA_PASSWORD}` in your docker compose file
```yml
DB_HOST=postgresql
DB_PORT=5432
DB_NAME=monitoring

DB_USERNAME=homestead
DB_PASSWORD=secret

PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
GRAFANA_PASSWORD=secret
```
