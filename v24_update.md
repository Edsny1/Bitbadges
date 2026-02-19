# BitBadges v24 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v24 Upgrade Bilgileri

- **Upgrade Adı:** v24
- **Upgrade Blok Yüksekliği:** 8890000
- **Tahmini Zaman:** 24 Şubat 2026, 11:19:18 EST (18:19:18 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/31
- **Mevcut Versiyon:** v23
- **Hedef Versiyon:** v24
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v24

---

## 🆕 v24 Upgrade Özeti - MAJOR UPDATE

Bu upgrade **büyük değişiklikler** içerir:

### 🔥 Kaldırılan Özellikler:
- **CosmWASM/WASM modülleri tamamen kaldırıldı**
- **libwasmvm artık gerekli değil**

### ✨ Yeni Özellikler:
- **Tam EVM uyumluluğu** - JSON-RPC API ile
- **ERC20 token desteği**
- **EVM precompiles** eklendi
- **Badges modülü → tokenization olarak yeniden adlandırıldı**
- **Interchain Queries** - Cross-chain token ownership doğrulaması için

### 🌐 EVM Endpoints:
- **Mainnet:** `https://evm-rpc.bitbadges.io` (Chain ID: 50024)
- **Testnet:** `https://evm-rpc-testnet.bitbadges.io` (Chain ID: 50025)

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v23 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v24 Özel Notları:

- **CosmWASM kaldırıldı** - libwasmvm bağımlılığı artık yok
- **EVM desteği eklendi** - Opsiyonel JSON-RPC yapılandırması gerekebilir
- **Modül ismi değişti** - badges → tokenization
- Daha hafif binary (WASM kaldırıldığı için)

---

## 🚀 v24 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v24 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v24 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v24
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v24`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind version --long
```

---

## 🎯 Upgrade Bloğuna Ulaşıldığında Yapılacaklar

### ⏰ Upgrade Bloğunu İzleyin

Başka bir terminal penceresinde sürekli izleyin:

```bash
watch -n 5 'bitbadgeschaind status 2>&1 | jq -r ".SyncInfo.latest_block_height"'
```

Veya logları takip edin:

```bash
journalctl -u bitbadgeschaind -f | grep -i "upgrade\|halt"
```

### 🚨 Upgrade Bloğuna Ulaştığında (Node Durduğunda)

Node'unuz upgrade bloğuna ulaştığında otomatik olarak duracak ve şu hatayı verecek:
```
error during handshake: error on replay: UPGRADE "v24" NEEDED at height: 8890000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v24 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v24`

### Adım 9: Servisi Başlatın

```bash
sudo systemctl start bitbadgeschaind
```

### Adım 10: Logları İzleyin

```bash
journalctl -u bitbadgeschaind -f
```

---

## ⚙️ EVM/JSON-RPC Yapılandırması (Opsiyonel)

### EVM desteğini etkinleştirmek isterseniz:

### app.toml Yapılandırması

```bash
nano $HOME/.bitbadgeschain/config/app.toml
```

Aşağıdaki bölümü ekleyin veya güncelleyin:

```toml
[evm]
# Mainnet için 50024, Testnet için 50025
evm-chain-id = 50024
```

### config.toml Yapılandırması

```bash
nano $HOME/.bitbadgeschain/config/config.toml
```

Aşağıdaki bölümü ekleyin:

```toml
[json-rpc]
# JSON-RPC sunucusunu etkinleştir (EVM RPC için gerekli)
enable = true

# Bağlanma adresi (harici erişim için 0.0.0.0 kullanın)
address = "127.0.0.1:8545"

# WebSocket adresi
ws-address = "127.0.0.1:8546"
```

**Not:** JSON-RPC yapılandırması opsiyoneldir. Sadece EVM özelliklerini kullanmak istiyorsanız yapılandırın.

### Alternatif: Komut Satırı Flag'leri

Servis dosyanızda flag ekleyebilirsiniz:

```bash
sudo nano /etc/systemd/system/bitbadgeschaind.service
```

ExecStart satırına ekleyin:
```
--json-rpc.enable --json-rpc.address="127.0.0.1:8545"
```

Ardından servisi yeniden yükleyin:
```bash
sudo systemctl daemon-reload
sudo systemctl restart bitbadgeschaind
```

---

## ✅ Upgrade Sonrası Doğrulama

```bash
# Versiyon kontrolü
bitbadgeschaind version
# Çıktı: v24 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# JSON-RPC çalışıyor mu (yapılandırdıysanız)
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  http://127.0.0.1:8545
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v24 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v24 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```


### Evm/Json-Rpc yapılandırması (Opsiyonel)-Hızlı Komut
```
sed -i '/^\[evm\]/,/^\[/{/evm-chain-id/d}' $HOME/.bitbadgeschain/config/app.toml 2>/dev/null; \
grep -q "^\[evm\]" $HOME/.bitbadgeschain/config/app.toml || echo -e "\n[evm]" >> $HOME/.bitbadgeschain/config/app.toml; \
grep -q "evm-chain-id" $HOME/.bitbadgeschain/config/app.toml && \
sed -i 's/^evm-chain-id.*/evm-chain-id = 50024/' $HOME/.bitbadgeschain/config/app.toml || \
echo 'evm-chain-id = 50024' >> $HOME/.bitbadgeschain/config/app.toml; \
grep -q "^\[json-rpc\]" $HOME/.bitbadgeschain/config/config.toml || echo -e "\n[json-rpc]" >> $HOME/.bitbadgeschain/config/config.toml; \
sed -i 's/^enable *= *.*/enable = true/' $HOME/.bitbadgeschain/config/config.toml 2>/dev/null || echo 'enable = true' >> $HOME/.bitbadgeschain/config/config.toml; \
sed -i 's/^address *= *.*/address = "127.0.0.1:8545"/' $HOME/.bitbadgeschain/config/config.toml 2>/dev/null || echo 'address = "127.0.0.1:8545"' >> $HOME/.bitbadgeschain/config/config.toml; \
sed -i 's/^ws-address *= *.*/ws-address = "127.0.0.1:8546"/' $HOME/.bitbadgeschain/config/config.toml 2>/dev/null || echo 'ws-address = "127.0.0.1:8546"' >> $HOME/.bitbadgeschain/config/config.toml; \
echo "EVM ve JSON-RPC ayarları tamamlandı."
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v24_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=8890000
TARGET_VERSION="v24"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v24 Upgrade in $BLOCKS_LEFT blocks!"
        
        if [ "$BLOCKS_LEFT" -le 50 ]; then
            echo "🚨 ALERT: Only $BLOCKS_LEFT blocks left! Be ready!"
        fi
        
        if [ "$BLOCKS_LEFT" -le 20 ]; then
            echo "🔥 CRITICAL: Only $BLOCKS_LEFT blocks left! Prepare for manual intervention!"
        fi
    fi
    
    if [ "$CURRENT_HEIGHT" -ge "$UPGRADE_HEIGHT" ] && [ "$CURRENT_VERSION" != "$TARGET_VERSION" ]; then
        echo "🔥 CRITICAL: Upgrade block reached! Manual intervention needed NOW!"
        echo "Run the manual upgrade commands immediately!"
    fi
    
    sleep 30
done
EOF

chmod +x $HOME/v24_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v24_monitor
./v24_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v24
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v24/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v24/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v24/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v24/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v24/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/
cd $HOME/bitbadgeschain
git checkout v24
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind
sudo systemctl restart bitbadgeschaind
```

### Problem: Validator Jailed Oldu

```bash
bitbadgeschaind tx slashing unjail \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

### Problem: JSON-RPC Çalışmıyor

```bash
# Port dinleniyor mu kontrol edin
netstat -tulpn | grep 8545

# Logları kontrol edin
journalctl -u bitbadgeschaind -f | grep -i "json-rpc\|evm"

# Yapılandırmayı kontrol edin
cat $HOME/.bitbadgeschain/config/app.toml | grep -A 2 "\[evm\]"
cat $HOME/.bitbadgeschain/config/config.toml | grep -A 5 "\[json-rpc\]"
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((8890000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
echo "Kalan blok: $REMAINING | Tahmini süre: $HOURS saat"
```

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v24 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v24/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 8890000 | Remaining: $((8890000 - CURRENT))"

echo "=== EVM Configuration (Optional) ==="
echo "Check app.toml for [evm] section"
echo "Check config.toml for [json-rpc] section"
```

---

## 📝 v24 Özellik Notları

### Büyük Değişiklikler:

#### 🔴 Kaldırılan:
- **CosmWASM/WASM modülleri tamamen kaldırıldı**
- **libwasmvm bağımlılığı artık gerekli değil**
- Binary boyutu küçüldü (WASM olmadan)

#### 🟢 Eklenen:
- **Tam EVM uyumluluğu**
  - Ethereum JSON-RPC API
  - ERC20 token standardı
  - Smart contract desteği
  - Precompiled contracts
- **Modül İsim Değişikliği:** badges → tokenization
- **Interchain Queries:** Cross-chain token ownership doğrulaması
- **EVM Chain ID:** 
  - Mainnet: 50024
  - Testnet: 50025

### EVM Özelliklerini Kullanmak İçin:

1. **app.toml'da EVM chain ID belirtin**
2. **config.toml'da JSON-RPC'yi etkinleştirin**
3. **Port 8545 (HTTP) ve 8546 (WebSocket) açık olmalı**
4. **MetaMask gibi wallet'larla bağlanabilirsiniz:**
   - Network Name: BitBadges Mainnet
   - RPC URL: https://evm-rpc.bitbadges.io (veya kendi node'unuz)
   - Chain ID: 50024
   - Currency Symbol: BADGE

### Önemli:
- JSON-RPC yapılandırması **opsiyoneldir**
- EVM özelliklerini kullanmayacaksanız yapılandırmanıza gerek yok
- Validator olarak çalışmaya devam edebilirsiniz

---

## 🌐 EVM RPC Test Komutları

### Eğer JSON-RPC'yi etkinleştirdiyseniz:

```bash
# Chain ID kontrolü
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  http://127.0.0.1:8545

# Block number kontrolü
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://127.0.0.1:8545

# Network version
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":1}' \
  http://127.0.0.1:8545
```

---

**Not:** Bu rehber, v23'ten v24'e geçiş için hazırlanmıştır. v24 büyük bir upgrade olduğu için tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı tamamlayın. EVM özelliklerini kullanmak opsiyoneldir.
