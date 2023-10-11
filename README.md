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

# test call
curl -s http://35.193.33.33 | jq

# remove the $CTX_1 service before testing DNS proxying
kubectl --context=$CTX_1 -n server delete -f server/service.yaml

kubectl --context=$CTX_1 -n istio-system get cm/istio-asm-managed-rapid -o yaml

cat <<EOF | kubectl --context=$CTX_1 apply -f -
apiVersion: v1
data:
  mesh: |-
    defaultConfig:
      proxyMetadata:
        ISTIO_META_DNS_AUTO_ALLOCATE: "true"
        ISTIO_META_DNS_CAPTURE: "true"
kind: ConfigMap
metadata:
  name: istio-asm-managed-rapid # modify to your release channel
  namespace: istio-system
EOF

cat <<EOF | kubectl --context=$CTX_2 apply -f -
apiVersion: v1
data:
  mesh: |-
    defaultConfig:
      proxyMetadata:
        ISTIO_META_DNS_AUTO_ALLOCATE: "true"
        ISTIO_META_DNS_CAPTURE: "true"
kind: ConfigMap
metadata:
  name: istio-asm-managed-rapid # modify to your release channel
  namespace: istio-system
EOF

# delete existing pod to restart 
kubectl --context=$CTX_1 -n client delete pod whereami-client-6b6f46b7-2bqv9

# test call
curl -s http://35.193.33.33 | jq
{
  "backend_result": {
    "cluster_name": "autopilot-cluster-us-east4",
    "gce_instance_id": "7607513027198458324",
    "gce_service_account": "gateway-multicluster-01.svc.id.goog",
    "host_header": "whereami-server.server.svc.cluster.local",
    "metadata": "server",
    "node_name": "gk3-autopilot-cluster-us-east4-pool-2-77c50910-q5ih",
    "pod_ip": "10.86.0.183",
    "pod_name": "whereami-server-6c86c6f8c5-c9jcm",
    "pod_name_emoji": "💂🏿‍♂️",
    "pod_namespace": "server",
    "pod_service_account": "whereami-server",
    "project_id": "gateway-multicluster-01",
    "timestamp": "2023-10-11T23:07:23",
    "zone": "us-east4-b"
  },
  "cluster_name": "autopilot-cluster-us-central1",
  "gce_instance_id": "4170813972286196988",
  "gce_service_account": "gateway-multicluster-01.svc.id.goog",
  "host_header": "35.193.33.33",
  "metadata": "client",
  "node_name": "gk3-autopilot-cluster-us-centr-pool-2-a79948c0-mm44",
  "pod_ip": "10.127.1.171",
  "pod_name": "whereami-client-6b6f46b7-nkm66",
  "pod_name_emoji": "🤹🏻",
  "pod_namespace": "client",
  "pod_service_account": "whereami-client",
  "project_id": "gateway-multicluster-01",
  "timestamp": "2023-10-11T23:07:23",
  "zone": "us-central1-a"
}

```