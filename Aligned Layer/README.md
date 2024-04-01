<div>
<h1 align="left" style="display: flex;"> Aligned Layer Node Setup for Testnet </h1>
<img src="https://i.hizliresim.com/81hxrzx.jpeg"  style="float: right;" width="1080" height="360"></img>
</div>

Resmi Dökümasayon:
>- [Tıkla](https://github.com/yetanotherco/aligned_layer_tendermint/tree/v0.1.0?tab=readme-ov-file#node-setup)

Explorer:
>- [Tıkla](https://explorer.alignedlayer.com/alignedlayer/staking/alignedvaloper1k4ghu260drcw4zuspjdwpfttwa9g2k7j9qhkkt)





### Manuel Kurulum

// İlk Öncelikle Gerekli Updateleri Yüklüyoruz.

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make gcc -y
sudo apt install moreutils
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
curl https://get.ignite.com/cli! | bash
source ~/.bash_profile

~~~


// Go Kurulumu Yapıyoruz

~~~bash
cd $HOME
VER="1.21.6"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
~~~

// Binnary Dosyasını İndiriyoruz

~~~bash
cd $HOME
wget https://github.com/yetanotherco/aligned_layer_tendermint/archive/refs/tags/v0.1.0.tar.gz
tar -xzf $HOME/v0.1.0.tar.gz
cd $HOME/aligned_layer_tendermint-0.1.0

~~~

/// Moniker İsminizi Girin
~~~
bash setup_node.sh <monikeradi>
~~~


/// Ve Servis Dosyası Oluşturuyoruz

~~~bash
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null << EOF
[Unit]
Description=AlignedLayerd Node Service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which alignedlayerd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
~~~

/// Servis Dosyasını Aktif Edip Çalıştırıyoruz

~~~bash
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd.service
sudo systemctl start alignedlayerd.service
sudo journalctl -u alignedlayerd.service -f --no-hostname -o cat
~~~

/// Wallet Oluşturma
Mnemonic Kelimelerinizi Güvenli Bir Yere Kaydedin Kaybetmeyin

~~~bash
alignedlayerd keys add <key adi>
~~~

/// Eskiden Wallet Oluşturduysanız Recovery Edebilirsiniz

~~~bash
alignedlayerd keys add <key adi> --recover
~~~

/// Faucet Sayfası
~~~bash
https://faucet.alignedlayer.com/
~~~



/// Sync Kontrol Edelim False ise Validator Oluşturabiliriz

~~~bash
alignedlayerd status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'

bash setup_validator.sh <key adi> 1050000stake
~~~

/// Addrbook İndirin

/// Delegate İçin

~~~bash
alignedlayerd tx staking delegate $(alignedlayerd keys show <keyadiniz> --bech val -a) 1050000stake --from <keyadi> --chain-id alignedlayer --gas-prices 0.1stake --gas-adjustment 1.5 --gas auto -y
~~~  



/// Servis Komutları
Log Kontrol

~~~bash
sudo journalctl -u alignedlayerd -f
~~~

/// Servis Durdurma

~~~bash
sudo systemctl stop alignedlayerd
~~~

/// Servis Başlatma

~~~bash
sudo systemctl start alignedlayerd
~~~

/// Servis Restart

~~~bash
sudo systemctl restart alignedlayerd
~~~


/// Cüzdan Miktar Kontrol

~~~bash
alignedlayerd query bank balances cüzdanadresiniz
~~~


/// Node Silme

~~~bash
sudo systemctl stop alignedlayerd
sudo systemctl disable alignedlayerd
sudo rm -rf /etc/systemd/system/alignedlayerd*
sudo rm $(which alignedlayerd)
sudo rm -rf $HOME/.alignedlayerd
sudo rm -fr $HOME/alignedlayerd
~~~

