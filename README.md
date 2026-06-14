# 🛡️ WireGuard VPN — Установка и настройка на Ubuntu

Пошаговое руководство по развёртыванию WireGuard VPN-сервера на Ubuntu VPS.

-----

## 📋 Содержание

1. [Подключение к серверу](#-подключение-к-серверу)
1. [Установка WireGuard](#-1-установка-wireguard)
1. [Создание папок и генерация ключей](#-2-создание-папок-и-генерация-ключей)
1. [Просмотр ключей](#-3-просмотр-ключей)
1. [Открытие порта](#-4-открытие-порта-опционально)
1. [Конфигурация сервера](#-5-конфигурация-сервера)
1. [Включение IP Forwarding](#-6-включение-ip-forwarding)
1. [Запуск VPN](#-7-запуск-vpn)


-----

## 🔌 Подключение к серверу

```bash
ssh -i ~/ssh-key.key ubuntu@IP_адрес_VPS
```

-----

## ⚙️ 1. Установка WireGuard

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wireguard -y
```

-----

## 📁 2. Создание папок и генерация ключей

```bash
# Переходим в режим root
sudo -i
cd /etc/wireguard

# umask 077 — ограничивает права доступа к создаваемым файлам
umask 077


# Генерируем ключи сервера и клиента
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

# Pre-Shared Key (PSK) — дополнительный ключ для усиленной защиты (опционально)
wg genpsk
```

-----

## 🔍 3. Просмотр ключей

```bash
# Краткий список всех сетевых интерфейсов (важно для настройки маршрутизации)
ip -br link

# Ключи сервера
cat /etc/wireguard/server_private.key
cat /etc/wireguard/server_public.key

# Ключи клиента
cat /etc/wireguard/client_private.key
cat /etc/wireguard/client_public.key
```

-----

## 🔓 4. Открытие порта (только для Варианта Б)

> 💡 При использовании **Варианта А (рекомендуемого)** этот шаг пропустите — порт открывается и закрывается автоматически через PostUp/PostDown.

```bash
# Выполнять только если выбран Вариант Б!
iptables -I INPUT -p udp --dport 51820 -j ACCEPT
```

-----

## 📝 5. Конфигурация сервера

```bash
nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
Address    = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <ВСТАВИТЬ server_private.key>

# =============================================================
# Выберите ОДИН из двух вариантов правил маршрутизации:
# =============================================================

# ✅ Вариант А — РЕКОМЕНДУЕТСЯ (порт управляется автоматически, шаг 4 не нужен)
# Порт открывается при старте VPN и закрывается при остановке — ничего не забудешь
PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE; iptables -I INPUT -p udp --dport 51820 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE; iptables -D INPUT -p udp --dport 51820 -j ACCEPT

# Вариант Б — простой (порт нужно открыть вручную на шаге 4)
# PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
# PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

[Peer]
PublicKey  = <ВСТАВИТЬ client_public.key>
AllowedIPs = 10.8.0.2/32
```

> ⚠️ Замените `ens3` на фактическое имя вашего сетевого интерфейса из вывода `ip -br link`.

-----

## 🔀 6. Включение IP Forwarding

```bash
sudo nano /etc/sysctl.conf
```

Найдите строку и раскомментируйте её (удалите `#`):

```
net.ipv4.ip_forward=1
```

Применяем изменения:

```bash
sudo sysctl -p

# Проверка — должна вернуть 1
sysctl net.ipv4.ip_forward
```

-----

## 🚀 7. Запуск VPN

```bash
sudo systemctl enable wg-quick@wg0   # добавить в автозапуск
sudo systemctl start wg-quick@wg0    # запустить соединение
```

-----

## ✅ 8. Проверка

```bash
wg show   # статистика подключения и список пиров
```

-----

## 🛠️ Полезные команды

### Управление сервисом

|Команда                                   |Описание               |
|------------------------------------------|-----------------------|
|`sudo systemctl status wg-quick@wg0`      |Статус сервиса         |
|`sudo systemctl restart wg-quick@wg0`     |Перезапуск соединения  |
|`sudo systemctl stop wg-quick@wg0`        |Остановка соединения   |
|`sudo systemctl enable wg-quick@wg0`      |Добавить в автозапуск  |
|`sudo systemctl disable wg-quick@wg0`     |Убрать из автозапуска  |
|`sudo systemctl reset-failed wg-quick@wg0`|Сбросить ошибку сервиса|

### Диагностика маршрутизации

```bash
ip rule show | grep 100
ip route show table 100
```

-----

## 🌐 Адресация

|Хост   |IP-адрес|
|-------|--------|
|Сервер |10.8.0.1|
|client1|10.8.0.2|
|client2|10.8.0.3|

-----

*Протестировано на Ubuntu 22.04 / 24.04 LTS*
