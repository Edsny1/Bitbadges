# BitBadges v29 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v29 Upgrade Bilgileri

- **Upgrade Adı:** v29
- **Upgrade Blok Yüksekliği:** 9768000
- **Tahmini Zaman:** 16 Nisan 2026, 22:30:28 EST (17 Nisan 2026, 05:30:28 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/38
- **Mevcut Versiyon:** v28
- **Hedef Versiyon:** v29
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v29

---

## 🆕 v29 Upgrade Özeti - Comprehensive Enhancement

Bu upgrade **governance, approvals, ve developer experience** odaklı kapsamlı iyileştirmeler içerir:

### ✨ Yeni Özellikler:

#### 1. **Voting Challenge Enhancements**
- **Reset after execution:** Challenge tamamlandıktan sonra otomatik reset
- **Delay after quorum:** Quorum'a ulaştıktan sonra execution delay
- **Better governance security:** Daha güvenli oylama mekanizması
- **Fayda:** Manipulation önleme, daha adil voting

#### 2. **Expanded Time-Based Restrictions**
- Gelişmiş zaman bazlı kısıtlamalar
- Granular time controls
- Flexible scheduling
- **Fayda:** Daha esnek token management

#### 3. **UserApprovalSettings**
- User-level approval ayarları
- Granular permission control
- Custom approval workflows
- **Fayda:** Kullanıcı bazında izin yönetimi

#### 4. **Amount Scaling with Max Multiplier Caps**
- v28'deki amount scaling genişletildi
- Maximum multiplier limitleri
- Overflow koruması
- **Fayda:** Güvenli fiyat ölçeklendirme

#### 5. **Self-Referencing mustOwnTokens (Enhanced)**
- v28'deki self-reference özelliği geliştirildi
- Daha karmaşık conditional logic
- Nested requirements desteği
- **Fayda:** Advanced on-chain logic

#### 6. **Allowed Denoms Enforcement**
- Whitelist/blacklist denom sistemi
- Token type restrictions
- Cross-chain denom kontrolü
- **Fayda:** Daha kontrollü token ekonomisi

#### 7. **Granular Approval Change Events**
- Detaylı approval event tracking
- Change history logging
- Better audit trail
- **Fayda:** Gelişmiş monitoring ve compliance

#### 8. **Enriched Msg Responses**
- Daha zengin transaction response'ları
- Additional metadata
- Better error messages
- **Fayda:** Improved developer experience

#### 9. **CLI/Tooling Improvements**
- Enhanced CLI commands
- Better developer tools
- Improved documentation
- **Fayda:** Easier development

#### 10. **Cross-Platform Release CI**
- Multi-platform builds
- Automated releases
- Better binary distribution
- **Fayda:** Wider platform support

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v28 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((9768000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v29 Özel Notları:

- **No Migration:** State migration yok, hızlı upgrade
- **Voting Enhancements:** Governance mekanizması güçlendirildi
- **UserApprovalSettings:** Yeni user-level permission sistemi
- **Enhanced Events:** Daha detaylı event tracking
- **Developer Tools:** CLI ve tooling iyileştirmeleri
- **Breaking Changes:** Minimal - geriye dönük uyumlu

---

## 🚀 v29 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v29 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v29 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v29
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v29`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v29" NEEDED at height: 9768000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v29 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v29`

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
# Çıktı: v29 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# Yeni CLI commands test
bitbadgeschaind tx tokenization --help | grep -i "approval\|settings"

# Governance voting test
bitbadgeschaind query gov proposals --status voting_period
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v29 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v29 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v29_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=9768000
TARGET_VERSION="v29"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v29 Upgrade in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v29_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v29_monitor
./v29_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v29
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v29/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v29/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v29/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v29/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v29/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/
cd $HOME/bitbadgeschain
git checkout v29
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind
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

### Problem: UserApprovalSettings Hatası

```bash
# User approval settings'i kontrol edin
bitbadgeschaind query tokenization user-approval-settings ADDRESS

# Yeni approval settings oluşturun
bitbadgeschaind tx tokenization set-user-approval-settings \
  --settings '{"approval_level":"strict"}' \
  --from wallet \
  --chain-id bitbadges-1
```

### Problem: Voting Challenge Reset Çalışmıyor

```bash
# Voting challenge durumunu kontrol edin
bitbadgeschaind query gov proposal PROPOSAL_ID

# Challenge execution history
journalctl -u bitbadgeschaind -n 200 | grep -i "voting\|challenge\|quorum"
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((9768000 - CURRENT))
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

echo "=== v29 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 9768000 | Remaining: $((9768000 - CURRENT))"

echo "=== CLI Help Check ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v29/bin/bitbadgeschaind --help | grep -c "tx\|query" && echo "✅ CLI OK"
```

---

## 📝 v29 Özellik Notları - Detaylı Açıklama

### 1. **Voting Challenge Enhancements** 🗳️

**Reset After Execution:**
- Challenge tamamlandıktan sonra otomatik reset
- Yeni voting round için hazır
- State cleanup

**Delay After Quorum:**
- Quorum'a ulaştıktan sonra execution delay
- Grace period for review
- Last-minute objections

**Örnek Kullanım:**
```bash
# Voting challenge parametreleri
{
  "votingChallenge": {
    "quorumThreshold": "0.67",
    "executionDelay": "86400",  # 24 saat
    "autoReset": true
  }
}

# Challenge durumu sorgulama
bitbadgeschaind query gov proposal 38 --output json | \
  jq '.voting_params.quorum, .voting_params.execution_delay'
```

**Güvenlik Faydaları:**
- Flash voting prevention
- More democratic process
- Better community review time

### 2. **Expanded Time-Based Restrictions** ⏰

**Yeni Özellikler:**
```json
{
  "timeRestrictions": {
    "allowedTimeRanges": [
      {"start": "2026-04-17T00:00:00Z", "end": "2026-04-17T23:59:59Z"}
    ],
    "blockedTimeRanges": [
      {"start": "2026-04-18T00:00:00Z", "end": "2026-04-18T23:59:59Z"}
    ],
    "recurringSchedule": {
      "daysOfWeek": [1, 2, 3, 4, 5],  # Monday-Friday
      "hoursOfDay": [9, 10, 11, 12, 13, 14, 15, 16, 17]  # 9AM-5PM
    }
  }
}
```

**Use Cases:**
- Business hours only transfers
- Weekend locked tokens
- Event-based restrictions
- Scheduled releases

### 3. **UserApprovalSettings** 👤

**User-Level Permissions:**
```bash
# User approval settings oluşturma
bitbadgeschaind tx tokenization set-user-approval-settings \
  --approval-level "strict" \
  --require-signature true \
  --whitelist "bitbadges1...,bitbadges1..." \
  --from wallet \
  --chain-id bitbadges-1

# Settings sorgulama
bitbadgeschaind query tokenization user-approval-settings ADDRESS

# Örnek response:
{
  "approvalLevel": "strict",
  "requireSignature": true,
  "whitelist": ["bitbadges1...", "bitbadges1..."],
  "blacklist": [],
  "customRules": [
    {
      "condition": "amount > 100",
      "action": "require_multi_sig"
    }
  ]
}
```

**Approval Levels:**
- **none:** Tüm işlemler otomatik onaylı
- **basic:** Standart onay mekanizması
- **strict:** Multi-sig gerekli
- **custom:** Özel kurallar

### 4. **Amount Scaling with Max Multiplier Caps** 💰

**v28'den İyileştirmeler:**
```json
{
  "amountScaling": {
    "basePrice": "10",
    "scaleFactor": "1.5",
    "maxMultiplier": "10",  # NEW: Maximum 10x
    "capPerUser": "1000",   # NEW: User başına limit
    "overflowProtection": true  # NEW: Overflow koruması
  }
}
```

**Güvenlik:**
- Overflow attacks önleme
- Price manipulation koruması
- Fair pricing enforcement

**Örnek:**
```bash
# İlk 100 token: 10 BADGE
# Sonraki 100: 15 BADGE (1.5x)
# Sonraki 100: 22.5 BADGE (2.25x)
# ...
# Maximum: 100 BADGE (10x cap)
```

### 5. **Self-Referencing mustOwnTokens (Enhanced)** 🔄

**v28'den İyileştirmeler:**
```json
{
  "mustOwnTokens": [
    {
      "collectionId": "0",  # Self-reference
      "tokenIds": ["badge_1", "badge_2"],
      "minAmount": "1",
      "operator": "AND"  # NEW: AND/OR logic
    },
    {
      "collectionId": "0",
      "tokenIds": ["level_1"],
      "minAmount": "5",
      "nestedRequirements": [  # NEW: Nested conditions
        {
          "collectionId": "0",
          "tokenIds": ["skill_a", "skill_b"],
          "operator": "OR"
        }
      ]
    }
  ]
}
```

**Complex Logic:**
- AND/OR operators
- Nested requirements
- Conditional chains
- Dynamic unlocks

### 6. **Allowed Denoms Enforcement** 🪙

**Whitelist/Blacklist:**
```json
{
  "allowedDenoms": {
    "whitelist": ["ubadge", "uatom", "uosmo"],
    "blacklist": ["scam_token"],
    "requireWhitelisted": true,
    "crossChainVerification": true
  }
}
```

**Kullanım:**
```bash
# Collection oluştururken
bitbadgeschaind tx tokenization create-collection \
  --allowed-denoms "ubadge,uatom" \
  --require-whitelisted true \
  --from wallet

# Denom enforcement kontrolü
bitbadgeschaind query tokenization collection 1 | \
  jq '.allowedDenoms'
```

**Fayda:**
- Scam token prevention
- Controlled token economy
- Cross-chain security

### 7. **Granular Approval Change Events** 📡

**Event Structure (Enhanced):**
```json
{
  "type": "approval_changed",
  "attributes": [
    {"key": "collection_id", "value": "1"},
    {"key": "approver", "value": "bitbadges1..."},
    {"key": "spender", "value": "bitbadges1..."},
    {"key": "old_amount", "value": "100"},
    {"key": "new_amount", "value": "200"},
    {"key": "change_type", "value": "increase"},
    {"key": "timestamp", "value": "2026-04-17T05:30:28Z"},
    {"key": "tx_hash", "value": "ABC123..."}
  ]
}
```

**Monitoring:**
```bash
# Approval change events izleme
journalctl -u bitbadgeschaind -f | grep "approval_changed"

# Event history query
bitbadgeschaind query txs --events 'approval_changed.collection_id=1'
```

**Audit Trail:**
- Complete change history
- Who changed what when
- Before/after values
- Better compliance

### 8. **Enriched Msg Responses** 📬

**Enhanced Response Data:**
```json
{
  "code": 0,
  "data": "0A1E0A1C2F...",
  "raw_log": "[{\"events\":[...]}]",
  "logs": [...],
  "info": "",
  "gas_wanted": "200000",
  "gas_used": "150000",
  "tx": {...},
  "timestamp": "2026-04-17T05:30:28Z",
  // NEW: Enhanced metadata
  "metadata": {
    "affected_collections": ["1", "2"],
    "affected_users": ["bitbadges1...", "bitbadges1..."],
    "state_changes": {
      "balances_updated": 5,
      "approvals_changed": 2
    },
    "performance_metrics": {
      "execution_time_ms": 250,
      "db_queries": 12
    }
  }
}
```

**Developer Benefits:**
- Better error messages
- More context
- Performance insights
- Easier debugging

### 9. **CLI/Tooling Improvements** 🛠️

**New Commands:**
```bash
# User approval settings
bitbadgeschaind tx tokenization set-user-approval-settings
bitbadgeschaind query tokenization user-approval-settings

# Enhanced queries
bitbadgeschaind query tokenization collection --format json
bitbadgeschaind query tokenization balances --output table

# Batch operations
bitbadgeschaind tx tokenization batch-transfer
bitbadgeschaind tx tokenization batch-approve

# Developer tools
bitbadgeschaind debug decode-tx
bitbadgeschaind debug encode-tx
bitbadgeschaind debug addr
```

**Improved Help:**
```bash
# Daha detaylı help messages
bitbadgeschaind tx tokenization --help
bitbadgeschaind query tokenization --help

# Examples in help
bitbadgeschaind tx tokenization transfer --help
# Shows: Usage examples, common patterns, error handling
```

### 10. **Cross-Platform Release CI** 🌐

**Supported Platforms:**
- Linux (amd64, arm64)
- macOS (amd64, arm64/M1)
- Windows (amd64)

**Automated Releases:**
- GitHub Actions CI/CD
- Automated builds
- Checksums verification
- Multi-platform testing

**Download:**
```bash
# Linux AMD64
wget https://github.com/BitBadges/bitbadgeschain/releases/download/v29/bitbadgeschain-linux-amd64

# Linux ARM64
wget https://github.com/BitBadges/bitbadgeschain/releases/download/v29/bitbadgeschain-linux-arm64

# macOS M1
wget https://github.com/BitBadges/bitbadgeschain/releases/download/v29/bitbadgeschain-darwin-arm64
```

---

## 🎯 Use Cases ve Örnekler

### Voting Challenge Örneği:

```bash
# Proposal oluştur with enhanced voting
bitbadgeschaind tx gov submit-proposal \
  --title "Enable New Feature" \
  --description "..." \
  --type Text \
  --deposit 10000000ubadge \
  --voting-params '{
    "voting_period": "172800s",
    "quorum": "0.4",
    "threshold": "0.5",
    "execution_delay": "86400s",
    "auto_reset": true
  }' \
  --from wallet
```

### UserApprovalSettings Örneği:

```bash
# Strict approval settings
bitbadgeschaind tx tokenization set-user-approval-settings \
  --approval-level strict \
  --require-signature true \
  --max-amount-per-tx 1000 \
  --whitelist "bitbadges1trusted...,bitbadges1friend..." \
  --from wallet
```

### Time-Based Restrictions Örneği:

```bash
# Business hours only collection
{
  "timeRestrictions": {
    "recurringSchedule": {
      "daysOfWeek": [1, 2, 3, 4, 5],
      "hoursOfDay": [9, 17],
      "timezone": "UTC"
    }
  }
}
```

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v29
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/38
- **CLI Documentation:** Enhanced help commands
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın

---

## 🔄 Upgrade Timeline

- **Proposal Submitted:** ✅ Completed
- **Voting Period:** Now + 24 hours
- **Upgrade Time:** 17 Nisan 2026, 05:30:28 (Türkiye Saati)
- **Expected Downtime:** ~5 dakika (no migration)
- **Preparation:** Start NOW
- **Backup:** Recommended

---

**Not:** Bu rehber, v28'den v29'a geçiş için hazırlanmıştır. Governance, approvals ve developer experience odaklı kapsamlı iyileştirmeler içerir. State migration olmadığı için hızlı bir upgrade olacak.
