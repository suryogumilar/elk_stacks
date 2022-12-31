# ELK Service

## How to run

```
## run detached
docker-compose --project-name ss_db -f ./docker-compose.yaml up -d

## run elasticsearch detached
docker-compose --project-name ss_db -f ./docker-compose.yaml up -d elasticsearch 


## run elasticsearch undetached
docker-compose --project-name ss_db -f ./docker-compose.yaml up elasticsearch


## stop
docker-compose --project-name ss_db -f ./docker-compose.yaml stop

## remove
docker-compose --project-name ss_db -f ./docker-compose.yaml rm elasticsearch
docker-compose --project-name ss_db -f ./docker-compose.yaml rm kibana-service

## remove 2 services
docker-compose --project-name ss_db -f ./docker-compose.yaml rm kibana-service elasticsearch


#### log
docker-compose --project-name ss_db -f ./docker-compose.yaml logs --timestamp --follow


## stop and remove orphaned (CAREFULL, ALL CONTAINERs WITHIN PROJECT NAME THAT ARE RUNNING WILL BE REMOVED)

docker-compose --project-name ss_db -f ./docker-compose.yaml down --remove-orphans --volumes


```
## install and run logstash

 - wget https://artifacts.elastic.co/downloads/logstash/logstash-7.10.2-linux-x86_64.tar.gz

 - tar -zxvf logstash-7.10.2-linux-x86_64.tar.gz
 - mkdir /home/apps/logstash/ 
 - mkdir /home/apps/logstash/patterns
 - copy logstash.conf to /apps/logstash/
 - copy pattern to /apps/logstash/patterns
 - run bin/logstash -f /home
/apps/logstash/logstash.conf


## enable security

after elastics search is running go to its console, then navigate to "bin" folder present in installation directory of Elasticsearch and run: 

`./bin/elasticsearch-setup-passwords interactive`

```
Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana_system]:
Reenter password for [kibana_system]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:

```

some explanation about users:

`elastic`
   A built-in superuser.

`kibana_system`
   The user Kibana uses to connect and communicate with Elasticsearch.
   
`logstash_system`
   The user Logstash uses when storing monitoring information in Elasticsearch.
   
`beats_system`
   The user the Beats use when storing monitoring information in Elasticsearch.
   
`apm_system`
   The user the APM server uses when storing monitoring information in Elasticsearch.
   
`remote_monitoring_user`
   The user Metricbeat uses when collecting and storing monitoring information in Elasticsearch. It has the `remote_monitoring_agent` and `remote_monitoring_collector` built-in roles.

>
The built-in users serve specific purposes and are not intended for general use. In particular, do not use the elastic superuser unless full access to the cluster is required. Instead, create users that have the minimum necessary roles or privileges for their activities.

then restart elasticsearch and kibana. Next we can add user and password and also role in Kibana using *elastic* user as *superuser*. 

Creating Dashboard and spaces and asigning it to created user are also done using *elastic* user or other assigned *superuser*

Creating Index management and Index pattern also done using the same user after some indices has been submitted through elastic servers

### Kibana keystore

We can use keystore to store password instead store it in `kibana.yml` file as entry `elasticsearch.password` as a plain text.

 - Create the Kibana keystore:   
   `kibana-keystore create`
 - Add the password for the `kibana_system` user to the Kibana keystore:   
   `kibana-keystore add elasticsearch.password`   
   ```
   bash-4.4$ kibana-keystore add elasticsearch.password
   Enter value for elasticsearch.password: *********

   ```
 - Restart Kibana by kiling its PID and run `kibana` command or just restart docker container if we use docker   
   `kibana` 
### configure logstash security through Kibana

