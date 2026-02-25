[Docker](#docker)

[Bloodhound](#bloodhound)

[Injestor](#injestor)


## Docker
### Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

### Add the repository to Apt sources:
```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
sudo apt update


### Install the Docker packages.
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
sudo systemctl status docker
sudo systemctl start docker
sudo docker run hello-world
```

## Bloodhound
> Instalacao
1. wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
2. tar -xvzf bloodhound-cli-linux-amd64.tar.gz
3. ./bloodhound-cli install

> Commando uteis
./bloodhound-cli resetpwd (se quiser mudar a senha)
http://localhost:8080/ui/login (admin:admin)
./bloodhound-cli update
docker compose up (dentro da pasta para cada vez que queira usar)
<img width="1209" height="563" alt="runbloodhound" src="https://github.com/user-attachments/assets/eb7fb4b4-b223-4996-a7b4-de81b1d22c08" />

> se tiver esse problema aqui
<img width="1504" height="227" alt="pro" src="https://github.com/user-attachments/assets/9b7d154b-de6e-418f-b95c-76894eb4ddbd" />

> instale o seguinte no chrome
<img width="1413" height="717" alt="chrome" src="https://github.com/user-attachments/assets/cad4d91c-e750-40e7-ba82-aea933ca65ed" />

## Injestor
- Netexec
```nxc ldap IP -u usuario -p senah --bloodhound --collection All --dns-server IP ``` (a salida sera um arquivo de data .zip)
- No bloodhound gui vai no Administrator
- Upload arquivo.zip e aguarde hasta completar
<img width="1681" height="779" alt="ggg" src="https://github.com/user-attachments/assets/a3352c81-6015-45e3-a066-d654ae92c23b" />




