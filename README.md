# BitBadges Node Kurulum Rehberi

## Sistem Güncelleme ve Gerekli Araçların Kurulumu

```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

## Go Versiyon Kontrolü ve Kurulumu

**ÖNEMLİ:** Önce mevcut Go versiyonunuzu kontrol edin:

```bash
go version
```

### Seçenek 1: Mevcut Go Versiyonu Uygunsa (1.21 veya üzeri)

Eğer sisteminizde Go 1.21 veya daha üst versiyonu varsa, Go kurulumunu atlayabilirsiniz. Doğrudan **Node Kurulumu** bölümüne geçin.

### Seçenek 2: Go Versiyonu Güncellenmeli

**⚠️ UYARI:** Aşağıdaki komutlar mevcut Go kurulumunu ve $HOME/go dizinindeki TÜM dosyaları silecektir! 

**Devam etmeden önce:**
1. `$HOME/go` dizininde başka projelere ait dosyalar varsa YEDEK ALIN
2. Diğer projelerin binary dosyalarını başka bir yere kopyalayın
3. Emin olmadığınız dosyaları silmeyin

```bash
# Yedekleme örneği (ihtiyacınıza göre düzenleyin)
mkdir -p $HOME/go_backup
cp -r $HOME/go/bin $HOME/go_backup/

# Eski Go'yu temizleme
rm -rf $HOME/go
sudo rm -rf /usr/local/go

# Yeni Go'yu kurma
cd $HOME
curl https://dl.google.com/go/go1.24.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -

# Ortam değişkenlerini ayarlama (.profile dosyasında zaten varsa tekrar eklemeyin)
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile
go version
```

### Seçenek 3: Alternatif - Go Version Manager (GVM) Kullanımı

Birden fazla Go versiyonu kullanmanız gerekiyorsa GVM önerilir:

```bash
# GVM kurulumu
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source ~/.gvm/scripts/gvm

# Go 1.24.5 kurulumu
gvm install go1.24.5 -B
gvm use go1.24.5 --default
```

## Node Kurulumu

**Not:** Eğer diğer Cosmos SDK tabanlı projeler için binary dosyalarınız `$HOME/go/bin` dizinindeyse, bu adımlar onları etkilemez. Ancak yine de önemli binary dosyalarınızı yedeklemeniz önerilir.

```bash
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v16
make build-linux/amd64

# Binary dosyasını kopyalama
mv build/bitbadgeschain-linux-amd64 $HOME/go/bin/bitbadgeschaind
chmod +x $HOME/go/bin/bitbadgeschaind
bitbadgeschaind version
```

## Cosmovisor Kurulumu

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest

# Cosmovisor dizinlerini oluştur
mkdir -p $HOME/.bitbadgeschain/cosmovisor/genesis/bin
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades

# Binary dosyasını kopyala
cp $HOME/go/bin/bitbadgeschaind $HOME/.bitbadgeschain/cosmovisor/genesis/bin/
```

## Node Başlatma

Moniker adınızı belirleyin (NodeName yerine kendi validator adınızı yazın):

```bash
bitbadgeschaind init NodeName --chain-id=bitbadges-1
```

## Genesis ve Addrbook İndirme

```bash
curl -Ls https://ss.bitbadges.nodestake.org/genesis.json > $HOME/.bitbadgeschain/config/genesis.json
curl -Ls https://ss.bitbadges.nodestake.org/addrbook.json > $HOME/.bitbadgeschain/config/addrbook.json
```

## Port Ayarları (Özel Port Kullanımı)

Varsayılan portlar yerine özel portlar kullanmak için aşağıdaki komutları çalıştırın. Bu örnekte 26 yerine 56 kullanılmıştır, siz kendi port numaranızı seçebilirsiniz:

```bash
# Port değişkeni tanımla (26 yerine istediğiniz iki haneli sayıyı yazın)
PORT=56

# app.toml dosyasında port ayarları
sed -i.bak -e "s%:1317%:${PORT}317%g;
s%:8080%:${PORT}080%g;
s%:9090%:${PORT}090%g;
s%:9091%:${PORT}091%g;
s%:8545%:${PORT}545%g;
s%:8546%:${PORT}546%g;
s%:6065%:${PORT}065%g" $HOME/.bitbadgeschain/config/app.toml

# config.toml dosyasında port ayarları
sed -i.bak -e "s%:26658%:${PORT}658%g;
s%:26657%:${PORT}657%g;
s%:6060%:${PORT}060%g;
s%:26656%:${PORT}656%g;
s%:26660%:${PORT}660%g" $HOME/.bitbadgeschain/config/config.toml
```

## Pruning Ayarları (Opsiyonel)

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.bitbadgeschain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.bitbadgeschain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.bitbadgeschain/config/app.toml
```

## Minimum Gas Price Ayarı

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.025ubadge"|g' $HOME/.bitbadgeschain/config/app.toml
```

## Prometheus Aktifleştirme

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.bitbadgeschain/config/config.toml
```

## Indexer Kapatma (Opsiyonel - Disk Alanı Tasarrufu)

```bash
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.bitbadgeschain/config/config.toml
```

## Servis Dosyası Oluşturma (Cosmovisor ile)

```bash
sudo tee /etc/systemd/system/bitbadgeschaind.service > /dev/null <<EOF
[Unit]
Description=BitBadges Node with Cosmovisor
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=always
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=bitbadgeschaind"
Environment="DAEMON_HOME=$HOME/.bitbadgeschain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable bitbadgeschaind
```

## Snapshot İndirme (Opsiyonel - Hızlı Senkronizasyon)

```bash
bitbadgeschaind tendermint unsafe-reset-all --home $HOME/.bitbadgeschain --keep-addr-book

