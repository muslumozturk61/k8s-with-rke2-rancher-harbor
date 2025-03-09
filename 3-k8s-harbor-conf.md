# Kubernetes-Harbor Ayarları
**Kubernetes ile Harbor** arasındaki yapılması gerekenlerin Ubuntu 22.04 ortamındaki adımlarına buradan ulaşabilirsiniz.

Harbor kurulum sürecinde oluşturduğumuz sertifikaları kullanacacağız.

**1:** host dosyasına domain'in tanımlanması

*Bu işlemler hem master hemde worker node'larda yapılacaktır*

```shell
$ echo '192.168.199.35  harbor.prometyum.net' | sudo tee -a /etc/hosts
```

**2:** ubuntuda kök sertifikaların tanılması

*Bu işlemler hem master hemde worker node'larda yapılacaktır*

```shell
$ sudo apt-get install -y ca-certificates

$ sudo cp selfsignCA.crt /usr/local/share/ca-certificates

$ sudo update-ca-certificates
```

rancher'dan harbor registery secret olarak eklenir.

![rancher harbor registry create](img-k8s/rancher-harbor-registry-create.png)


## Kaynaklar

[https://ubuntu.com/server/docs/security-trust-store](https://ubuntu.com/server/docs/security-trust-store)


[https://www.youtube.com/watch?v=Zx4KTsxs0XE](https://www.youtube.com/watch?v=Zx4KTsxs0XE)

[https://github.com/alperen-selcuk/private-registry-harbor-installation](https://github.com/alperen-selcuk/private-registry-harbor-installation)

[https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_set/kubectl_set_serviceaccount/