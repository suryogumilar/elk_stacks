# Elastic access API via CURL



```sh
## test akses

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200

## cluster health

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_cluster/health

### pretty print cluster health

curl -s --cacert cca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_cluster/health?pretty=true



## list all index
curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_aliases?pretty=true

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_cat/indices?v


curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_cat/indices  | cut -d\  -f3

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_status

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_nodes/stats/indices?pretty=true

## show stats
curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_stats?pretty=true


curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_nodes/stats?pretty=true


## IMPORTANT TO TEST KIBANA ELASTICSEARCH INTEGRATION !!!

## run this in the elasticsearch first

echo $ELASTIC_PASSWORD

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_nodes?filter_path=nodes.*.version%2Cnodes.*.http.publish_address%2Cnodes.*.ip

curl -s --cacert config/certs/ca/ca.crt -u kibana_system:${ELASTIC_PASSWORD} https://localhost:9200/_nodes?filter_path=nodes.*.version%2Cnodes.*.http.publish_address%2Cnodes.*.ip

### %2C = ,

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_nodes?filter_path=nodes.*.version,nodes.*.http.publish_address,nodes.*.ip


### get user's privileges 

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_security/user/_privileges


#### for privileseg of certain user:

curl -s --cacert config/certs/ca/ca.crt -u some_user:some_password https://localhost:9200/_security/user/_privileges?pretty=true


#### for cheking role 
curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://localhost:9200/_security/role/itgc_role_user?pretty=true
```


#### set role to  allow restricted indices to true

```sh

curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} \
-X POST \
-H "Content-Type: application/json" \
-d '{ "indices": [ { "names": [ "ds-logs-itgc_user-default-*"], "privileges": ["all", "manage", "read" ], "allow_restricted_indices": true } ] }' \
"https://localhost:9200/_security/role/itgc_role_user"


curl -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} \
-X POST \
-H "Content-Type: application/json" \
-d '{ "indices": [ { "names": [ "ds-logs-itgc_user-default-001"], "privileges": ["all", "manage", "read" ], "allow_restricted_indices": true } ] }' \
"https://localhost:9200/_security/role/itgc_role_user"
```