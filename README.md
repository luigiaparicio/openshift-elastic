# openshift-elastic


## Install ArgoCD in OpenShift

    oc apply -k https://github.com/luigiaparicio/openshift-elastic/argocd/install
    
## Install ECK + ES + Kibana

    oc apply -k https://github.com/luigiaparicio/openshift-elastic/cluster-config/config/overlays/default
