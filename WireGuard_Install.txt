ssh -i ~/ssh-key.key ubuntu@IP_адрес_VPS

# 1. Обновляем систему и устанавливаем WireGuard
sudo apt update && sudo apt upgrade -y
sudo apt install wireguard -y

# 2. Переходим в режим root и создаем папки для ключей (если надо)
sudo -i
cd /etc/wireguard
umask 077 - безопасность создаваемых папок

mkdir -p server clients/client1
sudo mkdir -p /etc/wireguard/clients/client..3
sudo ls -la /etc/wireguard/clients

# 3. Генерируем ключи для сервера и клиента (путь без указания доп папок)
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

#PSK (доп ключ)
wg genpsk

# 4. Выводим имя сетевой карты и ключи на экран (чтобы скопировать в конфиг)
ip -br link - краткий список всех сетевых карт (важно для маршрутизации)

cat /etc/wireguard/server/server_private.key
cat /etc/wireguard/server/server_public.key
cat /etc/wireguard/clients/client/client_private.key
cat /etc/wireguard/clients/client/client_public.key

# 5. Открываем входящий порт на самом сервере (Можно не открывать если использовать вторые правила маршрутизации)
iptables -I INPUT -p udp --dport 51820 -j ACCEPT

# 6. Создаем конфигурацию сервера
nano /etc/wireguard/wg0.conf

[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = [ВСТАВИТЬ_СЮДА_server_private.key] 

# Правила маршрутизации выбрать одни из двух!

# Автоматические правила для интернета (активируются сами при старте VPN)
PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

# В этих правилах прописан шаг # 5, (порт не закрывается после перезагрузки впс) 
PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE; iptables -I INPUT -p udp --dport 51820 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE; iptables -D INPUT -p udp --dport 51820 -j ACCEPT

[Peer]
PublicKey = [ВСТАВИТЬ_СЮДА_client_public.key] 
AllowedIPs = 10.8.0.2/32

# 7. Включаем пересылку трафика в ядре Linux
sudo nano /etc/sysctl.conf

Найдите строку #net.ipv4.ip_forward=1 и просто удалите символ #

sudo sysctl -p
sysctl net.ipv4.ip_forward


# 8. Запускаем VPN и добавляем его в автозагрузку

sudo systemctl enable wg-quick@wg0 - автозапуск
sudo systemctl start wg-quick@wg0 - старт соединения

# 9. Проверяем, что всё поднялось успешно

wg show - статистика подключения

# Полезные команды 
sudo systemctl disable wg-quick@wg0 - снятие автозапуска
sudo systemctl stop wg-quick@wg0 - остановка соединения
sudo systemctl restart wg-quick@wg0 - перезагрузка соединения
sudo systemctl reset-failed wg-quick@wg0 - сброс ошибки
sudo systemctl status wg-quick@wg0 - статус

#Статус маршрутизации
ip rule show | grep 100
ip route show table 100
