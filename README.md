
## Provisioning a CA and Generating TLS Certificates

 Generate TLS certificates for the following components: 
 * etcd
 * kube-apiserver
 * kube-controller-manager
 * kube-scheduler, kubelet
 * kube-proxy.

### Install CloudFlare's PKI toolkit `cfssl`

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