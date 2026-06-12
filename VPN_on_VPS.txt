# 1. Создаем конфиг VPN

sudo nano /etc/wireguard/vpn.conf

#Это добавить в конфиг vpn при стандартном рутинге wg0 и заполнить данными из конфига полученного от vpn сервиса.

[Interface]
PrivateKey = 
Address = 
DNS = 
Table = 100

PostUp = ip rule add from 10.8.0.0/24 table 100; iptables -t nat -A POSTROUTING -o vpn -j MASQUERADE
PostDown = ip rule del from 10.8.0.0/24 table 100; iptables -t nat -D POSTROUTING -o vpn -j MASQUERADE

[Peer]
PublicKey = 
AllowedIPs = 
Endpoint = 

# 2. Запускаем, добавляем в автозагрузку
sudo systemctl enable wg-quick@vpn - автозапуск
sudo systemctl start wg-quick@vpn - старт соединения

wg show - статистика подключения

# Полезные команды 

sudo systemctl disable wg-quick@vpn - снятие автозапуска
sudo systemctl stop wg-quick@vpn - остановка соединения
sudo systemctl restart wg-quick@vpn - перезагрузка соединения
sudo systemctl reset-failed wg-quick@vpn - сброс ошибки
sudo systemctl status wg-quick@vpn - статус