SNAP_NAME=$(curl -s https://ss.bitbadges.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.bitbadges.nodestake.org/${SNAP_NAME} | lz4 -c -d - | tar -x -C $HOME/.bitbadgeschain
```

## Node'u Başlatma

```bash
sudo systemctl restart bitbadgeschaind
```

## Logları Takip Etme

```bash
journalctl -u bitbadgeschaind -f
```

## Senkronizasyon Durumunu Kontrol Etme

```bash
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
```

`false` sonucu alırsanız node senkronize olmuştur.

---

# Cüzdan İşlemleri

## Yeni Cüzdan Oluşturma

```bash
bitbadgeschaind keys add cüzdan-adı
```

## Mevcut Cüzdanı İçe Aktarma (Mnemonic ile)

```bash
bitbadgeschaind keys add cüzdan-adı --recover
```

## Cüzdanları Listeleme

```bash
bitbadgeschaind keys list
```

## Cüzdan Bakiyesi Kontrol Etme

```bash
bitbadgeschaind query bank balances $(bitbadgeschaind keys show cüzdan-adı -a)
```

## Cüzdan Adresini Görüntüleme

```bash
bitbadgeschaind keys show cüzdan-adı -a
```

---

# Validator İşlemleri

## Validator Oluşturma

Node'unuzun tamamen senkronize olduğundan emin olun ve cüzdanınızda yeterli token olduğunu kontrol edin.

```bash
bitbadgeschaind tx staking create-validator \
  --amount 1000000ubadge \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.20" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey=$(bitbadgeschaind tendermint show-validator) \
  --moniker "VALIDATOR-ADINIZ" \
  --website "https://websiteadresiniz.com" \
  --identity "KEYBASE-ID" \
  --details "Validator açıklamanız" \
  --security-contact="email@adresiniz.com" \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

**Not:** 
- `--moniker`: Validator adınız
- `--website`: Web siteniz (opsiyonel)
- `--identity`: Keybase ID'niz (opsiyonel, keybase.io'dan alabilirsiniz)
- `--details`: Validator hakkında açıklama
- `--from`: Cüzdan adınız

## Validator Düzenleme

```bash
bitbadgeschaind tx staking edit-validator \
  --new-moniker "YENİ-VALIDATOR-ADI" \
  --website "https://yeniwebsite.com" \
  --identity "YENİ-KEYBASE-ID" \
  --details "Yeni açıklama" \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Validator Bilgilerini Görüntüleme

```bash
bitbadgeschaind query staking validator $(bitbadgeschaind keys show cüzdan-adı --bech val -a)
```

## Jailed Durumundan Çıkma (Unjail)

```bash
bitbadgeschaind tx slashing unjail \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

---

# Yararlı Komutlar

## Token Gönderme

```bash
bitbadgeschaind tx bank send cüzdan-adı ALICI-ADRES 1000000ubadge \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Delegate (Stake) Etme

```bash
bitbadgeschaind tx staking delegate $(bitbadgeschaind keys show cüzdan-adı --bech val -a) 1000000ubadge \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Redelegate (Başka Validator'a Stake Taşıma)

```bash
bitbadgeschaind tx staking redelegate $(bitbadgeschaind keys show cüzdan-adı --bech val -a) HEDEF-VALIDATOR-ADRES 1000000ubadge \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Unstake (Stake Geri Çekme)

```bash
bitbadgeschaind tx staking unbond $(bitbadgeschaind keys show cüzdan-adı --bech val -a) 1000000ubadge \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Ödülleri Çekme (Komisyon Dahil)

```bash
bitbadgeschaind tx distribution withdraw-rewards $(bitbadgeschaind keys show cüzdan-adı --bech val -a) \
  --commission \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

## Tüm Validator'lardan Ödülleri Çekme

```bash
bitbadgeschaind tx distribution withdraw-all-rewards \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

---

# Governance İşlemleri

## Proposal Listesini Görüntüleme

```bash
bitbadgeschaind query gov proposals
```

## Proposal Detaylarını Görüntüleme

```bash
bitbadgeschaind query gov proposal PROPOSAL-NUMARASI
```

## Oy Kullanma

Seçenekler: `yes`, `no`, `no_with_veto`, `abstain`

```bash
bitbadgeschaind tx gov vote PROPOSAL-NUMARASI yes \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

---

# Node Yönetimi

## Servisi Durdurma

```bash
sudo systemctl stop bitbadgeschaind
```

## Servisi Başlatma

```bash
sudo systemctl start bitbadgeschaind
```

## Servisi Yeniden Başlatma

```bash
sudo systemctl restart bitbadgeschaind
```

## Log Takibi

```bash
journalctl -u bitbadgeschaind -f
```

## Node Silme

```bash
sudo systemctl stop bitbadgeschaind
sudo systemctl disable bitbadgeschaind
sudo rm /etc/systemd/system/bitbadgeschaind.service
sudo systemctl daemon-reload
rm -rf $HOME/.bitbadgeschain
rm -rf $HOME/bitbadgeschain
```

---

## Faydalı Linkler

- **Discord**: [BitBadges Discord](https://discord.gg/bitbadges)
- **Website**: [BitBadges.io](https://bitbadges.io)
- **Explorer**: [BitBadges Explorer](https://explorer.bitbadges.io)
- **GitHub**: [BitBadges GitHub](https://github.com/BitBadges)

---

**Not:** Komutlardaki `cüzdan-adı` kısmını kendi cüzdan adınızla değiştirmeyi unutmayın.
