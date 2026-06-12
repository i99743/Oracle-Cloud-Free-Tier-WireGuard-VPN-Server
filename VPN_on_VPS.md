# 🌐 Исходящий VPN на VPS — Скрытие адреса сервера

Подключение VPS к внешнему VPN-сервису для маскировки реального IP-адреса сервера. Трафик клиентов WireGuard будет выходить через VPN-провайдера.

---

## 📋 Содержание

1. [Создание конфига VPN](#-1-создание-конфига-vpn)
2. [Запуск VPN](#-2-запуск-vpn)
3. [Проверка](#-3-проверка)
4. [Полезные команды](#-полезные-команды)

---

## 📝 1. Создание конфига VPN

```bash
sudo nano /etc/wireguard/vpn.conf
```

Заполните данными из конфига, полученного от вашего VPN-провайдера:

```ini
[Interface]
PrivateKey = <из конфига провайдера>
Address    = <из конфига провайдера>
DNS        = <из конфига провайдера>
Table      = 100

# Маршрутизация: трафик клиентов (10.8.0.0/24) направляется через VPN
PostUp   = ip rule add from 10.8.0.0/24 table 100; \
           iptables -t nat -A POSTROUTING -o vpn -j MASQUERADE
PostDown = ip rule del from 10.8.0.0/24 table 100; \
           iptables -t nat -D POSTROUTING -o vpn -j MASQUERADE

[Peer]
PublicKey  = <из конфига провайдера>
AllowedIPs = <из конфига провайдера>
Endpoint   = <из конфига провайдера>
```

> ⚠️ Интерфейс `vpn` в правилах `POSTROUTING` должен совпадать с именем конфига (`vpn.conf` → интерфейс `vpn`).

---

## 🚀 2. Запуск VPN

```bash
sudo systemctl enable wg-quick@vpn   # добавить в автозапуск
sudo systemctl start wg-quick@vpn    # запустить соединение
```

---

## ✅ 3. Проверка

```bash
wg show   # статистика подключения и список пиров
```

---

## 🛠️ Полезные команды

| Команда | Описание |
|---|---|
| `sudo systemctl status wg-quick@vpn` | Статус сервиса |
| `sudo systemctl restart wg-quick@vpn` | Перезапуск соединения |
| `sudo systemctl stop wg-quick@vpn` | Остановка соединения |
| `sudo systemctl enable wg-quick@vpn` | Добавить в автозапуск |
| `sudo systemctl disable wg-quick@vpn` | Убрать из автозапуска |
| `sudo systemctl reset-failed wg-quick@vpn` | Сбросить ошибку сервиса |

---

*Используется совместно с [WireGuard VPN — Установка и настройка](./README.md)*
