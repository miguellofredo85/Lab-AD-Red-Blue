
```smbmap -H IP (permisos por cada recurso compartilhado ex readonly)```


```smbmap -H IP  -r recurso```


```smbmap -H IP -u user -p pass -r recurso``` (se tem recursos -r mostra o que tem dentro)


```smbmap -H IP --download recurso/recurso/recurso``` (baixa um arquivo)


```smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt" ``` (subir arquivo) 