> Note that **Management > Roles** can be accesed through [**Stack management menu**](http://kibana_host:kibana_port/app/management)


Use the the **Management > Roles** UI in Kibana or the role API to create a `logstash_writer` role. 

For **cluster** privileges, add `manage_index_templates` and `monitor`. 

For **indices** privileges, add `write`, `create`, and `create_index`. For **Indices** add default `logstash-*`

Add `manage_ilm` for cluster and `manage` and `manage_ilm` for indices if you plan to use index lifecycle management.

And then Create a `logstash_internal` user and assign it the `logstash_writer` role. You can create users from the **Management > Users** UI in Kibana. 

Also add `logstash_system` role to enable monitoring elastics

test :

```
curl -u logstash_internal:password -X PUT "[elastic_host]:[elastic_port]/logstash-000001?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 2 
    }
  }
}
'
```

or run logstash agent and check if the indices were submitted and can be searched through **Index Management** in Kibana **Stack Management**

example running logstash agent on remote monitored machines

```
## enter into directory where logstash installed 
## or extracted

cd logstash-7.10.2
./bin/logstash -f /home/apps/logstash/logstash.conf
```

Or run logstash instances that connected to elasticsearch if we want to use it to digest/aggregates filebeat messages from other monitored resources

### configure logstash security keystore


We can use keystore to store password instead store it in `logstash.yml` file as a plain text.
 - set envi `LOGSTASH_KEYSTORE_PASS`
 - Create the logstash keystore:   
   `logstash-keystore create`
 - Add the password for the `logstash_internal` user to the logstash keystore:   
   `logstash-keystore add LGS_PWD`   
   
   ```
   bash-4.2$ logstash-keystore add LGS_PWD
   Using bundled JDK: /usr/share/logstash/jdk
   OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.

   Enter value for LGS_PWD:
   Added 'lgs_pwd' to the Logstash keystore.
   ```
   
 - Restart logstash


## Run Filebeat

Run filebeat in each monitored machines

1. Install filebeat
   ```
   ## get the binary
   curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.10.2-linux-x86_64.tar.gz
   
   ## extract it
   tar xzvf filebeat-7.10.2-linux-x86_64.tar.gz
   ```
2. Edit `filebeat.yml`
3. Check modules
   `./filebeat modules list`
4. enable logstash
   `./filebeat modules enable logstash`   
5. start filebeat
   `./filebeat -e`

### run via docker compose

edit `docker-compose_filebeat.yaml` volume to define the log directory location and also edit `filebeat.inputs.path` inside `filebeat.yml` file.

Then run:

`docker-compose --project-name ss_db -f ./docker-compose_filebeat.yaml up -d`

### Kibana space and dashboard for viewer user 

#### Create space

From space management create new space for viewing dashboards, name it for example: `dashboard_view` also set its features

and after that create role that can access this space 

#### Create role and user for viewing dashboard (through space)

login using *elastic* user to create new user and role to display dashboard through newly created space

Use the the **Management > Roles** UI in Kibana or the role API to create a `kibana_dashboard_viewer` role. 

For **Indices** add default `logstash-*`

For **cluster** privileges, add `monitor`. 

For **indices** privileges, add `manage`, `monitor`, `read`. `read_cross_cluster`, and `view_index_metadata`. 

Grant its access to Kibana. Give 'All' privileges if you want for this user to be able to access Index management and Index pattern.

Click `create role`.

Create also user with the newly created roles. Forexample 
user is `kibana_dv_user1` and assign `kibana_dashboard_viewer` role 

Tes by login into Kibana in another new session using the created user


Now you can access index management and index pattern to define metrices and displaying it to a dashboard
### Creating Index management and Index pattern

 - access [home dashboard kibana](http://kibana_host:5601/app/home)
 - access [index management](http://kibana_host:5601/app/management/data/index_management/indices)
 - make sure log from logstash is in indices list
 - access [index pattern](http://kibana_host:5601/app/management/kibana/indexPatterns) to create entry (Create index pattern menu)
 - Define an index pattern: choose index logstash that is available (for example `logstash-*`)
 - choose primary time field as global time filter. Then click 'Create Index Pattern'
 - edit fields inside created index 
 
#### Create dahsboard

Akses space and create necessary dashboard

 - access [dashboard](http://kibana_host:5601/app/dashboards) 
 - create new dashboard
 - Add an existing or new object to dashboard

## link for elastics and kibana

##### Elasticsearch

elastic_port usually 9200

Elastic search health :
http://[elastic_host]:[elastic_port]/_cat/health

##### Kibana

kibana_port usually 5601

http://[kibana_host]:[kibana_port]/

login using elastic user for create dashboard etc

### troubleshoot

###### running "Too Many Requests" error in kibana when create index pattern

possible cause : Low of disk space

try run :
`curl -XPUT -H "Content-Type: application/json" https://[YOUR_ELASTICSEARCH_ENDPOINT]:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'`

ref: 
 - https://stackoverflow.com/questions/50609417/elasticsearch-error-cluster-block-exception-forbidden-12-index-read-only-all


## Ref

#### authentication
 -  https://forum.uipath.com/t/enable-basic-authentication-on-elasticsearch-and-kibana/410484
 - https://www.elastic.co/guide/en/logstash/7.10/ls-security.html
 - https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html