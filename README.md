Ambassador / Consul Experiments
================================

```
# Initialise
# Must use larger size VMs in order to fit everything in
gcloud container clusters create ambassador-demo --machine-type n1-standard-2 --preemptible

kubectl create clusterrolebinding my-cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud info --format="value(config.account)")

kubectl apply -f helm-rbac.yaml
helm init --wait --service-account=tiller

helm repo add consul https://consul-helm-charts.storage.googleapis.com
helm install --name=consul consul/consul -f values.yaml

# This is a customized config to use the dev build of Ambassador with endpoint routing, and enable RBAC on ConfigMaps
kubectl apply -f ambassador-rbac.yaml
kubectl apply -f https://getambassador.io/yaml/ambassador/ambassador-service.yaml
kubectl apply -f https://getambassador.io/yaml/ambassador/pro/ambassador-consul-connector.yaml


# Port forward for the Ambassador Diagnostic Console and Consul UI
kubectl port-forward $(kubectl get pods --no-headers -o custom-columns=":metadata.name" | grep ambassador | head -n 1) 8877

# View in browser -> http://localhost:8877/ambassador/v0/diag/


kubectl port-forward $(kubectl get pods --no-headers -o custom-columns=":metadata.name" | grep consul-server | head -n 1) 8500

# View in browser -> http://localhost:8500/ui/dc1/services

# Apply our deplpoyment/config/mapping
kubectl apply -f qotm-deployment.yaml
kubectl apply -f qotm-config.yaml
kubectl apply -f qotm-consul-mapping.yaml

# Check Consul UI for service
# Check Ambassador UI for Mapping

# Get the AMBASSADOR_IP external load balancer IP
export AMBASSADOR_IP=$(kubectl get services --field-selector=metadata.name=ambassador -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}')


# Make the curl -- we're good!
curl -v $AMBASSADOR_IP/qotm-consul-ssl/

# Delete everything
kubectl delete -f qotm-consul-mapping.yaml
kubectl delete -f qotm-config.yaml
kubectl delete -f qotm-deployment.yaml

# Reapply everything
kubectl apply -f qotm-deployment.yaml
kubectl apply -f qotm-config.yaml
kubectl apply -f qotm-consul-mapping.yaml


# Wait until k8s shows consul-sd svc, and then check Ambassador UI for Mapping -- no Mapping registered?
# curl, with no success
curl -v $AMBASSADOR_IP/qotm-consul-ssl/

# Tidy up
gcloud container clusters delete ambassador-demo
```
