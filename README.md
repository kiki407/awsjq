Examples of AWS DOCKER OPENSHIFT outputs filtered and sorted with jq / miller
===

### AWS

#### All instances of a region names vs public ip address.
```
aws ec2 describe-instances  | jq '[ .Reservations[].Instances[0] | select(.NetworkInterfaces[0].Association.PublicIp) | {(.Tags[] | select (.Key=="Name") | .Value ): .NetworkInterfaces[0].Association.PublicIp }]'
```

#### All instances of a region names vs public ip addresses (elastic or not).
```
aws ec2 describe-instances | jq '[ .Reservations[].Instances[0] | select(.NetworkInterfaces[0].Association.PublicIp) | {(.Tags[] | select (.Key=="Name") | .Value ): [ .NetworkInterfaces[] | if .Association.IpOwnerId=="amazon" then { "publicip" : (.Association.PublicIp) } else { "elasticip" : (.Association.PublicIp) } end ]}]'
```

#### Use miller to get "aws ls bucket" output into a json format.
```
aws s3 ls mybucket | mlr --ipprint --ojson label date,time,size,filename | jq
```

#### Same as previous but Latest 5 files in reverse datetime order.
```
aws s3 ls mybucket | mlr --ipprint --ojson label date,time,size,filename then sort -r time then sort -r date then head -n 5 | jq
```

### Docker

#### Ip addresses of all running containers on a host
```
docker ps -q | xargs docker inspect | jq '[.[] | {"id":(.Id), "name": (.Name), "ips": [ (.NetworkSettings.Networks[].IPAddress) ] } ]'
```
