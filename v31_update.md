# BitBadges v31 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v31 Upgrade Bilgileri

- **Upgrade Adı:** v31
- **Upgrade Blok Yüksekliği:** 10210000
- **Tahmini Zaman:** 13 Mayıs 2026, 12:13:00 EST (13 Mayıs 2026, 19:13:00 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/40
- **Mevcut Versiyon:** v30
- **Hedef Versiyon:** v31
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v31

---

## 🆕 v31 Upgrade Özeti - Minor Maintenance + Protocol Fee Change

Bu upgrade **minor maintenance ve önemli protocol fee değişikliği** içerir:

### 🛠️ Ana Özellikler:

#### 1. **Protocol Fee Change (IMPORTANT)** 💰
- **Eski Sistem:** 1 USDC + 0.01 fee = 1.01 USDC total
- **Yeni Sistem:** 1 USDC (0.99 + 0.01 fee) = 1 USDC total
- **Fee artık dahil (inclusive)** - kullanıcı açısından daha net
- **Breaking Change:** Ücret hesaplama mantığı değişti

**Örnek:**
```
Eski (v30):
- İşlem tutarı: 1 USDC
- Protocol fee: 0.01 USDC
- Total ödenen: 1.01 USDC
- Alıcının aldığı: 1 USDC

Yeni (v31):
- İşlem tutarı: 1 USDC
- Alıcının aldığı: 0.99 USDC
- Protocol fee: 0.01 USDC
- Total ödenen: 1 USDC (inclusive)
```

#### 2. **Minor Maintenance** 🧹
- Küçük bakım düzeltmeleri
- Code cleanup
- Bug fixes

#### 3. **Quality of Life Improvements** ✨
- Kullanıcı deneyimi iyileştirmeleri
- Better error messages
- Improved logging

#### 4. **Dependency Fixes** 📦
- Bağımlılık güncellemeleri
- Security patches
- Library updates

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v30 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((10210000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v31 Özel Notları:

- **Protocol Fee Change:** ÖNEMLİ - Ücret hesaplama değişti
- **Inclusive Fee:** Fee artık dahil (1 USDC → 0.99 + 0.01)
- **Minor Maintenance:** Küçük düzeltmeler
- **No Migration:** State migration yok
- **Fast Upgrade:** Hızlı geçiş (~5 dakika)
- **QoL Improvements:** Kullanıcı deneyimi iyileştirmeleri

---

## 🚀 v31 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v31 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v31 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v31
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v31`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v31" NEEDED at height: 10210000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v31 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v31`

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
# Çıktı: v31 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# Protocol fee parametrelerini kontrol edin
bitbadgeschaind query tokenization params | grep -i "fee"
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v31 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v31 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v31_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=10210000
TARGET_VERSION="v31"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v31 Upgrade in $BLOCKS_LEFT blocks!"
        echo "📢 Protocol Fee Change: Inclusive fee model!"
        
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

chmod +x $HOME/v31_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v31_monitor
./v31_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v31
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v31/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v31/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v31/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v31/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v31/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/
cd $HOME/bitbadgeschain
git checkout v31
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind
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

### Problem: Protocol Fee Hesaplama Yanlış Görünüyor

v31 ile fee inclusive oldu:

```bash
# Yeni sistem (v31):
# 1 USDC işlem = 0.99 alıcıya + 0.01 fee
# Total: 1 USDC (fee dahil)

# Fee parametrelerini kontrol edin
bitbadgeschaind query tokenization params
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((10210000 - CURRENT))
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

echo "=== v31 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v31/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 10210000 | Remaining: $((10210000 - CURRENT))"
```

---

## 📝 v31 Detaylı Notlar

### 💰 Protocol Fee Change - ÖNEMLİ!

#### **Eski Sistem (v30 ve öncesi):**
```
İşlem: 1 USDC
Fee: +0.01 USDC
Total: 1.01 USDC ödenen
Alıcı: 1 USDC alır

Kullanıcı görünümü:
"1 USDC gönderiyorum, 1.01 USDC ödeyeceğim"
```

#### **Yeni Sistem (v31):**
```
İşlem: 1 USDC
Alıcı: 0.99 USDC
Fee: 0.01 USDC
Total: 1 USDC ödenen (fee inclusive)

Kullanıcı görünümü:
"1 USDC gönderiyorum, 1 USDC ödeyeceğim"
```

#### **Neden Bu Değişiklik?**

1. **Daha Net Kullanıcı Deneyimi:**
   - Kullanıcı ne gördüyse onu öder
   - Sürpriz ücret yok
   - Daha öngörülebilir

2. **Standart Pratik:**
   - Çoğu platform inclusive fee kullanır
   - PayPal, Stripe, vs. benzer model
   - Industry standard

3. **Daha Kolay Hesaplama:**
   - Frontend için daha basit
   - Kullanıcı için daha anlaşılır
   - Muhasebe için daha net

#### **Developer'lar İçin:**

**API Response Değişikliği:**
```json
// v30
{
  "amount": "1000000",  // 1 USDC
  "fee": "10000",        // 0.01 USDC
  "total": "1010000"     // 1.01 USDC
}

// v31
{
  "amount": "1000000",      // 1 USDC (inclusive)
  "recipient_gets": "990000", // 0.99 USDC
  "protocol_fee": "10000",   // 0.01 USDC
  "total": "1000000"         // 1 USDC (fee included)
}
```

**Frontend Kodu Güncelleme:**
```javascript
// Eski (v30)
const total = amount + fee;
displayMessage(`Send ${amount} + ${fee} fee = ${total} total`);

// Yeni (v31)
const recipientGets = amount - fee;
displayMessage(`Send ${amount} (recipient gets ${recipientGets})`);
```

### 🛠️ Minor Maintenance

**Bug Fixes:**
- Edge case handling
- Error recovery improvements
- Panic prevention

**Code Cleanup:**
- Dead code removal
- Better code organization
- Improved comments

### ✨ Quality of Life Improvements

**Better Error Messages:**
```bash
# v30
"transaction failed: invalid amount"

# v31
"Transaction failed: Amount must be greater than protocol fee (0.01 USDC). 
Minimum sendable amount: 0.02 USDC (recipient gets 0.01 USDC)"
```

**Improved Logging:**
- More detailed transaction logs
- Better error context
- Easier debugging

### 📦 Dependency Updates

**Updated Libraries:**
- Security patches
- Bug fixes
- Performance improvements

**Version Updates:**
```
cosmos-sdk: patch update
tendermint: security patch
go modules: dependency updates
```

---

## 🎯 Impact Analysis

### For Users:
- ✅ **Clearer pricing** - Fee inclusive
- ✅ **No surprises** - Pay what you see
- ✅ **Better UX** - More intuitive

### For Developers:
- ⚠️ **API Changes** - Update frontend code
- ✅ **Better errors** - Easier debugging
- ✅ **Cleaner code** - Easier maintenance

### For Validators:
- ✅ **Fast upgrade** - No migration
- ✅ **Low risk** - Minor changes
- ✅ **Better stability** - Bug fixes

---

## 🔄 Migration Guide for Developers

### Frontend Integration Changes:

**1. Display Logic:**
```javascript
// Before v31
function displayTransactionCost(amount) {
  const fee = calculateFee(amount);
  return `Total: ${amount + fee} USDC (${amount} + ${fee} fee)`;
}

// After v31
function displayTransactionCost(amount) {
  const fee = calculateFee(amount);
  const recipientGets = amount - fee;
  return `Total: ${amount} USDC (recipient gets ${recipientGets})`;
}
```

**2. Validation:**
```javascript
// Before v31
if (amount <= 0) {
  throw new Error("Amount must be positive");
}

// After v31
const minAmount = protocolFee * 2; // 0.02 USDC minimum
if (amount < minAmount) {
  throw new Error(
    `Amount must be at least ${minAmount} USDC to cover protocol fee`
  );
}
```

**3. Receipt Display:**
```javascript
// After v31
{
  "You sent": "1.00 USDC",
  "Recipient receives": "0.99 USDC",
  "Protocol fee": "0.01 USDC",
  "Total charged": "1.00 USDC"
}
```

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v31
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/40
- **Protocol Fee Documentation:** Check release notes
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın

---

## 🔄 Upgrade Timeline

- **Proposal Submitted:** ✅ Completed
- **Voting Period:** Now + 24 hours
- **Upgrade Time:** 13 Mayıs 2026, 19:13:00 (Türkiye Saati)
- **Expected Downtime:** ~5 dakika
- **Migration:** NO
- **Risk Level:** LOW
- **API Changes:** YES (fee calculation)
- **Preparation:** Start NOW

---

## ⚡ Quick Summary

**What's Changing:**
- Protocol fee now inclusive (1 USDC = 0.99 + 0.01)
- Minor maintenance and bug fixes
- QoL improvements
- Dependency updates

**What's NOT Changing:**
- No new features
- No state migration
- No breaking protocol changes (except fee)

**Action Required:**
- Validators: Upgrade node (standard process)
- Developers: Update frontend fee display logic
- Users: Enjoy clearer pricing!

---

**Not:** v31 minor bir maintenance release olmasına rağmen, protocol fee değişikliği nedeniyle frontend entegrasyonları güncellenmelidir. Fee artık inclusive model kullanıyor - kullanıcı ne görüyorsa onu öder.
