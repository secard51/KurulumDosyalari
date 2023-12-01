 <h1 align="center">Setup Guide</h1>
 
 ### Recommended System Requirements
- CPU: 4 cores
- RAM: 16GB RAM 
- SSD: 300GB
- İşletim Sistemi: Ubuntu 20.04LTS

### Installation Steps

## Updates
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y && cd $HOME
```

## Go Install
```
ver="1.20.3"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## C4e Binary
- After completing the first 3 codes regularly, make sure to check the version.
```
git clone --depth 1 --branch  v1.2.1  https://github.com/chain4energy/c4e-chain.git
cd c4e-chain/
make install
```
```
c4ed version
output: 1.2.1
```

## C4e Config Settings
- In these steps, we continue the network and operation to ensure that the node runs smoothly. First, we make our Network and Moniker entries. 
```
export CHAINID=perun-1
export MONIKER=Your Own Name Your Name
```

- Inıt Process
```
c4ed init $MONIKER --chain-id $CHAINID
rm -rf ~/.c4e-chain/config/genesis.json
```

- Genesis Transport
```
git clone https://github.com/chain4energy/c4e-chains.git
cd c4e-chains/$CHAINID
cp genesis.json ~/.c4e-chain/config/
```

- Config Settings
```
peers="5a0e2f1a0541ba34cab61f64b62faaeb4113f2df@65.108.6.45:61056"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
```

- He has to do the pruning operations. If you want your node to take up less space, you can make the following settings.
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.c4e-chain/config/app.toml
```

- Creating a Service
```
sudo tee /etc/systemd/system/c4ed.service > /dev/null <<EOF
[Unit]
Description=c4e
After=network-online.target

[Service]
User=$USER
ExecStart=$(which c4ed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- SnapShot will start your node from the closest location to the network according to the time it was taken, it is not mandatory. It allows the system to sync faster. It was published here among the snapshots in which Obojay was released.
```
cd $HOME
apt install lz4
sudo systemctl stop c4ed
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
rm -rf $HOME/.c4e-chain/data
curl -o - -L http://c4e.snapshot.stavr.tech:1018/c4e/c4e-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.c4e-chain --strip-components 2
mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json
```

- Starting the Node
```
sudo systemctl daemon-reload
sudo systemctl enable c4ed
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

- Creating a Wallet
    - Calculating New Wallet, Importing Old Wallet
```
c4ed keys add cuzdanismi
```
```
c4ed keys add cuzdanismi --recover
```

- We check whether the node's system is equal or not with this code. If it gives the wrong result when you enter the code, the node system is synchronized and you can perform the step of creating a validator.
```
c4ed status 2>&1 | jq .SyncInfo
```

- Creating an Authenticator
```
c4ed tx staking create-validator \
  --amount 1000000uc4e \
  --from walletismi \
  --moniker $MONIKER \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --chain-id perun-1 \
  --identity="" \
  --details="" \
  --website=""
```


