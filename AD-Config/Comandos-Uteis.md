# Windows 

- Abrir porta
  ```New-NetFirewallRule -DisplayName "Permitir Puerto 8080" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow```
- Habilitar execucao de script
  ```Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process```
