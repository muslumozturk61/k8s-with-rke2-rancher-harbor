*docker'la harbora olu�turmu� oldu�umuz  `pushuser` kullan�c�s� ile login oluyoruz.*
```shell
$ docker login harbor.prometyum.net

#Note:logout olmak istersek a�a��daki komut kullan�l�r
$ docker logout harbor.prometyum.net 
```



*proje i�erisinden imaj olu�turulur*
```shell
$ docker build -t sample-api-image:v1 .
```

*imaj�m�z�n tag ini olu�turuyoruz ve harbor'a push yap�yoruz*
```shell
$ docker tag sample-api-image:v1 harbor.prometyum.net/pi-project/sample-api-image:v1

$ docker push harbor.prometyum.net/pi-project/sample-api-image:v1
```