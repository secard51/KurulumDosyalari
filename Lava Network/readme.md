# LAVA Testnet Kurulum Rehberi
![lava](https://lavanet.xyz/assets/banner.png)
[Resmi WebSite](https://lavanet.xyz) \
[Resmi GitHub](https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git)
=
[EXPLORER](https://explorer.secardnode.com/lava%20network/staking)
=
[ADDRBOOK](https://github.com/secard51/KurulumDosyalari/blob/main/Lava%20Network/addrbook.json)
=
- **Minimum Sistem Gereksinimi**:

| Node Durum |İşlemci | RAM  | Harddisk  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |



# 2) Manuel Kurulum ( Kodları Tek Tek Kopyalayıp Yapıştırın)

### Başlamadan Önce Sistem Gücellemeleri Ve Go 1.19 Yüklemesi

## Update

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## Go Kurulum
```wget https://go.dev/dl/go1.19.linux-amd64.tar.gz \
&& sudo tar -xvf go1.19.linux-amd64.tar.gz && sudo mv go /usr/local \
&& echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile \
&& source ~/.bash_profile
```
## Go Version Kontrol
```python
go version
# go version go1.19 linux/amd64 olmalıdır
```
## Kurulum (Kurulumu Yaptıktan Sonra Update Yapacağız)
```python
cd $HOME
git clone https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.3.0/lavad
chmod +x lavad
mv lavad $HOME/go/bin/
```
*******🟢UPDATE ( GÜNCELLE )🟢******* 04.01.23

```python
cd $HOME
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.4.0/lavad
chmod +x lavad
mv lavad $HOME/go/bin/
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
`lavad version --long | head`
- version: 0.4.0-rc2-e2c69db
- commit: 

*******🟢UPDATE ( GÜNCELLE )🟢******* v0.4.3-5673a81  22.300 BLOKTAN SONRA BU GÜNCELLEME YAPILACAK
```python
cd $HOME
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.4.3/lavad
chmod +x lavad
mv lavad $HOME/go/bin/
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
## Moniker Adınız Yerine Kendi Moniker İsminizi Yazın
```python
lavad init "MonikerAdınız" --chain-id lava-testnet-1
lavad config chain-id lava-testnet-1
```    
## Oluşturma/Geri Yükleme Cüzdan ( Oluşturduğunuzda Size Vereceği Gizli Kelimeleri Not Etmeyi Unutmayın)
```python
lavad keys add <Cüzdanİsminiz>
      OR
lavad keys add <Cüzdanİsminiz> --recover
```

## Genesis İndirin
```python
cp $HOME/GHFkqmTzpdNLDd6T/testnet-1/genesis_json/genesis.json $HOME/.lava/config
```
`sha256sum $HOME/.lava/config/genesis.json`
+ 72170a8a7314cb79bc57a60c1b920e26457769667ce5c2ff0595b342c0080d78
## Genel Chain Ayarlarını Yapın
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ulava\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.lava/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.lava/config/config.toml
peers="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lava/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.lava/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.lava/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.lava/config/config.toml

```
## StateSync Opsiyonel Hızlı Sync Olmak İçin (UTSA'dan Alıntıdır)
```python
SNAP_RPC=https://t-lava.rpc.utsa.tech:443
peers="433be6210ad6350bebebad68ec50d3e0d90cb305@217.13.223.167:60856"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.lava/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lava/config/config.toml
lavad tendermint unsafe-reset-all --home /root/.lava --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.lava/config/app.toml
systemctl restart lavad && journalctl -u lavad -f -o cat
```

# Servis Dosyası Oluşturun
```
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=lava
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Başlatın
```python
sudo systemctl daemon-reload
sudo systemctl enable lavad
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
### Validator Oluşturma
```
lavad tx staking create-validator \
  --amount 1000000ulava \
  --from <CüzanİsminiziYazın> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(lavad tendermint show-validator) \
  --moniker MonikerİsminiziYazın \
  --chain-id lava-testnet-1 \
  --identity="" \
  --details="" \
  --website="" -y
```
### Sync Durumu
```python
lavad status 2>&1 | jq .SyncInfo
```
### Node Durumu
```python
lavad status 2>&1 | jq .NodeInfo
```
### Node Logları Kontrol Etme
```python
sudo journalctl -u lavad -f -o cat
```
### Cüzdan Token Miktarı Kontrol Etme 
```python
lavad query bank balances lava...address1yjgn7z09ua9vms259j
```

## Node Silme
```bash
sudo systemctl stop lavad && \
sudo systemctl disable lavad && \
rm /etc/systemd/system/lavad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf GHFkqmTzpdNLDd6T && \
rm -rf .lava && \
rm -rf $(which lavad)
```
