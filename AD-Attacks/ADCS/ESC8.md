## Compreendendo o Ataque ESC8
- Antes de colocarmos a mão no teclado, é crucial entendermos o que vamos fazer. O ataque segue esta lógica :
- O Alvo: Uma Autoridade Certificadora (CA) com o serviço de "Inscrição Web de Certificados" (CES) ativo e acessível via HTTP (não HTTPS).
- A Falha: Esse endpoint web aceita autenticação NTLM, um protocolo vulnerável a ataques de "relay" ou "retransmissão" .
- O Ataque: Nós, como atacantes (na máquina THEPUNISHER), vamos forçar um servidor privilegiado (o Domain Controller HYDRA-DC) a tentar se autenticar em um servidor malicioso que controlamos.
- O Relay: Em vez de completar a autenticação, nosso servidor malicioso vai "retransmitir" essa tentativa de autenticação para o endpoint web vulnerável do AD CS.
- O Resultado: O AD CS, achando que a solicitação veio do HYDRA-DC, emite um certificado para o próprio HYDRA-DC.
- O Impacto: Com o certificado do Controlador de Domínio, podemos forjar tickets e efetivamente controlar todo o domínio


- Para o ESC8 funcionar, a CA precisa:
  . Ter o papel de "Certification Authority Web Enrollment" instalado. (Provavelmente já está, pois você teve resultados no certipy).
  . Ter o endpoint HTTP (porta 80) habilitado e acessível. Esta é a misconfiguração chave.Aceitar autenticação NTLM neste endpoint (é o comportamento padrão).


Abra o Server Manager no HYDRA-DC → Manage → Add Roles and Features → Clique em Next até chegar em Server Roles → 
Na lista de funções, encontre e expanda (não apenas marque) Active Directory Certificate Services → Agora você verá a lista de Role Services:

  - Certification Authority (já deve estar marcado)
  - Certification Authority Web Enrollment (é este que precisamos marcar) → MARQUE a opção "Certification Authority Web Enrollment" →
      Uma janela pop-up vai aparecer perguntando sobre adicionar recursos necessários (IIS). Clique em Add Features
      Clique em Next até chegar em Confirm installation selections
      Clique em Install
<img width="1421" height="784" alt="config1" src="https://github.com/user-attachments/assets/e13f6896-eb1e-4481-911b-fe6cf3ad5e36" />
  - Click no link e instale 
<img width="1262" height="730" alt="conf2" src="https://github.com/user-attachments/assets/8e963600-adde-4a7e-baad-77ab9f35300b" />


 - Verificar se o IIS Manager Apareceu
  Agora, ao ir em Tools no Server Manager, você deve ver:

  ✅ Internet Information Services (IIS) Manager
<img width="1393" height="715" alt="3" src="https://github.com/user-attachments/assets/0e874b07-e86a-4c50-adc8-b431e6186ea3" />

 - Isso aqui quere dizer que esta funcionando!
<img width="962" height="268" alt="4" src="https://github.com/user-attachments/assets/5953ae7a-7e4d-493c-b12f-6259fb25f17f" />


### Recon
```certipy find -u 'fcastle@MARVEL.local' -p 'MYpassword#' -dc-ip 192.168.0.40 -vulnerable -stdout | grep -A 10 "Web Enrollment"```
<img width="676" height="763" alt="5" src="https://github.com/user-attachments/assets/b7b3cb21-dfcb-4cef-a046-ad76512b3c8c" />

### Attack

# em proceso............



























