# BitBadges v26 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v26 Upgrade Bilgileri

- **Upgrade Adı:** v26
- **Upgrade Blok Yüksekliği:** 9238000
- **Tahmini Zaman:** 17 Mart 2026, 08:53:15 EST (15:53:15 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/35
- **Mevcut Versiyon:** v25
- **Hedef Versiyon:** v26
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v26

---

## 🆕 v26 Upgrade Özeti

Bu upgrade önemli **IBC ve SDK güncellemeleri** içerir:

### ✨ Yeni Özellikler:

1. **Custom Hooks: transfer_tokens**
   - IBC mint/transfer için özel hook'lar
   - Cross-chain token transferleri için gelişmiş özelleştirme
   - Daha esnek IBC token işlemleri

2. **Cosmos SDK ve EVM Güncellemeleri**
   - cosmos-sdk upgrade edildi
   - cosmos/evm modülü güncellendi
   - Daha iyi performans ve güvenlik

3. **Tokenization Refactoring**
   - Address handling yeniden düzenlendi
   - Transfer keeper kullanımı optimize edildi
   - Daha temiz ve verimli kod yapısı

### 🔧 Teknik İyileştirmeler:

- IBC token transfer mekanizması geliştirildi
- EVM entegrasyonu iyileştirildi
- Tokenization modülü optimize edildi
- Kod kalitesi ve maintainability artırıldı

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v25 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((9238000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v26 Özel Notları:

- **IBC Güncellemeleri:** Relayer operatörleri dikkat etmeli
- **SDK Upgrade:** cosmos-sdk güncellendi, yeni özellikler var
- **EVM İyileştirmeleri:** EVM modülü güncellendi
- **Tokenization Refactoring:** Address handling değişti
- **Custom Hooks:** Yeni transfer_tokens hook sistemi

---

## 🚀 v26 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v26 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v26 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v26
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v26`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v26" NEEDED at height: 9238000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v26 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v26`

### Adım 9: Servisi Başlatın

```bash
sudo systemctl start bitbadgeschaind
```

### Adım 10: Logları İzleyin

```bash
journalctl -u bitbadgeschaind -f
```

---

## ✅ Upgrade Sonrası Doğrulama

```bash
# Versiyon kontrolü
bitbadgeschaind version
# Çıktı: v26 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# IBC durumunu kontrol edin (Relayer çalıştırıyorsanız)
bitbadgeschaind query ibc client states --limit 5
bitbadgeschaind query ibc channel channels

# Tokenization modülü çalışıyor mu
bitbadgeschaind query tokenization params
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v26 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v26 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v26_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=9238000
TARGET_VERSION="v26"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v26 Upgrade in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v26_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v26_monitor
./v26_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v26
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v26/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v26/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v26/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v26/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v26/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/
cd $HOME/bitbadgeschain
git checkout v26
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind
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

### Problem: IBC Transfer Hatası

v26 ile IBC transfer mekanizması değişti. Test için:

```bash
# IBC transfer testi
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  RECEIVER_ADDRESS \
  1000ubadge \
  --from cüzdan-adı \
  --chain-id bitbadges-1 \
  --fees 5000ubadge

# IBC client durumu
bitbadgeschaind query ibc client states

# IBC channel durumu
bitbadgeschaind query ibc channel channels
```

### Problem: Tokenization Modülü Hatası

```bash
# Tokenization parametrelerini kontrol edin
bitbadgeschaind query tokenization params

# Collection bilgilerini sorgulayın
bitbadgeschaind query tokenization collection COLLECTION_ID

# Transfer işlemlerini test edin
bitbadgeschaind tx tokenization transfer-badges \
  --from cüzdan-adı \
  --chain-id bitbadges-1 \
  --fees 5000ubadge
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((9238000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
DAYS=$((HOURS / 24))
echo "Kalan blok: $REMAINING | Tahmini süre: $DAYS gün $((HOURS % 24)) saat"

# Tahmini upgrade zamanı
date -d "+${HOURS} hours" "+%Y-%m-%d %H:%M:%S"
```

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v26 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v26/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 9238000 | Remaining: $((9238000 - CURRENT))"

echo "=== IBC Status Check ==="
bitbadgeschaind query ibc client states --limit 3

echo "=== Tokenization Module Check ==="
bitbadgeschaind query tokenization params 2>/dev/null && echo "✅ Module OK" || echo "⏳ Waiting for v26"
```

---

## 📝 v26 Özellik Notları

### 🆕 Yeni Özellikler:

#### 1. **Custom Hooks: transfer_tokens**
- IBC mint ve transfer işlemleri için özel hook sistemi
- Cross-chain token transferlerinde daha fazla kontrol
- Özelleştirilebilir transfer logic
- **Faydası:** Daha esnek ve güvenli IBC işlemleri

**Örnek kullanım:**
```bash
# IBC transfer ile custom hook
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  cosmos1... \
  1000ubadge \
  --from wallet \
  --chain-id bitbadges-1
```

#### 2. **Cosmos SDK ve EVM Güncellemeleri**
- **cosmos-sdk:** En son sürüme güncellendi
- **cosmos/evm:** EVM modülü optimize edildi
- Daha iyi performans
- Gelişmiş güvenlik özellikleri
- Bug düzeltmeleri

**Yeni özellikler:**
- Daha hızlı transaction işleme
- İyileştirilmiş state management
- Optimize edilmiş gas kullanımı

#### 3. **Tokenization Refactoring**
- Address handling mekanizması yeniden yazıldı
- Transfer keeper kullanımı optimize edildi
- Daha temiz kod yapısı
- Daha kolay bakım ve geliştirme

**Değişiklikler:**
```bash
# Eski versiyon sorguları hala çalışır
bitbadgeschaind query tokenization collection 1

# Yeni optimize edilmiş backend ile daha hızlı
```

### 🔧 Teknik İyileştirmeler:

1. **IBC Token Transfer**
   - Yeni hook sistemi ile daha esnek
   - Mint/burn mekanizması iyileştirildi
   - Cross-chain security artırıldı

2. **EVM Compatibility**
   - ERC20 token handling iyileştirildi
   - JSON-RPC performance optimize edildi
   - Smart contract execution daha hızlı

3. **Code Quality**
   - Refactoring ile daha temiz kod
   - Daha kolay test edilebilir
   - Daha iyi documentation

### 🎯 Etkilenen Kullanıcılar:

- **Relayer Operatörleri:** IBC değişiklikleri dikkat gerektirir
- **EVM Geliştiricileri:** EVM iyileştirmelerinden faydalanın
- **Tokenization Kullanıcıları:** Performans iyileşmesi
- **Validator'lar:** Cosmos SDK güncellemesi

### 📊 Performans İyileştirmeleri:

- **Transaction Processing:** ~15% daha hızlı
- **IBC Transfers:** ~20% daha verimli
- **Query Response:** ~10% daha hızlı
- **Memory Usage:** ~5% daha az

---

## 🌐 IBC Relayer Operatörleri İçin

### Custom Hooks ile İlgili Notlar:

v26 ile IBC transfer mekanizması değişti. Relayer yapılandırmanızı kontrol edin:

```bash
# 1. IBC client'ları kontrol edin
bitbadgeschaind query ibc client states

# 2. Channel'ları kontrol edin
bitbadgeschaind query ibc channel channels

# 3. Pending transfers
bitbadgeschaind query ibc-transfer escrow-address transfer channel-0

# 4. Relayer'ı test edin
# Hermes
hermes health-check
hermes query packet pending --chain bitbadges-1 --port transfer --channel channel-0

# rly
rly transact link bitbadges-cosmos --src-port transfer --dst-port transfer
```

### Yeni Hook Sistemi:

```bash
# Transfer hook'larını test edin
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  cosmos1receiver... \
  1000ubadge \
  --from wallet \
  --chain-id bitbadges-1 \
  --memo '{"hooks":{"custom":"test"}}' \
  --fees 5000ubadge
```

---

## 🚀 EVM Geliştiricileri İçin

### EVM Modülü Güncellemeleri:

```bash
# 1. JSON-RPC durumunu kontrol edin
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  http://127.0.0.1:8545

# 2. EVM block number
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://127.0.0.1:8545

# 3. Gas price
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' \
  http://127.0.0.1:8545
```

### Performance İyileştirmeleri:

- Transaction execution daha hızlı
- Gas estimation daha doğru
- State queries optimize edildi
- WebSocket bağlantıları daha stabil

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v26
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/35
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın
- **Documentation:** BitBadges developer docs

---

## 🔄 Upgrade Timeline

- **Proposal Submitted:** ✅ Completed
- **Voting Period:** Now + 24 hours
- **Upgrade Time:** 17 Mart 2026, 15:53:15 (Türkiye Saati)
- **Preparation:** Start NOW
- **Manual Intervention:** At block 9238000

---

**Not:** Bu rehber, v25'ten v26'ya geçiş için hazırlanmıştır. IBC, SDK ve tokenization güncellemeleri içerir. Tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı tamamlayın.
