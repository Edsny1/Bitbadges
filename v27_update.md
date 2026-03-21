# BitBadges v27 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v27 Upgrade Bilgileri

- **Upgrade Adı:** v27
- **Upgrade Blok Yüksekliği:** 9380000
- **Tahmini Zaman:** 24 Mart 2026, 18:53:57 EST (25 Mart 2026, 01:53:57 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/36
- **Mevcut Versiyon:** v26
- **Hedef Versiyon:** v27
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v27

---

## 🆕 v27 Upgrade Özeti

Bu upgrade **güvenlik ve doğrulama** odaklı iyileştirmeler içerir:

### ✨ Yeni Özellikler ve İyileştirmeler:

#### 1. **Collection Invariant Enforcement (Güçlendirildi)**
- Collection invariant kontrolü daha sıkı hale getirildi
- Veri tutarlılığı garantileri artırıldı
- Geçersiz durumları önlemek için ekstra doğrulamalar
- **Fayda:** Daha güvenli ve tutarlı collection yönetimi

#### 2. **IBC Transfer Handling İyileştirmeleri**
- GAMM (Generalized Automated Market Maker) için IBC transfer işleme geliştirildi
- custom-hooks ile IBC entegrasyonu optimize edildi
- Cross-chain token transferlerinde daha iyi hata yönetimi
- **Fayda:** Daha güvenilir IBC işlemleri

#### 3. **Backed-Minting Approval Validation**
- Backed-minting approvals için validation guardrails eklendi
- Hatalı minting işlemlerini önleyecek kontroller
- Approval mekanizması daha güvenli hale getirildi
- **Fayda:** Token minting işlemlerinde daha fazla güvenlik

### 🔒 Güvenlik İyileştirmeleri:

- **Invariant Enforcement:** Veri bütünlüğü garantileri
- **IBC Security:** Cross-chain transfer güvenliği artırıldı
- **Minting Validation:** Backed-minting için ekstra doğrulama katmanları
- **Error Handling:** Daha iyi hata yönetimi ve logging

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v26 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((9380000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v27 Özel Notları:

- **Güvenlik Odaklı:** Collection invariant ve minting validation güçlendirildi
- **IBC İyileştirmeleri:** GAMM ve custom-hooks entegrasyonu optimize edildi
- **Validation Guardrails:** Backed-minting için yeni doğrulama mekanizmaları
- **Breaking Changes YOK:** Geriye dönük uyumlu
- **Performans:** Minimal overhead ile güvenlik artışı

---

## 🚀 v27 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v27 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v27 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v27
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v27`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v27" NEEDED at height: 9380000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v27 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v27`

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
# Çıktı: v27 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# Collection invariant kontrolü
bitbadgeschaind query tokenization params

# IBC durumu (Relayer çalıştırıyorsanız)
bitbadgeschaind query ibc client states --limit 3
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v27 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v27 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v27_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=9380000
TARGET_VERSION="v27"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v27 Upgrade in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v27_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v27_monitor
./v27_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v27
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v27/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v27/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v27/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v27/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v27/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/
cd $HOME/bitbadgeschain
git checkout v27
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind
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

### Problem: Collection Invariant Hatası

v27 ile invariant kontrolü güçlendirildi. Eğer hata alırsanız:

```bash
# Collection durumunu kontrol edin
bitbadgeschaind query tokenization collection COLLECTION_ID

# Collection parametrelerini kontrol edin
bitbadgeschaind query tokenization params

# Logları detaylı inceleyin
journalctl -u bitbadgeschaind -n 100 | grep -i "invariant"
```

### Problem: Backed-Minting Approval Hatası

```bash
# Minting approval'larını kontrol edin
bitbadgeschaind query tokenization collection COLLECTION_ID

# Approval validation hatası alıyorsanız, logları kontrol edin
journalctl -u bitbadgeschaind -f | grep -i "backed-minting\|approval"
```

### Problem: IBC Transfer GAMM Hatası

```bash
# IBC transfer durumunu kontrol edin
bitbadgeschaind query ibc client states
bitbadgeschaind query ibc channel channels

# GAMM pool'ları kontrol edin (varsa)
bitbadgeschaind query gamm pools 2>/dev/null || echo "GAMM query not available"

# Custom hooks durumu
journalctl -u bitbadgeschaind -n 50 | grep -i "gamm\|custom-hook"
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((9380000 - CURRENT))
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

echo "=== v27 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v27/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 9380000 | Remaining: $((9380000 - CURRENT))"

echo "=== Collection Invariant Check ==="
bitbadgeschaind query tokenization params 2>/dev/null && echo "✅ Tokenization OK" || echo "⏳ Waiting"

echo "=== IBC Status Check ==="
bitbadgeschaind query ibc client states --limit 2 2>/dev/null && echo "✅ IBC OK" || echo "⏳ Waiting"
```

---

## 📝 v27 Özellik Notları

### 🔒 Güvenlik ve Doğrulama İyileştirmeleri:

#### 1. **Collection Invariant Enforcement (Güçlendirildi)**

**Ne değişti?**
- Collection invariant kontrolleri daha sıkı hale getirildi
- Veri tutarlılığı her işlemde doğrulanıyor
- Geçersiz durumlar tespit edilip önleniyor

**Neden önemli?**
- Collection verilerinin her zaman tutarlı olması garantileniyor
- Hatalı durumları önleyerek data corruption riski azalıyor
- Daha güvenilir collection yönetimi

**Örnek senaryolar:**
```bash
# Collection invariant kontrolü
bitbadgeschaind query tokenization collection 1

# Eğer invariant ihlali varsa, transaction reject edilecek
# v27 öncesi: Bazı geçersiz durumlar geçebiliyordu
# v27 sonrası: Tüm geçersiz durumlar engelleniyor
```

#### 2. **IBC Transfer Handling - GAMM Integration**

**Ne değişti?**
- GAMM (Generalized Automated Market Maker) için IBC transfer işleme geliştirildi
- custom-hooks ile daha iyi entegrasyon
- Cross-chain token transferlerinde gelişmiş hata yönetimi

**Neden önemli?**
- DEX ve liquidity pool işlemlerinde daha güvenilir IBC transferleri
- Cross-chain arbitrage ve swap işlemlerinde daha az hata
- Relayer'lar için daha öngörülebilir davranış

**Örnek kullanım:**
```bash
# IBC transfer ile GAMM entegrasyonu
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  cosmos1receiver... \
  1000ubadge \
  --from wallet \
  --chain-id bitbadges-1 \
  --memo '{"gamm":{"pool_id":"1","action":"swap"}}'
```

#### 3. **Backed-Minting Approval Validation Guardrails**

**Ne değişti?**
- Backed-minting approval'ları için yeni validation katmanları
- Hatalı minting işlemlerini önleyen kontroller
- Approval mekanizması daha güvenli hale getirildi

**Neden önemli?**
- Yetkisiz veya hatalı token minting işlemleri engelleniyor
- Supply manipulation riskini azaltıyor
- Daha güvenli token ekonomisi

**Validation kontrolleri:**
```bash
# Backed-minting approval oluştururken
# v27 şunları kontrol ediyor:
# 1. Approval sahibinin yetkisi var mı?
# 2. Mint edilecek miktar geçerli mi?
# 3. Backing asset yeterli mi?
# 4. Invariant'lar korunuyor mu?

# Örnek minting transaction
bitbadgeschaind tx tokenization mint-badges \
  --collection-id 1 \
  --amount 100 \
  --backing-asset cosmos1... \
  --from wallet \
  --chain-id bitbadges-1
```

### 📊 Teknik Detaylar:

#### Invariant Enforcement

**Kontrol edilen invariant'lar:**
- Total supply consistency
- Address balance totals
- Permission coherence
- Timeline validity
- Metadata integrity

**Örnek invariant kontrolü:**
```bash
# Her transaction sonrası otomatik kontrol
# Eğer invariant ihlali varsa, transaction revert edilir

# Manuel invariant kontrolü (testing için)
bitbadgeschaind query tokenization validate-invariants COLLECTION_ID
```

#### IBC GAMM Handling

**İyileştirmeler:**
- Atomic swap operations
- Better timeout handling
- Improved error messages
- Enhanced logging

**Custom hooks örneği:**
```bash
# GAMM pool'a IBC transfer
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  cosmos1pooladdress... \
  1000ubadge \
  --memo '{"wasm":{"contract":"pool","msg":{"swap":{}}}}' \
  --from wallet
```

#### Backed-Minting Validation

**Validation steps:**
1. **Authorization Check:** Minter yetkili mi?
2. **Amount Validation:** Miktar geçerli aralıkta mı?
3. **Backing Verification:** Backing asset mevcut mu?
4. **Invariant Check:** İşlem sonrası invariant'lar korunuyor mu?
5. **Supply Limit:** Max supply aşılmıyor mu?

### 🎯 Etkilenen Kullanıcılar:

#### Collection Creators/Managers:
- Daha güvenli collection yönetimi
- Invariant ihlallerinin önlenmesi
- Hatalı durumlardan korunma

#### IBC Relayer Operatörleri:
- GAMM entegrasyonundan faydalanabilir
- Daha öngörülebilir IBC davranışı
- Gelişmiş hata raporlama

#### Token Minters:
- Backed-minting için ekstra validation
- Daha güvenli minting işlemleri
- Clear error messages

#### Developers:
- Daha tutarlı API davranışı
- Better error handling
- Enhanced logging

### 🚀 Performans Etkileri:

- **Invariant Checks:** Minimal overhead (~1-2% CPU)
- **IBC Processing:** Marginally improved
- **Validation:** Negligible impact
- **Overall:** Güvenlik artışı >> Performans maliyeti

### ⚙️ Yapılandırma Değişiklikleri:

**v27 yeni yapılandırma gerektirmiyor**
- Mevcut config dosyaları çalışmaya devam eder
- Opsiyonel: Logging level artırılabilir (invariant debugging için)

```toml
# config.toml (opsiyonel)
[log]
level = "info"  # veya "debug" daha detaylı loglar için
```

---

## 🌐 IBC Relayer Operatörleri İçin

### GAMM Integration Notları:

```bash
# 1. IBC channel'ları kontrol edin
bitbadgeschaind query ibc channel channels

# 2. GAMM pool durumu (varsa)
bitbadgeschaind query gamm pools

# 3. Custom hooks ile transfer test
bitbadgeschaind tx ibc-transfer transfer \
  transfer channel-0 \
  cosmos1... \
  1000ubadge \
  --memo '{"custom":"test"}' \
  --from wallet

# 4. Relayer loglarını izleyin
journalctl -u relayer -f | grep -i "gamm\|custom"
```

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v27
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/36
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın

---

## 🔄 Upgrade Timeline

- **Proposal Submitted:** ✅ Completed
- **Voting Period:** Now + 24 hours
- **Upgrade Time:** 25 Mart 2026, 01:53:57 (Türkiye Saati)
- **Preparation:** Start NOW
- **Manual Intervention:** At block 9380000

---

**Not:** Bu rehber, v26'dan v27'ye geçiş için hazırlanmıştır. Güvenlik ve doğrulama odaklı iyileştirmeler içerir. Tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı tamamlayın.
