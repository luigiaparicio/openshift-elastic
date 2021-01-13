# openshift-elastic

*Test Env: OCP 4.5 on AWS us-east-2*


## Install ArgoCD in OpenShift

    until oc apply -k https://github.com/luigiaparicio/openshift-elastic/argocd/install; do sleep 3; done
    
## Add Nodes for Elastic

### Create MachineSet

    CLUSTERID=$(oc get machineset -n openshift-machine-api -o jsonpath='{.items[0].metadata.labels.machine\.openshift\.io/cluster-api-cluster}')
    echo $CLUSTERID

    curl -s https://raw.githubusercontent.com/luigiaparicio/openshift-elastic/master/cluster-config/elastic-machineset.yaml | sed -e "s/CLUSTERID/${CLUSTERID}/g" | oc apply -f -
    
    
  After a couple of minutes you'll see new Nodes added and in Ready state
  
    oc get nodes -l elastic-cluster=elasticsearch-sample
    
    
  Example output:
   
    NAME                                         STATUS   ROLES    AGE    VERSION
    ip-10-0-132-221.us-east-2.compute.internal   Ready    elastic  100s   v1.18.3+10e5708
    ip-10-0-174-238.us-east-2.compute.internal   Ready    elastic  100s   v1.18.3+10e5708
    ip-10-0-216-236.us-east-2.compute.internal   Ready    elastic  99s    v1.18.3+10e5708
    
### Create MachineConfigPool

    oc apply -f https://raw.githubusercontent.com/luigiaparicio/openshift-elastic/master/cluster-config/elastic-mcp.yaml
    
### Create Tuned config

    oc apply -f https://raw.githubusercontent.com/luigiaparicio/openshift-elastic/master/cluster-config/elastic-tuned.yaml

    
## Install ECK + ES + Kibana

    oc apply -k https://github.com/luigiaparicio/openshift-elastic/cluster-config/config/overlays/default
    
## Access to Kibana

  A default user named elastic is automatically created with the password stored in a Kubernetes secret

    oc get secret -n elastic-system elasticsearch-sample-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'

## Install openshift-logging

openshift-logging must be installed in the cluster

- OperatorHub -> Openshift Logging -> install in openshift-logging namespace
- OperatorHub -> Elasticsearch 4.5 by Red Hat -> install all namespaces
- Create Openshift logging Instance (follow Form steps)
  - Log store:  2Gi Req, 2Gi Limit, 50G disk space, ZeroRedundancy
