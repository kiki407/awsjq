Examples of AWS DOCKER OPENSHIFT outputs filtered and sorted with jq / miller
===

Those are just examples they may not work for you, you might want to adapt them.

[jq is available here.](https://github.com/stedolan/jq)
[miller is available here.](https://github.com/johnkerl/miller)


### AWS

#### Ec2

##### All instances of a region names vs public ip address.
```
aws ec2 describe-instances  | jq '[ .Reservations[].Instances[0] | select(.NetworkInterfaces[0].Association.PublicIp) | {(.Tags[] | select (.Key=="Name") | .Value ): .NetworkInterfaces[0].Association.PublicIp }]'
```

##### All instances of a region names vs public ip addresses (elastic or not).
```
aws ec2 describe-instances | jq '[ .Reservations[].Instances[0] | select(.NetworkInterfaces[0].Association.PublicIp) | {(.Tags[] | select (.Key=="Name") | .Value ): [ .NetworkInterfaces[] | if .Association.IpOwnerId=="amazon" then { "publicip" : (.Association.PublicIp) } else { "elasticip" : (.Association.PublicIp) } end ]}]'
```

#### S3

##### Use miller to get "aws ls bucket" output into a json format.
```
aws s3 ls mybucket | mlr --ipprint --ojson label date,time,size,filename | jq
```

##### Same as previous but Latest 4 files in reverse datetime order.
```
aws s3 ls mybucket | mlr --ipprint --ojson label date,time,size,filename then sort -r date,time then head -n 4 | jq
```

#### CodePipeline

##### Executes pipeline and prints message while code pipeline is still running Print execution status at the end.
```
pipename=mypipeline; execid=$(aws codepipeline start-pipeline-execution --name ${pipename} | jq -r ".pipelineExecutionId"); while [[ $(aws codepipeline get-pipeline-execution --pipeline-name ${pipename} --pipeline-execution-id ${execid} | jq -r '.pipelineExecution.status') == "InProgress" ]] ; do echo pipeline still running; sleep 2; done; aws codepipeline get-pipeline-execution --pipeline-name ${pipename} --pipeline-execution-id ${execid} | jq -r '.pipelineExecution.status'
```


### Docker

##### Ip addresses of all running containers on a host
```
docker ps -q | xargs docker inspect | jq '[.[] | {"id":(.Id), "name": (.Name), "ips": [ (.NetworkSettings.Networks[].IPAddress) ] } ]'
```


### OPENSHIFT

##### All pods per status
```
oc get pod -a -o wide --all-namespaces -o json | jq 'reduce .items[] as $item ({}; .[($item.status.phase)] += [($item.metadata.name)])'
```

##### All pods per node then status
```
oc get pod -a -o wide --all-namespaces -o json | jq 'reduce .items[] as $item ( {}; .[($item.spec.nodeName)][($item.status.phase)] += [($item.metadata.name)] )'
```

##### All pods per node
```
oc get pod -a -o wide --all-namespaces -o json | jq 'reduce .items[] as $item ({}; .[($item.spec.nodeName)] += [($item.metadata.name)])'
```

##### Only the running nodes per pod
```
oc get pod -a -o wide --all-namespaces -o json | jq 'reduce .items[] as $item ({}; if $item.status.phase == "Running" then .[($item.spec.nodeName)] += [($item.metadata.name)] else . end )'
```


### RABBITMQ
#### find a message queue and return messages (and a few other things)
```
curl -s -u guest:guest http://localhost:15672/api/queues/ | jq ' . | map(select(.name == "thenameofthequeue")) | .[0] | {node:.node, name:.name, messages:.messages, messages_ready:.messages_ready}'
```

### SOLR
#### core information throu web api replace <corename> with your core name
```
curl -s 'http://127.0.0.1:8983/solr/admin/cores?action=STATUS&core=<corename>&wt=json' | jq '.status.<corename> | {corename: .name, size:.index.size}'
```