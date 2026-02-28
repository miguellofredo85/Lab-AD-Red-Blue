- Nome de dominio, configuracoes

```ldapsearch -x -h IP -s base namingcontexts```

- Recolher dados de usuarios, computers, grupos, etc

```ldapdomaindump -u "dominio\usuario" -p "senha" IP```

- Query

```nxc ldap <ip> -u username -p password --query "(sAMAccountName=Administrator)"```

```nxc ldap <ip> -u username -p password --query "(sAMAccountName=Administrator)" "sAMAccountName objectClass pwdLastSet"```
