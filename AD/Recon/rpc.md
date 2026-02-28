## Usuarios por RID Cycling
  ```
   rpcclient.py -U "guest%" IP -c "lookupnames administrator"
  administrator S-1-5-21-4078382237-1492182817-2568127209-500 (User: 1)
  ```
- Forca bruta
  ```seq 1000 2000 | xargs -P 10 -I {} rpcclient -U "guest%" 10.10.11.236 -c 'lookupsids S-1-5-21-4078382237-1492182817-2568127209-{}'```
  <img width="1519" height="721" alt="image" src="https://github.com/user-attachments/assets/4095fa6f-7e0d-4c3b-ac58-26bfbdbb6c05" />


## Grupos e usuarios
```
rpcclient -NU “usuario%pass” ip -c “enumdomgroups”
rpcclient -NU “usuario%pass” ip -c “enumdomusers”
```





