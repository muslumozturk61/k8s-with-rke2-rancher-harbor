*docker'la harbora oluþturmuþ olduðumuz  `pushuser` kullanýcýsý ile login oluyoruz.*
```shell
$ docker login harbor.prometyum.net

#Note:logout olmak istersek aþaðýdaki komut kullanýlýr
$ docker logout harbor.prometyum.net 
```



*proje içerisinden imaj oluþturulur*
```shell
$ docker build -t sample-api-image:v1 .
```

*imajýmýzýn tag ini oluþturuyoruz ve harbor'a push yapýyoruz*
```shell
$ docker tag sample-api-image:v1 harbor.prometyum.net/pi-project/sample-api-image:v1

$ docker push harbor.prometyum.net/pi-project/sample-api-image:v1
```