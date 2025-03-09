# Harbor
**Harbor'ın** Ubuntu 22.04 ortamında kurulumu adımlarına buradan ulaşabilirsiniz.

---

**0:** minimum gereksinimler

| Resource | Minimum | Recommended |
| -------- | ------- | ----------- |
| CPU      |   2 CPU |	  4 CPU    |
| Mem      |   4 GB  |	  8 GB     |
| Disk     |  40 GB  |  160 GB     |

---

**1:** host dosyasına domain'in tanımlanması

```shell
$ echo '192.168.199.35  harbor.prometyum.net' | sudo tee -a /etc/hosts
$ hostnamectl set-hostname pi-harbor

# Ubuntu instructions 
# stop the software firewall
sudo systemctl disable --now ufw

# get updates, install nfs, and apply
sudo apt update
sudo apt install nfs-common -y  
sudo apt upgrade -y

# clean up
sudo apt autoremove -y

```
---

**2:** sertifika üretimi

```shell
$ mkdir harbor-files && cd harbor-files

$ sudo apt install openssl

$ openssl genrsa -out ca-harbor.key 4096

$ openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=CN/ST=Istanbul/L=Istanbul/O=devops/OU=Personal/CN=harbor.prometyum.net" -key ca-harbor.key -out ca-harbor.crt

$ openssl genrsa -out harbor.prometyum.net.key 4096

$ openssl req -sha512 -new -subj "/C=CN/ST=Istanbul/L=Istanbul/O=devops/OU=Personal/CN=harbor.prometyum.net" -key harbor.prometyum.net.key -out harbor.prometyum.net.csr

$ cat > v3.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.prometyum.net
DNS.2=harbor.prometyum
DNS.3=harbor
EOF

$ openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca-harbor.crt -CAkey ca-harbor.key -CAcreateserial -in harbor.prometyum.net.csr -out harbor.prometyum.net.crt

$ openssl x509 -inform PEM -in harbor.prometyum.net.crt -out harbor.prometyum.net.cert

$ openssl x509 -inform PEM -in ca-harbor.crt -out ca-harbor.cert
```

---

**3:** ubuntuda kök sertifikaların tanılması

*Bu işlemler hem master hemde worker node'larda yapılacaktır*

```shell
$ sudo apt-get install -y ca-certificates

$ sudo cp ca-harbor.crt /usr/local/share/ca-certificates

$ sudo update-ca-certificates
```

---

**4:** docker kurulumu

```shell
$ sudo apt update

$ sudo apt upgrade

$ sudo apt install curl lsb-release ca-certificates apt-transport-https software-properties-common -y

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt update

$ apt-cache policy docker-ce

$ sudo apt install docker-ce -y

$ sudo systemctl enable docker && sudo systemctl start docker

```

*check docker is working*

```shell
$ docker --version
```
---

**5:** docker-compose kurulumu

```shell
$ sudo usermod -aG docker ${USER}

$ sudo curl -SL https://github.com/docker/compose/releases/download/v2.32.4/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
 
$ sudo chmod +x /usr/local/bin/docker-compose

```

*check docker-compose is working*

```shell
$ docker-compose --version
```
---

**6:** harbor kurulumu

```shell
$ sudo mkdir /etc/harbor

$ sudo cp harbor.prometyum.net.crt /etc/harbor/

$ sudo cp harbor.prometyum.net.key /etc/harbor/
```

```shell
$ wget https://github.com/goharbor/harbor/releases/download/v2.12.2/harbor-online-installer-v2.12.2.tgz

$ tar xzvf harbor-online-installer-v2.12.2.tgz
```

*harbor/harbor.yml dosyasının düzenlenmesi*
```shell
$ cd harbor
$ sudo cp harbor.yml.tmpl harbor.yml
$ sudo nano harbor.yml
```

```yaml
hostname: harbor.prometyum.net
https:
    certificate: /etc/harbor/harbor.prometyum.net.crt
    private_key: /etc/harbor/harbor.prometyum.net.key

harbor_admin_password: Harbor12345
```


```shell
$ sudo ./prepare

$ sudo ./install.sh --with-trivy

$ sudo docker-compose up -d

# remove to container
# $ sudo docker-compose down -v

```

*harbor restart sorunu çözümü*

```shell

$ sudo -s

$ sudo cat > /etc/systemd/system/harbor.service << EOF
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/local/bin/docker-compose -f /home/muslum/harbor/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose -f /home/muslum/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target
EOF

$ exit

$ systemctl enable harbor

$ systemctl start harbor 

```

*kontrol etmek için internet tarayıcıdan harbor arayüzüne girilir*
[https://harbor.prometyum.net](https://harbor.prometyum.net)

```
Kullanıcı Adı: admin
Parola: Harbor12345 (harbor.yml içinde tanımlı olan paroladır.)
```
---

**7:** kullanıcı tanımlama

`"Administration\User"` menüsünden kullanıcı ların tanımlandığı sayfa açılır. `"NEW USER"` butonuna tıklayarak yeni bir kullanıcı eklenir. Bu dökümantasyondaki örnek senaryo için `"pushuser"` adında bir kullanıcı ile devam edilecektir.

Kullanıcı eklendikten sonra `"SET AS ADMIN"` butonuna tıklayarak kullanıcıya yetki verilme işlemi tamamlanmış olur.

---

**8:** private repository oluşturma

`"Project"` menüsünden repositorylerin olduğu sayfa açılır. `"NEW PROJECT"` butonuna tıklayarak yeni repository tanımlama işlemini yapıyoruz. Private repository oluşturmak istediğimiz için `"public"` özelliğinin seçili <ins>**olmamasına**</ins> dikkat ediyoruz.  

---

**9:** docker desktop üzerinden harbor'a imaj gönderme

*docker desktop ile harbor a bağlanmak için harbor' a ait ssl sertifikalar aşağıdaki dosyaların içine konulmalı ve  **Docker Desktop** uygulaması restart edilmelidir*

```
C:\Users\muslum\.docker\certs.d\harbor.prometyum.net

C:\ProgramData\docker\certs.d\harbor.prometyum.net
```

![docker-desktop-harbor-ssl](images/docker-desktop-harbor-ssl.png)

*docker'la harbora oluşturmuş olduğumuz  `pushuser` kullanıcısı ile login oluyoruz.*
```shell
$ docker login harbor.prometyum.net

#Note:logout olmak istersek aşağıdaki komut kullanılır
$ docker logout harbor.prometyum.net 
```



*proje içerisinden imaj oluşturulur*
```shell
$ docker build -t sample-api-image:v1 .
```

*imajımızın tag ini oluşturuyoruz ve harbor'a push yapıyoruz*
```shell
$ docker tag sample-api-image:v1 harbor.prometyum.net/pi-project/sample-api-image:v1

$ docker push harbor.prometyum.net/pi-project/sample-api-image:v1
```
---

## Kaynaklar

[https://goharbor.io/docs/1.10/install-config/installation-prereqs/](https://goharbor.io/docs/1.10/install-config/installation-prereqs/)

[https://www.youtube.com/watch?v=Zx4KTsxs0XE](https://www.youtube.com/watch?v=Zx4KTsxs0XE)

[https://github.com/alperen-selcuk/private-registry-harbor-installation](https://github.com/alperen-selcuk/private-registry-harbor-installation)
