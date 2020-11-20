# openshift-elastic


## Install ArgoCD in OpenShift

    until oc apply -k https://github.com/luigiaparicio/openshift-elastic/argocd/install; do sleep 3; done
    
## Add Nodes for Elastic

    CLUSTERID=$(oc get machineset -n openshift-machine-api -o jsonpath='{.items[0].metadata.labels.machine\.openshift\.io/cluster-api-cluster}')
    echo $CLUSTERID

    curl -s https://raw.githubusercontent.com/luigiaparicio/openshift-elastic/master/cluster-config/elastic-machineset.yaml | sed -e "s/CLUSTERID/${CLUSTERID}/g" | oc apply -f -
    
## Install ECK + ES + Kibana

    oc apply -k https://github.com/luigiaparicio/openshift-elastic/cluster-config/config/overlays/default
    
## Access to Kibana

  A default user named elastic is automatically created with the password stored in a Kubernetes secret

    oc get secret -n elastic-system elasticsearch-sample-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'

