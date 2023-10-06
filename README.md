# asm-cross-cluster-test-01
testing singleton pod comms cross-cluster 

```
export CTX_1=autopilot-cluster-us-central1
export CTX_2=autopilot-cluster-us-east4

kubectl --context=$CTX_1 create namespace client 
kubectl --context=$CTX_2 create namespace server 

kubectl --context=$CTX_1 label namespace client istio-injection=enabled
kubectl --context=$CTX_2 label namespace server istio-injection=enabled

kubectl --context=$CTX_2 -n server apply -f server/
kubectl --context=$CTX_1 -n client apply -f client/

# check for service endpoints 
istioctl --context=$CTX_01 -n client proxy-config endpoints whereami-client-6b6f46b7-2bqv9

# shell to client pod to test 
kubectl --context=$CTX_01 -n client exec --stdin --tty whereami-client-6b6f46b7-2bqv9 -- /bin/sh

# set if adding namespace and service for `server` to remote clusters helps
kubectl --context=$CTX_1 create namespace server
kubectl --context=$CTX_1 label namespace server istio-injection=enabled
kubectl --context=$CTX_1 -n server apply -f server/service.yaml
```