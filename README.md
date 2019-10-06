
## Provisioning a CA and Generating TLS Certificates

 Generate TLS certificates for the following components: 
 * kubelet
 * kube-controller-manager
 * kube-proxy
 * kube-scheduler
 * kube-apiserver
 * Admin user

#### Install [CloudFlare](https://github.com/cloudflare/cfssl)'s PKI toolkit `cfssl`

```
brew install cfssl
```

### Provision a Certificate Authority

Provision a Certificate Authority that can be used to generate additional TLS certificates.

Generate the CA configuration file, certificate, and private key:

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Chicago",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "IL"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

Results:

```
ca-key.pem
ca.pem
```

### Kubelet certificates

```
{
for instance in worker-0 worker-1 worker-2; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "L": "Chicago",
      "ST": "IL",
      "C": "US"
    }
  ]
}
EOF

EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
}
```
Result
```
worker-0-key.pem
worker-0.pem
worker-1-key.pem
worker-1.pem
worker-2-key.pem
worker-2.pem
```

###  Controller Manager Client Certificate
```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "keys": {
    "alog": "rsa",
    "size": 2048
  },
  "names": [
   {
    "O": "system:kube-controller-manager",
    "OU": "Kubernetes The Hard Way",
    "L": "Chicago",
    "ST": "IL",
    "C": "US"
   }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
}
```

Results:
```
kube-controller-manager-key.pem
kube-controller-manager.pem
```

### Kube Proxy Client Certificate
```
{
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "keys": {
    "alog": "rsa",
    "size": 2048
  },
  "names": [
   {
    "O": "system:kube-proxy",
    "OU": "Kubernetes The Hard Way",
    "L": "Chicago",
    "ST": "IL",
    "C": "US"
   }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
}
```
Results:
```
kube-proxy-key.pem
kube-proxy.pem
```

### Scheduler Client Certificate
```
{
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "keys": {
    "alog": "rsa",
    "size": 2048
  },
  "names": [
   {
    "O": "system:kube-scheduler",
    "OU": "Kubernetes The Hard Way",
    "L": "Chicago",
    "ST": "IL",
    "C": "US"
   }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
}
```
Results:
```
kube-scheduler-key.pem
kube-scheduler.pem
```
### Kubernetes API Server Certificate
The `kubernetes-the-hard-way` static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients.


```
{
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local,api.kubernetes.blog

cat > kubernetes-csr.json <<EOF
{
  "CN": "system:kube-apiserver",
  "keys": {
    "alog": "rsa",
    "size": 2048
  },
  "names": [
   {
    "O": "system:kube-apiserver",
    "OU": "Kubernetes The Hard Way",
    "L": "Chicago",
    "ST": "IL",
    "C": "US"
   }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
}
```
Results:
```
kube-apiserver-key.pem
kube-apiserver.pem
```

### Admin user certificate

Generate the admin client certificate and private key:
```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
   {
    "O": "system:masters",
    "OU": "Kubernetes The Hard Way",
    "L": "Chicago",
    "ST": "IL",
    "C": "US"
   }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```
Results:
```
admin-key.pem
admin.pem
```
### Service Account Key Pair
```
{
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Chicago",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "IL"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```
Result:
```
	service-account.csr
	service-account.pem
```
### Certs for Node Authorization
Node authorization is a special-purpose authorization mode that specifically authorizes API requests made by kubelets.

