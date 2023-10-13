# Tangle-Network

#### сайт - https://www.tangle.tools/
#### дискорд - https://discord.gg/ceSSxQUchr
#### документация https://docs.webb.tools/docs/ecosystem-roles/validator/quickstart/
#### кошелек https://polkadot.js.org/apps/?rpc=wss%253A%252F%252Frpc.tangle.tools&ref=blog.webb.tools#/staking
#### эксплоер https://explorer.tangle.tools/?ref=blog.webb.tools
#### телеметрия https://telemetry.polkadot.io/#list/0xea63e6ac7da8699520af7fb540470d63e48eccb33f7273d2e21a935685bf1320

### Tangle - это сеть, совместимая с Ethereum и основанная на Substrate, которая поддерживает как инструменты Substrate, так и Ethereum для разработки


### В настоящее время запущена тестовая сеть Tangle - https://blog.webb.tools/announcing-the-tangle-network-testnet-launch/

Подготовка сервера

```
apt update && apt upgrade -y
```
```
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev libgmp3-dev tar clang bsdmainutils ncdu unzip llvm libudev-dev make protobuf-compiler -y
```

# Установка Ubuntu 22.04 !!!!!!!!!!!!!!

Скачиваем бинарный файл

```
mkdir -p $HOME/.tangle && cd $HOME/.tangle
```

```
wget -O tangle https://github.com/webb-tools/tangle/releases/download/v0.4.7/tangle-standalone-linux-amd64
```
```
chmod 744 tangle
```
```
mv tangle /usr/bin/
```
```
tangle --version
```
#### смотрим версию  tangle 0.4.5-b932226-x86_64-linux-gnu

Скачиваем json

```
wget -O $HOME/.tangle/tangle-standalone.json "https://raw.githubusercontent.com/webb-tools/tangle/main/chainspecs/testnet/tangle-standalone.json"
```
```
chmod 744 ~/.tangle/tangle-standalone.json
```
#### Проверим json
```
sha256sum ~/.tangle/tangle-standalone.json
```
#### 17f4e994b5724242a2299bffeb12c7945ab569577c8f730686e0497e7ce160e4

Создаем сервисный файл

```
yourname=задаем_имя_ноды_вместо_этого_текста
```
```
tee /etc/systemd/system/tangle.service > /dev/null << EOF
[Unit]
Description=Tangle Validator Node
After=network-online.target
StartLimitIntervalSec=0
[Service]
User=$USER
Restart=always
RestartSec=3
LimitNOFILE=65535
ExecStart=/usr/bin/tangle \
  --base-path $HOME/.tangle/data/ \
  --name '$yourname' \
  --chain $HOME/.tangle/tangle-standalone.json \
  --node-key-file "$HOME/.tangle/node-key" \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9615 \
  --auto-insert-keys \
  --validator \
  --telemetry-url "wss://telemetry.polkadot.io/submit 0" \
  --no-mdns
[Install]
WantedBy=multi-user.target
EOF
```

```
systemctl daemon-reload
systemctl enable tangle
systemctl restart tangle && journalctl -u tangle -f -o cat
```

Теперь наша нода начала синхронизироваться. Мы можем проверить нашу ноду в телеметрии
https://telemetry.polkadot.io/#list/0xea63e6ac7da8699520af7fb540470d63e48eccb33f7273d2e21a935685bf1320


# Ожидаем пока наша нода засинхронизируется и продолжаем

После того как нода синхронизировалась вытаскиваем ключ из нашей ноды введя команду

```
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```


``
Если получили подобный результат, то все замечательно {"jsonrpc":"2.0","result":"0xa0very0long0hex0string","id":1} - копируем ключ (выделено жирным) он нам понадобится в ближайшее время
``

# ВАЖНО - сохраняем ключи расположенные в 

``
$HOME/.tangle/node-key 
``

и

``
 $HOME/.tangle/data/chains/tangle-standalone-testnet/keystore/
``

создание кошелька
```
https://chrome.google.com/webstore/detail/polkadot%7Bjs%7D-extension/mopnmbcafieddcagagdcbnhejhlodfdd
```

Далее, Переходим на сайт и для начала создаем кошелек stash https://polkadot.js.org/apps/?rpc=wss%253A%252F%252Frpc.tangle.tools&ref=blog.webb.tools#/staking

Для stash настраиваем Set on-chain Identity для идентификации 

Создаем валидатора. Для этого выбираем Network - Staking - Accounts - Validator

Далее вставляем наш ключ полученный с ноды валидатора, выбираем процент комиссионного вознаграждения

 ## Как только освободится место среди валидаторов, вы появитесь во вкладке Staking Overview, а пока можете себя найти на вкладке Waiting

# Перенос валидатора

В отличии от стандартных нод Polkadot в Tangle валидатор также является центром DKG - это означает, что придется продолжать использовать те же ключи,  которые были созданы ранее на первом сервере, иначе валидатор не сможет подписывать блоки

1. Охлаждаем (chill) валидатора в polkadot js и останавливаем ноду на старом сервере. Если ранее Вы не копировали ключи, то самое время сделать это сейчас

2. Запускаем ноду на новом сервере как обычно и полностью синхронизируемся

3. После синхронизации на новом сервере останавливаем ноду, заменяем ключи в $HOME/.tangle/data/chains/tangle-standalone-testnet/keystore/ и включаем ноду


# Полезные команды

### просмотреть логи
```
journalctl -u tangle -f -o cat
```

### удалить ноду

```
systemctl stop tangle && \
systemctl disable tangle && \
rm /etc/systemd/system/tangle.service && \
systemctl daemon-reload && \
cd \
rm -r .tangle
```
