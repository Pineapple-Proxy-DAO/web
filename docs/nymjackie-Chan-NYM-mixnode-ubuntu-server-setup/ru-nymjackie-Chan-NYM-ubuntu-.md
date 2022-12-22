# Nymjackie Chan - установка миксноды NYM на ubuntu server

[EN](../Nymjackie%20Chan%20-%20NYM%20mixnode%20ubuntu%20server%20setup%20903d42371d00423f8ff1855e7d8a8588.md)

### Проверка доступности IPv6 адреса

Прежде чем устанавливать сервер закажите у вашего провайдера дополнительный IPv6 адрес. Чтобы убедиться, что все хорошо, выполните команду отображения всех IP адресов включая IPv6

```bash
hostname -I
```

```bash
194.135.119.150 2a12:e340:2000:7::2
```

Делаем пинг внешнего IPv6 адреса

```bash
ping6 -c 5 ipv6.google.com
```

```bash
PING ipv6.google.com(par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e)) 56 data bytes
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=1 ttl=115 time=72.2 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=2 ttl=115 time=72.1 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=3 ttl=115 time=73.0 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=4 ttl=115 time=72.2 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=5 ttl=115 time=73.3 ms
```

### Добавляем пользователя mixnode

```bash
adduser mixnode
```

Выдаём ему ‘sudo’ привилегии

```bash
usermod -aG sudo mixnode
```

Заходим в учётную запись только что созданного пользователя ‘mixnode’

```bash
su - mixnode
```

### Устанавливаем необходимое ПО

Обновляем системное ПО

```bash
sudo apt-get update
sudo apt-get upgrade
```

Устанавливаем пакеты

```bash
sudo apt-get install pkg-config build-essential libssl-dev curl jq git
```

Устанавливаем rust компилятор. Актуальную ссылку можно взять [тут](https://www.rust-lang.org/tools/install) 

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Перезаходим под текущей учётной записью чтобы стал доступен путь до компилятора cargo в текущей оболочке

### Включаем фаервол

Устанавливаем фаервол ufw если он не был установлен ранее 

```bash
sudo apt-get install ufw
```

Включаем его, по-умолчанию закрыты все входящие соединения

```bash
sudo ufw enable
```

Разрешаем nym-mixnode порты (1789,1790,8000) и ssh (22)

```bash
sudo ufw allow 1789,1790,8000,22/tcp
```

Проверяем статус фаервола

```bash
sudo ufw status
```

### Собираем ноду из исходников

Устанавливаем актуальную версию релиза в команде checkout

```bash
git clone https://github.com/nymtech/nym.git
cd nym
git checkout release/v1.1.3
cargo build --release
```

### Инициализируем ноду

Переходим в директорию с исполняемым файлом nym-mixnode 

```bash
cd target/release
```

Сохраняем адрес кошелька в файл с именем ‘wallet.txt’ и загружаем в текущую директорию, он понадобится в команде ниже. Запускаем инициализацию ноду и сохраняем вывод в файл ‘bonding.txt’, его содержимое пригодится в последующем шаге

```bash
./nym-mixnode init --id pineapple --host $(curl ifconfig.me) --wallet-address $(cat wallet.txt) > bonding.txt
```

Будет создана директория ‘~/.nym’ с конфигом и ключами шифрования  

### Привязываем кошелёк к ноде

Рекомендуется это делать через Nym Wallet. Запустите его и перейдите в Bonding меню.

Шаг 1. Вся необходимая информация сохранилась ранее в файле bonding.txt 

![Screenshot from 2022-12-18 15-59-02.png](Screenshot_from_2022-12-18_15-59-02.png)

Шаг 2. Установите ‘Amount’ который должен быть не менее 100 NYM и ‘Profit margin’ (обычно он в пределах 1-10%).

![Screenshot from 2022-12-18 16-09-38.png](Screenshot_from_2022-12-18_16-09-38.png)

### Запустите ноду сервисом

Сервис позволяет запускать ноду в фоновом режиме и запускать ее автоматически вместе со стартом системы. Загрузите файл ‘/etc/systemd/system/nym-mixnode.service’ с содержимым ниже

```
[Unit]
Description=Nym Mixnode (1.1.3)
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=mixnode
LimitNOFILE=65536
ExecStart=/home/mixnode/nym/target/release/nym-mixnode run --id pineapple
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Запустите сервис

```bash
sudo systemctl enable nym-mixnode.service
sudo service nym-mixnode start
```

### Добавьте описание ноды

Отредактируйте файл ‘~/.nym/mixnodes/pineapple/config/description.toml’ на ваше усмотрение

```toml
name = "Pineapple on the pizza"
description = "We, the supporters of pineapple pizza, declare our resistance to all forms of taste oppression. We refuse to be dictated to by arbitrary prejudices and conventional tastes. We believe that everyone has the right to choose what they want to eat, without fear of reprisal or discrimination."
link = "https://pastenym.ch/#/5F5D49XB&key=c6872aafb31dbdfa8e411d884248a14d"
location = "Pizza, Atop"
```

Перезапустите сервис, чтобы он считал обновлённое описание

```bash
sudo service nym-mixnode restart
```

Проверьте как отображается описание в обозревателе заменив %NODE_ID% с вашим ‘Identity Key’

```bash
https://mixnet.explorers.guru/mixnode/%NODE_ID%
```

### Проверьте корректность лимита открытых файлов

Процессы в линуксе обычно имеют лимит на количество открываемых файлов. Проверьте его командой ниже

```bash
grep -i "open files" /proc/$(ps -A -o pid,cmd|grep nym-mixnode | grep -v grep |head -n 1 | awk '{print $1}')/limits
```

Если в выводе все значения установлены в 65535 то всё хорошо и ничего делать не нужно, в противном случае перейдите к следующему шагу 

```bash
sudo echo DefaultLimitNOFILE=65535 >> /etc/systemd/user.conf
sudo echo DefaultLimitNOFILE=65535 >> /etc/systemd/system.conf
```

Перезагрузите сервер, чтобы изменения вступили в силу

### Проверьте статус ноды

Выполните команду

```bash
sudo journalctl -u nym-mixnode -o cat | grep "Since startup mixed"
```

Количество ‘mixed packets’ не должно быть нулевым и будет увеличиваться со временем

```
2022-12-18T05:23:59.829Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4271 packets! (0 in last 30 seconds)
2022-12-18T05:24:59.831Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4271 packets! (0 in last 30 seconds)
2022-12-18T05:25:59.832Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4283 packets! (12 in last 30 seconds)
2022-12-18T05:26:59.834Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4283 packets! (0 in last 30 seconds)
```