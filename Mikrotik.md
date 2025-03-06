внезапный микротик:
/system identity
set name=fw-dt.au.team 
/ip dns static
add name=fw-dt.au.team address=10.0.0.2

1. Настройка IP-адреса на интерфейсе ether2
Команды на MikroTik:
Copy
/ip address
add address=10.0.0.2/24 interface=ether2


Пояснение команд:
/interface vlan — создает VLAN-интерфейсы на основе физического интерфейса ether1 с указанными VLAN-идентификаторами (vlan-id).

vlan111 — VLAN с идентификатором 111.

vlan222 — VLAN с идентификатором 222.

vlan333 — VLAN с идентификатором 333.


/interface vlan
add name=vlan111 interface=ether1 vlan-id=111
add name=vlan222 interface=ether1 vlan-id=222
add name=vlan333 interface=ether1 vlan-id=333

/ip address
add address=10.100.190.1/26 interface=vlan111
add address=192.168.33.65/28 interface=vlan222
add address=192.168.33.81/29 interface=vlan333

Настройка NAT (Masquerade) на MikroTik
Предположим:
Внутренний интерфейс (где находится сеть 192.168.33.0/24) — ether1.

Внешний интерфейс (подключенный к интернету) — ether2.

Команды на MikroTik:
/ip firewall nat
add chain=srcnat src-address=10.100.190.0/24 out-interface=ether2 action=masquerade


1. Настройка статических маршрутов
Команды на MikroTik
/ip route
также в 0.0.0.0/0
add dst-address=10.110.190.0/26 gateway=10.0.0.1(r-dt)
add dst-address=192.168.11.64/28 gateway=192.168.33.89
add dst-address=192.168.11.80/29 gateway=192.168.33.89


/system ntp client
set enabled=yes primary-ntp=192.168.11.66

Настройка DHCP-сервера для VLAN 111
Команды на MikroTik:
shell
Copy
/ip pool
add name=dhcp-pool-vlan111 ranges=192.168.33.2-192.168.33.62

/ip dhcp-server
add name=dhcp-vlan111 interface=vlan111 address-pool=dhcp-pool-vlan111

/ip dhcp-server network
add address=192.168.33.0/26 gateway=192.168.33.1 dns-server=192.168.33.66 domain=au.team

прокси-проблема с сертификатом: 

Докер: 
Тебе нужно перетащить сертификат с share в мастерской на раб стол, потом через scp в нужную машину(srv):

1)scp /path/to/ca.root.crt user@192.168.33.X:/home/user/(учти , что реальная машина должна пинговать виртуальный сервер(srv-dt!)
cp /home/user/ca.crt /etc/pki/ca-trust/source/anchors/ 
 update-ca-trust
sudo usermod -a -G docker user
docker run hello-world sudo systemctl
enable --now docker
sudo systemctl status docker sudo mkdir -p /etc/systemd/system/docker.service.d 
sudo touch http-proxy.conf
sudo vim http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://10.0.83.52:3128/Environment="HTTPS_PROXY=http://10.0.83.52:3128/
sudo update-ca-certificates --fresh заново!

docker run hello-world
docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry: 2
docker ps

docker:
Установим пакет для работы с Docker:
apt-get update && apt-get install -y docker-engine
Запускаем и добавляем в автозагрузку службу docker:
systemctl enable --now docker.service
Создаём и запускаем локальный Docker Registry:
Поднимает контейнер Docker с именем DockerRegistry из образа registry:2 
Контейнер будет слушать сетевые запросы на порту 5000
Параметр --restart=always позволит автоматически запускаться контейнеру после перезагрузки сервера.
docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2


b) Напишите Dockerfile для приложения web.

1. В качестве базового образа используйте nginx:alpine
2. Содержание index.html
<html>
    <body>
        <center><h1><b>WEB</b></h1></center>
    </body>
</html>

Напишим Dockerfile для приложения web:
vim Dockerfile

![image](https://github.com/user-attachments/assets/3fa22d8e-fb48-42ed-9482-4e779adb424d)
Выполняем сборку docker-образа:
-t - позволяет присвоить имя собираемому образу;
"." - говорит о том что Dockerfile находится в текущей директории откуда выполняется данная команда и имеет имя именно Dockerfile:
docker build -t localhost:5000/web:1.0 .
docker push localhost:5000/web:1.0
docker run -d -p 80:80 --restart=always --name web localhost:5000/web:1.0




