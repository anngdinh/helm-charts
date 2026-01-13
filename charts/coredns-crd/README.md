# CoreDNS CRD

This chart help deploy this [plugin](https://github.com/k8gb-io/coredns-crd-plugin).

## How to use

### deploy

```bash
helm repo add anngdinh https://anngdinh.github.io/helm-charts         
helm search repo anngdinh

helm --namespace=coredns-crd install coredns-crd anngdinh/coredns-crd --create-namespace

# get service clusterIP
kubectl -n coredns-crd get svc coredns-crd
# NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
# coredns-crd   ClusterIP   10.96.255.105   <none>        53/UDP,53/TCP   4m11s

# update default coredns to forward to this new coredns
kubectl -n kube-system edit cm coredns
# add this: forward vks.vngcloud.vn 10.96.255.105
```

### install crd

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/external-dns/refs/heads/master/charts/external-dns/crds/dnsendpoints.externaldns.k8s.io.yaml
```

### test

```bash
kubectl apply -f <(cat <<'EOF'
apiVersion: externaldns.k8s.io/v1alpha1
kind: DNSEndpoint
metadata:
  name: my-svc
  namespace: default
spec:
  endpoints:
  - dnsName: my-svc.vks.vngcloud.vn
    recordType: A
    recordTTL: 30
    targets:
    - 10.96.123.45
EOF
)


kubectl create deployment echo-server --image=vcr.vngcloud.vn/81-vks-public/nginx:alpine
kubectl exec deploy/echo-server -it -- nslookup my-svc.vks.vngcloud.vn
# Server:  10.96.0.10
# Address: 10.96.0.10:53

# Non-authoritative answer:
# Name: my-svc.vks.vngcloud.vn
# Address: 10.96.123.45
```
