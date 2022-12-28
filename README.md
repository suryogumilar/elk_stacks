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

then restart elasticsearch and kibana. Next we can add user and password and also role in Kibana using *elastic* user as *superuser*. 

Creating Dashboard and spaces and asigning it to created user are also done using *elastic* user or other assigned *superuser*

#### configure logstash security

Use the the **Management > Roles** UI in Kibana or the role API to create a `logstash_writer` role. 

For **cluster** privileges, add `manage_index_templates` and `monitor`. 

For **indices** privileges, add `write`, `create`, and `create_index`. For **Indices** add default `logstash-*`

Add `manage_ilm` for cluster and `manage` and `manage_ilm` for indices if you plan to use index lifecycle management.

And then Create a `logstash_internal` user and assign it the `logstash_writer` role. You can create users from the **Management > Users** UI in Kibana

test :

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