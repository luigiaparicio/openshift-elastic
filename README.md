# openshift-elastic


## Install ArgoCD in OpenShift

    until oc apply -k https://github.com/luigiaparicio/openshift-elastic/argocd/install; do sleep 3; done
    
## Install ECK + ES + Kibana

    oc apply -k https://github.com/luigiaparicio/openshift-elastic/cluster-config/config/overlays/default
    
## Access to Kibana

  A default user named elastic is automatically created with the password stored in a Kubernetes secret

    oc get secret -n elastic-system elasticsearch-sample-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'

