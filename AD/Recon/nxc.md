- Coercao para relay de auth

```nxc smb <ip> -u '' -p '' -M coerce_plus```

- Certificados

```nxc ldap <ip> -u user -p pass -M adcs```

- Printnightmare

```nxc smb <ip> -u '' -p '' -M printnightmare```

- ZeroLogon

```nxc smb <ip> -u '' -p '' -M zerologon```

- NTLM reflection (CVE-2025-33073)

```nxc smb <ip> -u 'user' -p 'pass' -M ntlm_reflection```

- Enumeracao de discos

```nxc smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --disks```

- Enum de interfaces

```nxc smb 192.168.56.11 -u USERNAME -p 'PASSWORDHERE' --interfaces```

- Enum de arquivos compartilhados

```nxc smb 192.168.1.0/24 -u user -p 'PASSWORDHERE' --shares```

- Enum usuarios logados

```nxc smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --loggedon-users```

- Enum bitlocker

```nxc smb <ip> -u username -p password -M bitlocker```
