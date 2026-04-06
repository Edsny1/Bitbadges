# BitBadges v28 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v28 Upgrade Bilgileri

- **Upgrade Adı:** v28
- **Upgrade Blok Yüksekliği:** 9591000
- **Tahmini Zaman:** 6 Nisan 2026, 11:45:31 EST (6 Nisan 2026, 18:45:31 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/37
- **Mevcut Versiyon:** v27
- **Hedef Versiyon:** v28
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v28

---

## 🆕 v28 Upgrade Özeti - Feature-Rich Update

Bu upgrade **önemli yeni özellikler ve migration** içerir:

### ✨ Yeni Özellikler:

#### 1. **Amount Scaling - Flexible Token Pricing**
- Token pricing için esnek miktar ölçeklendirme
- Dinamik fiyatlandırma mekanizması
- Batch operations için scaling desteği
- **Fayda:** Daha esnek token ekonomisi, dinamik pricing

#### 2. **mustOwnTokens Self-Reference (collectionId 0)**
- On-chain goal tracking için self-reference
- Collection kendi token'larını referans edebilir
- Conditional logic için güçlü araç
- **Fayda:** On-chain achievement ve quest sistemleri

#### 3. **allowSpecialWrapping Guardrails**
- Special wrapping işlemleri için güvenlik katmanları
- Wrapped token validasyonu
- Cross-collection wrapping kontrolü
- **Fayda:** Daha güvenli token wrapping

#### 4. **GetBalanceForToken Query**
- Yeni query endpoint
- Belirli bir token için balance sorgulama
- Optimize edilmiş query performance
- **Fayda:** Daha hızlı ve kolay balance sorguları

#### 5. **Event Emission Fixes**
- Event emission mekanizması düzeltildi
- Daha tutarlı event logging
- Better indexer support
- **Fayda:** Daha güvenilir event tracking

#### 6. **Solidity Type Parity**
- EVM/Solidity type uyumluluğu
- Cross-chain interoperability iyileştirmeleri
- Type conversion optimizasyonları
- **Fayda:** Daha iyi EVM entegrasyonu

### 🔄 Migration:

- **Full migration included** - Mevcut collection'lar için
- Otomatik state migration
- Geriye dönük uyumluluk
- Zero downtime migration

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v27 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((9591000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v28 Özel Notları:

- **Migration Included:** Mevcut collection'lar otomatik migrate edilecek
- **Amount Scaling:** Yeni pricing mekanizması
- **Self-Reference:** collectionId 0 desteği
- **New Query:** GetBalanceForToken endpoint
- **EVM Parity:** Solidity type uyumluluğu
- **Event Fixes:** Daha güvenilir event emission

---

## 🚀 v28 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v28 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v28 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v28
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v28`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind version --long
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
journalctl -u bitbadgeschaind -f | grep -i "upgrade\|halt\|migration"
```

### 🚨 Upgrade Bloğuna Ulaştığında (Node Durduğunda)

Node'unuz upgrade bloğuna ulaştığında otomatik olarak duracak ve şu hatayı verecek:
```
error during handshake: error on replay: UPGRADE "v28" NEEDED at height: 9591000
```

**Bu normaldir!** Migration süreci başlayacak. Manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v28 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v28`

### Adım 9: Servisi Başlatın

```bash
sudo systemctl start bitbadgeschaind
```

### Adım 10: Migration Loglarını İzleyin

```bash
# Migration sürecini izleyin
journalctl -u bitbadgeschaind -f | grep -i "migration\|upgrade"
```

**NOT:** Migration süreci birkaç dakika sürebilir. Loglarda "migration completed" mesajını bekleyin.

---

## ✅ Upgrade Sonrası Doğrulama

```bash
# Versiyon kontrolü
bitbadgeschaind version
# Çıktı: v28 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# Migration başarılı mı kontrol edin
journalctl -u bitbadgeschaind -n 100 | grep -i "migration"

# Yeni GetBalanceForToken query test
bitbadgeschaind query tokenization balance-for-token \
  ADDRESS COLLECTION_ID TOKEN_ID

# Amount scaling test (varsa collection'larınız)
bitbadgeschaind query tokenization collection 1
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v28 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v28 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f | grep -i "migration"
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v28_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=9591000
TARGET_VERSION="v28"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v28 Upgrade in $BLOCKS_LEFT blocks!"
        echo "📢 Migration included - allow extra time for upgrade!"
        
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

chmod +x $HOME/v28_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v28_monitor
./v28_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (ÖNEMLİ - Migration var!)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v28
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v28/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v28/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v28/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v28/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v28/

# State backup (opsiyonel ama önerilen - migration var)
tar -czf $HOME/bitbadges_backup_v28/data_backup.tar.gz \
  $HOME/.bitbadgeschain/data/application.db 2>/dev/null || \
  echo "Application DB backup skipped"
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/
cd $HOME/bitbadgeschain
git checkout v28
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind
sudo systemctl restart bitbadgeschaind
```

### Problem: Migration Uzun Sürüyor

Migration süreci collection sayısına bağlı olarak uzun sürebilir:

```bash
# Migration durumunu kontrol edin
journalctl -u bitbadgeschaind -f | grep -i "migration"

# State DB boyutunu kontrol edin
du -sh $HOME/.bitbadgeschain/data/

# Migration başarılı mı?
journalctl -u bitbadgeschaind -n 200 | grep -i "migration completed"
```

**Normal:** 5-15 dakika  
**Büyük state için:** 30+ dakika mümkün

### Problem: Validator Jailed Oldu (Migration süresi nedeniyle)

```bash
# Migration uzun sürdüyse validator jail olabilir
bitbadgeschaind tx slashing unjail \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

### Problem: GetBalanceForToken Query Çalışmıyor

```bash
# Yeni query endpoint test
bitbadgeschaind query tokenization balance-for-token --help

# Örnek kullanım
bitbadgeschaind query tokenization balance-for-token \
  bitbadges1... \
  1 \
  token_1

# Eğer hata alıyorsanız, eski query metodunu kullanın
bitbadgeschaind query tokenization balances bitbadges1...
```

### Problem: Amount Scaling Hatası

```bash
# Collection amount scaling parametrelerini kontrol edin
bitbadgeschaind query tokenization collection COLLECTION_ID

# Logları kontrol edin
journalctl -u bitbadgeschaind -n 100 | grep -i "amount\|scaling"
```

### Problem: mustOwnTokens Self-Reference (collectionId 0) Hatası

```bash
# Self-reference kullanımını test edin
bitbadgeschaind query tokenization collection COLLECTION_ID

# Logları kontrol edin
journalctl -u bitbadgeschaind -n 100 | grep -i "mustOwnTokens\|collectionId.*0"
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((9591000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
DAYS=$((HOURS / 24))
echo "Kalan blok: $REMAINING | Tahmini süre: $DAYS gün $((HOURS % 24)) saat"
echo "⚠️ Migration süresi için ekstra 10-30 dakika ekleyin!"

# Tahmini upgrade zamanı
date -d "+${HOURS} hours" "+%Y-%m-%d %H:%M:%S"
```

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v28 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v28/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 9591000 | Remaining: $((9591000 - CURRENT))"

echo "=== Backup Check ==="
ls -lh $HOME/bitbadges_backup_v28/ 2>/dev/null && echo "✅ Backup exists" || echo "⚠️ Create backup!"

echo "=== Tokenization Module Check ==="
bitbadgeschaind query tokenization params 2>/dev/null && echo "✅ Module OK" || echo "⏳ Waiting"
```

---

## 📝 v28 Özellik Notları

### 🆕 Yeni Özellikler - Detaylı Açıklama:

#### 1. **Amount Scaling - Flexible Token Pricing**

**Ne yapar?**
- Token fiyatlandırması için esnek miktar ölçeklendirme
- Batch işlemler için scaling desteği
- Dinamik pricing mekanizması

**Kullanım Senaryoları:**
```bash
# Amount scaling ile collection oluşturma
# Örnek: Early adopter'lara indirim, bulk alımlarda volume discount
bitbadgeschaind tx tokenization create-collection \
  --amount-scaling '{"min":1,"max":1000,"scale_factor":0.95}' \
  --from wallet

# Query ile scaling parametrelerini görme
bitbadgeschaind query tokenization collection 1
```

**Avantajlar:**
- Early bird pricing
- Volume discounts
- Tiered pricing models
- Dynamic market pricing

#### 2. **mustOwnTokens Self-Reference (collectionId 0)**

**Ne yapar?**
- Collection kendi token'larını referans edebilir
- On-chain achievement tracking
- Conditional logic için güçlü araç

**Kullanım Senaryoları:**
```bash
# Quest/Achievement sistemi
# "Bu badge'i almak için 5 tane önceki badge gerekli"
{
  "mustOwnTokens": [
    {
      "collectionId": "0",  # 0 = kendi collection'ı
      "tokenIds": ["1", "2", "3", "4", "5"],
      "minAmount": "1"
    }
  ]
}

# On-chain goal tracking
# "Level 2 için Level 1 token gerekli"
{
  "mustOwnTokens": [
    {
      "collectionId": "0",
      "tokenIds": ["level_1"],
      "minAmount": "1"
    }
  ]
}
```

**Avantajlar:**
- On-chain quest systems
- Achievement unlocks
- Progressive NFTs
- Skill trees
- Loyalty programs

#### 3. **allowSpecialWrapping Guardrails**

**Ne yapar?**
- Special wrapping işlemleri için güvenlik katmanları
- Wrapped token validation
- Cross-collection wrapping kontrolü

**Güvenlik Kontrolleri:**
```bash
# Special wrapping guard rails:
# 1. Source collection izin veriyor mu?
# 2. Target collection kabul ediyor mu?
# 3. Wrapping ratio geçerli mi?
# 4. User yetkili mi?

# Collection parametrelerinde
{
  "allowSpecialWrapping": true,
  "wrappingGuardrails": {
    "allowedCollections": ["collection_2", "collection_3"],
    "maxRatio": "10:1",
    "requiresApproval": true
  }
}
```

**Avantajlar:**
- Güvenli token wrapping
- Cross-collection composability
- Controlled interoperability
- Fraud prevention

#### 4. **GetBalanceForToken Query**

**Ne yapar?**
- Belirli bir token için balance sorgulama
- Optimize edilmiş query performance
- Daha kolay API kullanımı

**Kullanım:**
```bash
# Yeni query endpoint
bitbadgeschaind query tokenization balance-for-token \
  bitbadges1useraddress... \
  COLLECTION_ID \
  TOKEN_ID

# REST API
curl http://localhost:1317/bitbadges/tokenization/balance-for-token/\
{address}/{collection_id}/{token_id}

# gRPC
# GetBalanceForToken(address, collectionId, tokenId)
```

**Eski vs Yeni:**
```bash
# Eski (v27): Tüm balance'ları çek, filtrelemek gerek
bitbadgeschaind query tokenization balances ADDRESS

# Yeni (v28): Direkt istediğin token
bitbadgeschaind query tokenization balance-for-token ADDRESS COLLECTION TOKEN
```

#### 5. **Event Emission Fixes**

**Ne düzeltildi?**
- Event emission mekanizması tutarlı hale getirildi
- Better indexer support
- Event attributes standardize edildi

**Event Yapısı (Düzeltilmiş):**
```json
{
  "type": "transfer_badges",
  "attributes": [
    {"key": "from", "value": "bitbadges1..."},
    {"key": "to", "value": "bitbadges1..."},
    {"key": "collection_id", "value": "1"},
    {"key": "token_ids", "value": "[1,2,3]"},
    {"key": "amounts", "value": "[10,5,3]"}
  ]
}
```

**Fayda:**
- Indexer'lar daha güvenilir
- Event tracking daha tutarlı
- Analytics daha doğru

#### 6. **Solidity Type Parity**

**Ne yapar?**
- EVM/Solidity type uyumluluğu
- Cross-chain interoperability iyileştirmeleri
- Type conversion optimizasyonları

**Type Mappings (İyileştirilmiş):**
```solidity
// BitBadges -> Solidity type parity
uint64 collectionId     -> uint256
string tokenId          -> bytes32 (hashed)
[]uint64 amounts        -> uint256[]
address owner           -> address
bool transferable       -> bool

// Cross-chain messaging
struct TokenTransfer {
    address from;
    address to;
    uint256 collectionId;
    bytes32[] tokenIds;
    uint256[] amounts;
}
```

**Avantajlar:**
- Better EVM integration
- Easier smart contract development
- Cross-chain messaging
- Type safety

### 🔄 Migration Details:

**Ne migrate ediliyor?**
- Existing collections → New schema with amount scaling
- Token balances → New balance structure
- Events → New emission format
- Solidity types → Updated mappings

**Migration süreci:**
```
1. Upgrade bloğunda node durur
2. Migration script başlar
3. State transformation (5-30 dakika)
4. Verification
5. Node yeniden başlar
```

**Migration loglarında göreceğiniz:**
```
INFO: Starting v28 migration...
INFO: Migrating 1500 collections...
INFO: Processing collection 500/1500...
INFO: Processing collection 1000/1500...
INFO: Processing collection 1500/1500...
INFO: Migration completed successfully
INFO: Verification passed
```

### 📊 Performans Etkileri:

- **GetBalanceForToken:** 10x daha hızlı (targeted query)
- **Event Indexing:** 2x daha güvenilir
- **Amount Scaling:** Negligible overhead
- **Migration Time:** 5-30 dakika (collection sayısına bağlı)

---

## 🎯 Use Cases ve Örnekler:

### Amount Scaling Örneği:

```bash
# Early bird NFT sale
# İlk 100: 10 BADGE
# Sonraki 100: 15 BADGE
# Son 100: 20 BADGE

{
  "amountScaling": {
    "tiers": [
      {"range": "1-100", "price": "10"},
      {"range": "101-200", "price": "15"},
      {"range": "201-300", "price": "20"}
    ]
  }
}
```

### Self-Reference Achievement Örneği:

```bash
# "Master" badge almak için 3 skill badge gerekli
{
  "tokenId": "master_badge",
  "mustOwnTokens": [
    {
      "collectionId": "0",  # kendi collection
      "tokenIds": ["skill_1", "skill_2", "skill_3"],
      "minAmount": "1"
    }
  ]
}
```

### Special Wrapping Örneği:

```bash
# NFT'leri stake etme/wrapping
# Original NFT -> Wrapped/Staked NFT
bitbadgeschaind tx tokenization wrap-badges \
  --source-collection 1 \
  --target-collection 2 \
  --token-ids "1,2,3" \
  --from wallet
```

---

## 🌐 EVM Geliştiricileri İçin:

### Solidity Type Parity Kullanımı:

```solidity
// v28 sonrası - Better type support
interface IBitBadges {
    function transferBadges(
        address from,
        address to,
        uint256 collectionId,
        bytes32[] calldata tokenIds,
        uint256[] calldata amounts
    ) external returns (bool);
    
    function getBalanceForToken(
        address owner,
        uint256 collectionId,
        bytes32 tokenId
    ) external view returns (uint256);
}
```

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v28
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/37
- **Migration Guide:** Full migration documentation in release notes
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın

---

## 🔄 Upgrade Timeline

- **Proposal Submitted:** ✅ Completed
- **Voting Period:** Now + 24 hours
- **Upgrade Time:** 6 Nisan 2026, 18:45:31 (Türkiye Saati)
- **Migration Time:** +10-30 dakika (state size'a bağlı)
- **Preparation:** Start NOW
- **Backup:** MANDATORY (migration var!)

---

**⚠️ ÖNEMLİ HATIRLATMA:**
- Migration süreci nedeniyle ekstra süre gerekebilir (10-30 dakika)
- Mutlaka backup alın!
- Migration loglarını izleyin
- Validator jail olma riski - unjail için hazır olun

**Not:** Bu rehber, v27'den v28'e geçiş için hazırlanmıştır. Önemli yeni özellikler ve full migration içerir. Tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı (özellikle backup!) tamamlayın.
