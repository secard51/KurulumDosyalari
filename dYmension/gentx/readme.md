<p align="center">
  <img height="" height="auto" src="https://pbs.twimg.com/media/Fn5MX4BWYAUKt8e.png">
</p>

# dYmension Testnet İçin Gentx Oluşturma

## Ön Ayarlar
Aşağıdaki Kısıma Validator İsmimizin Ne Olmasını İstiyorsak Onu Yazıyoruz
```
NODENAME=<Senin_Validator_İsmin>
```

Sistem Değişkenlerini Kaydetme (Sıra İle Satırları Kopyalayıp Yapıştırın)
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=35-C" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Paket Güncellemeleri 
```
sudo apt update && sudo apt upgrade -y
```

## Essential Yüklemeleri
```
sudo apt-get install make build-essential gcc git jq chrony -y
```

## Go Yüklemesi 
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```

## dYmension Dosyalarının İndirilip Yüklenmesi
```
cd $HOME
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v0.2.0-beta
make install
```

## App Yapılandırmaları
```
dymd config chain-id $CHAIN_ID
dymd config keyring-backend test
```

## Nodu İçeri Alma
```
dymd init $NODENAME --chain-id $CHAIN_ID
```

## Yeni Cüzdan Oluşturulması veya Mevcut Cüzdanı İçeri Alma
Yeni Cüzdan Oluşturma
```
dymd keys add "Wallet İsminiz"
```

Mevcut Cüzdanı İçeri Alma
```
dymd keys add "Wallet İsminiz" --recover
```

## Genesis Ekleme
```
WALLET_ADDRESS=$(dymd keys show $WALLET -a)
dymd add-genesis-account $WALLET_ADDRESS 600000000000udym
```

## Gentx Oluşturma 
```
dymd gentx "Wallet İsminiz" 500000000000udym \
--chain-id $CHAIN_ID \
--moniker=$NODENAME \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--identity="" \
--website="" \
--details="" \
--min-self-delegation=1
```

## UNUTMAYIN
- Yeni Bir Cüzdan Oluşturdu İseniz Çıktıda Verilen Gizli Kelimelerinizi Güvende Tutun 
- Gentx Dosyanızıda Local Bilgisayarınıza Kaydetmeyi Unutmayın ( Gentx Oluşturduğunuzda Kaydettiği Yeri Belirtecektir )

## Gentx GitHuba Yükleme Ve Pr Oluşturma
1. Gentx Dosyanızı Bu Adresten Kopyalayın  ${HOME}/.dymension/config/gentx/gentx-XXXXXXXX.json.
2. Githuba Girerek Linkini Verdiğim Kısımdan Projeyi Klonlayın https://github.com/dymensionXYZ/testnets
3. Dosyaya Validator İsminizi Ekleyin `gentx-<VALIDATOR_NAME>.json` daha sonra `dymension-hub/35-C/gentx` Forkladığınız Dosyada Bulunan Bölüme İsmini Değiştirdiğiniz Dosyayı Upload Ediniz
4. Yeni Bir Pr Oluştunuz 

