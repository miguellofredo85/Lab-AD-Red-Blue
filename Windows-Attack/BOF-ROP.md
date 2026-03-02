### Vamos comecar desde a parte do BOF anterior na qual o mona vai procural pelo JMP ESP, so que agora com o DEP ativado, nao poderemos executar codigo desde o stack.
- Para vencer o DEP, vamos usar a técnica de ROP (Return-Oriented Programming). O objetivo não é mais pular para o shellcode, mas sim montar uma "corrente" de endereços que chame uma função do Windows para desativar o DEP antes de rodar o seu código.

- Desative ASLR Para começar, desative o ASLR globalmente  para que os endereços das DLLs não mudem. Assim você foca apenas em vencer o DEP primeiro.

  . `Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "MoveImages" -Value 0`

  ou

  . `bcdedit /set {current} moveimages Never`
