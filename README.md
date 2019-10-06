
## Provisioning a CA and Generating TLS Certificates

 Generate TLS certificates for the following components: 
 * etcd
 * kube-apiserver
 * kube-controller-manager
 * kube-scheduler, kubelet
 * kube-proxy.

### Install [CloudFlare](https://github.com/cloudflare/cfssl)'s PKI toolkit `cfssl`

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

### Certs for Node Authorization
Node authorization is a special-purpose authorization mode that specifically authorizes API requests made by kubelets.

