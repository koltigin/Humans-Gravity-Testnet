# Humans Gravity Testnet Kurulum Rehberi

[Kaynak](https://docs.humans.zone) / [Explorer](https://explorer.humans.zone)
![Humans-GitHub](https://user-images.githubusercontent.com/102043225/218226260-8b425422-4ba6-4fa6-9e58-9deac0edd891.jpg)

## Bağlantılar
 ✔️ [Website](https://humans.ai/)<br>
 ✔️ [Blockchain Explorer](https://explorer.humans.zone)<br>
 ✔️ [Doküman](https://docs.humans.zone)<br>
 ✔️ [Discord](https://discord.gg/humansdotai)<br>
 ✔️ [S.S.S.](https://docs.humans.zone/learn/faq.html)
 

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | 8 |
| RAM	| 8 GB | 32 GB |
| Storage	| 100 GB SSD | 200 GB SSD |

## Sistemi Güncelleme
```shell
apt update && apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$HUMAN_NODENAME` validator adınız
* `$HUMAN_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export HUMAN_NODENAME=$HUMAN_NODENAME"  >> $HOME/.bash_profile
echo "export HUMAN_WALLET=$HUMAN_WALLET" >> $HOME/.bash_profile
echo "export HUMAN_PORT=11" >> $HOME/.bash_profile
echo "export HUMAN_CHAIN_ID=testnet-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export HUMAN_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export HUMAN_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export HUMAN_PORT=18" >> $HOME/.bash_profile
echo "export HUMAN_CHAIN_ID=Testnet3" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Humans'ın Kurulması

```shell
git clone https://github.com/humansdotai/humans
cd humans || return
git checkout v1.0.0
go build -o humansd cmd/humansd/main.go
cp humansd /usr/local/bin/humansd
humansd version
```

## Uygulamayı Yapılandırma ve Başlatma
```shell
humansd config keyring-backend test
humansd config chain-id $HUMAN_CHAIN_ID
humansd init --chain-id $HUMAN_CHAIN_ID $HUMAN_NODENAME
```

## Genesis Dosyasının Kopyalanması
```shell
curl -s https://rpc-testnet.humans.zone/genesis | jq -r .result.genesis > $HOME/.humans/config/genesis.json
curl -s https://snapshots-testnet.nodejumper.io/humans-testnet/addrbook.json > $HOME/.humans/config/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025uheart"|g' $HOME/.humans/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.humans/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.humans/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.humans/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${HUMAN_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${HUMAN_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${HUMAN_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${HUMAN_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${HUMAN_PORT}660\"%" $HOME/.humans/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${HUMAN_PORT}317\"%; s%^address = \":8080\"%address = \":${HUMAN_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${HUMAN_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${HUMAN_PORT}091\"%" $HOME/.humans/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${HUMAN_PORT}657\"%" $HOME/.humans/config/client.toml
```

## Servis Dosyası Oluşturma
```shell
mv $HOME/go/bin/humansd /usr/bin/
tee <<EOF >/dev/null /etc/systemd/system/humansd.service
[Unit]
Description=Humans AI Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```

## Snapshot Yükleme
```shell
humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book

cp $HOME/.humans/data/priv_validator_state.json $HOME/.humans/priv_validator_state.json.backup 

humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book 
curl https://snapshots-testnet.nodejumper.io/humans-testnet/testnet-1_2023-02-10.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.humans

mv $HOME/.humans/priv_validator_state.json.backup $HOME/.humans/data/priv_validator_state.json

```

## Servisi Başlatma
```shell
systemctl daemon-reload
systemctl enable humansd
systemctl restart humansd
```

## Logları Kontrol Etme
```shell
journalctl -u humansd -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$HUMAN_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
humansd keys add $HUMAN_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
humansd keys add $HUMAN_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
HUMAN_WALLET_ADDRESS=$(humansd keys show $HUMAN_WALLET -a)
HUMAN_VALOPER_ADDRESS=$(humansd keys show $HUMAN_WALLET --bech val -a)
```

```shell
echo 'export HUMAN_WALLET_ADDRESS='${HUMAN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HUMAN_VALOPER_ADDRESS='${HUMAN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Faucet
[humans](https://discord.gg/humansdotai) adresine giderek `#testnet-faucet` kanalından `$request cuzdan-adresi` şeklinde mesaj atarak token istiyoruz. 

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almadıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
humansd status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**


## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlirttiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
humansd tx staking create-validator \
--amount=9500000uheart \
--pubkey=$(humansd tendermint show-validator) \
--moniker=$HUMAN_NODENAME \
--chain-id=$HUMAN_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation="1" \
--gas-prices=0.1uheart \
--gas-adjustment=1.5 \
--gas=auto \
--from=$HUMAN_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
```

## Validator Rolü Alma
[Discord'da](https://discord.gg/humansdotai) `#create-ticket` sayfasına giderek `Validator Role Request` butonuna tıklayarak validatorumuze ait exlporer linkini gönderiyoruz.
![image](https://user-images.githubusercontent.com/102043225/218235056-78d89f04-f45d-42e0-b630-08f91c182f7a.png)
![image](https://user-images.githubusercontent.com/102043225/218235208-814755ef-2b91-4a45-9d4c-d18e0646eb7d.png)

## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu humansd -o cat
```

### Sistemi Başlatma

```
systemctl start humansd
```

### Sistemi Durdurma
```
systemctl stop humansd
```

### Sistemi Yeniden Başlatma
```
systemctl restart humansd
```

### Node Senkronizasyon Durumu
```
humansd status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
humansd status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
humansd status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
humansd tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
humansd keys list
```

### Cüzdan Adresini Görme

```
humansd keys show $HUMAN_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
humansd keys add $HUMAN_WALLET --recover
```

### Cüzdanı Silme

```
humansd keys delete $HUMAN_WALLET
```

### Cüzdan Bakiyesine Bakma

```
humansd query bank balances $HUMAN_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
humansd tx bank send $HUMAN_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000uheart
```

### Proposal Oylamasına Katılma
```
humansd tx gov vote 1 yes --from $HUMAN_WALLET --chain-id=$HUMAN_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
humansd tx staking delegate $VALOPER_ADDRESS 100000000uheart --from=$HUMAN_WALLET --chain-id=$HUMAN_CHAIN_ID --gas=auto --fees 5000uheart
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
humansd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000uheart --from=$HUMAN_WALLET --chain-id=$HUMAN_CHAIN_ID --gas=auto
```

### Ödülleri Çekme

```
humansd tx distribution withdraw-all-rewards --from=$HUMAN_WALLET --chain-id=$HUMAN_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
humansd tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$HUMAN_WALLET --commission --chain-id=$HUMAN_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
humansd tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$HUMAN_CHAIN_ID \
--from=$HUMAN_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
humansd tx staking edit-validator --commission-rate "0.02" --moniker=$HUMAN_NODENAME --chain-id=$HUMAN_CHAIN_ID --from=$HUMAN_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$HUMAN_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
humansd tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$HUMAN_CHAIN_ID \
--from=$HUMAN_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
humansd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$HUMAN_CHAIN_ID \
  --gas=auto
  --gas-adjustment=1.4
```

### Node'u Tamamen Silme 

```
systemctl stop humansd && \
systemctl disable humansd && \
rm /etc/systemd/system/humansd.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .humans humans && \
rm -rf $(which humansd) \
sed -i '/HUMAN_/d' ~/.bash_profile
```

# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
