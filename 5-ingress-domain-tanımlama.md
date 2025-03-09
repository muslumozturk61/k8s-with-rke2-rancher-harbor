
# Ingress domain tanımlama


## 1- sertifika üretimi

Öncelikle selfsigned sertifikamızı üretiyoruz. Bu sertifika ile pi-dev kurulumunu yapacağız.

```shell
$ mkdir pi-dev-certs && cd pi-dev-certs

$ apt install openssl

$ openssl genrsa -out ca-pi-dev.key 4096

$ openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=CN/ST=Istanbul/L=Istanbul/O=devops/OU=Personal/CN=pi-dev.prometyum.net" -key ca-pi-dev.key -out ca-pi-dev.crt

$ openssl genrsa -out pi-dev.prometyum.net.key 4096

$ openssl req -sha512 -new -subj "/C=CN/ST=Istanbul/L=Istanbul/O=devops/OU=Personal/CN=pi-dev.prometyum.net" -key pi-dev.prometyum.net.key -out pi-dev.prometyum.net.csr

$ cat > v3.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=pi-dev.prometyum.net
DNS.2=pi-dev.prometyum
DNS.3=pi-dev
EOF

$ openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca-pi-dev.crt -CAkey ca-pi-dev.key -CAcreateserial -in pi-dev.prometyum.net.csr -out pi-dev.prometyum.net.crt

$ openssl x509 -inform PEM -in pi-dev.prometyum.net.crt -out pi-dev.prometyum.net.cert

$ openssl x509 -inform PEM -in ca-pi-dev.crt -out ca-pi-dev.cert

# important! filename must be cacerts.pem
$ openssl x509 -inform PEM -in ca-pi-dev.crt -out cacerts.pem
```

---

## 2- ubuntuda kök sertifikaların tanılması

*Bu işlemler pi-k8s-master makinesinde yapılacaktır*

```shell
$ apt-get install -y ca-certificates

$ cp ca-pi-dev.crt /usr/local/share/ca-certificates

$ update-ca-certificates
```


## 3- ingress tanımlarının yapılması
```bash

kubectl -n default create secret tls tls-pi-dev-ingress --cert=pi-dev.prometyum.net.crt --key=pi-dev.prometyum.net.key

$ cat > pi-dev-ingress.yaml << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    field.cattle.io/publicEndpoints: '[{"addresses":["192.168.199.30","192.168.199.31","192.168.199.32"],"port":443,"protocol":"HTTPS","serviceName":"default:sample-api","ingressName":"default:pi-dev","hostname":"pi-dev.prometyum.net","path":"/","allNodes":false}]'
    meta.helm.sh/release-name: pi-dev
    meta.helm.sh/release-namespace: default
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"
  creationTimestamp: "2025-03-09T07:34:03Z"
  generation: 1
  labels:
    app: pi-dev
    release: pi-dev
  name: pi-dev
  namespace: default
  resourceVersion: "144132"
  uid: 22e1a548-d947-4f9b-aef0-d3660d800dfd
spec:
  rules:
  - host: pi-dev.prometyum.net
    http:
      paths:
      - backend:
          service:
            name: sample-api
            port:
              number: 8080
        path: /
        pathType: ImplementationSpecific
  tls:
  - hosts:
    - pi-dev.prometyum.net
    secretName: tls-pi-dev-ingress
status:
  loadBalancer:
    ingress:
    - ip: 192.168.199.30
    - ip: 192.168.199.31
    - ip: 192.168.199.32
EOF

$ kubectl apply -f pi-dev-ingress.yaml
```