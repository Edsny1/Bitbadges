# BitBadges v25 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: HIZLI PATCH UPGRADE - <48 SAAT!

Bu bir **acil patch upgrade**'dir! Normal 4 günlük süreç yerine **48 saatten kısa** sürede gerçekleşecek. v24'teki bazı kritik sorunları düzeltmek için hızlı uygulama yapılıyor.

**Hazırlıklarınızı hemen yapın!**

---

## 📋 v25 Upgrade Bilgileri

- **Upgrade Adı:** v25
- **Upgrade Blok Yüksekliği:** 8944000
- **Tahmini Zaman:** 28 Şubat 2026, 10:58:09 EST (17:58:09 Türkiye Saati)
- **Kalan Süre:** ~2 gün
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/32
- **Mevcut Versiyon:** v24
- **Hedef Versiyon:** v25
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v25

---

## 🚨 v25 HIZLI PATCH UPGRADE - Kritik Bilgiler

### Bu Neden Hızlı Bir Upgrade?

v24 upgrade'inden sonra bazı kritik sorunlar tespit edildi ve hızla düzeltilmesi gerekiyor:

### 🔧 Düzeltilen Kritik Sorunlar:

1. **IBC Sorunu Düzeltildi**
   - IBC v8'den v10'a geçiş sırasında eski client'lar kayıt olmuyordu
   - "route not found" hataları düzeltildi
   - Relayer'lar için kritik düzeltme

2. **Eski Gov Proposal Query Sorunu**
   - Eski message type'lı gov proposal'ları sorgularken hata veriyordu
   - Explorer'lar için kritik düzeltme
   - Şimdi düzeltildi

3. **Coin Type Değişikliği (KALICI)**
   - Coin type: 118 → 60'a değişti
   - **ÖNEMLİ:** Eski adres kurtarma için `--hd-path "m/44'/118'/0'/0/0"` kullanın
   - Bu değişiklik kalıcıdır

### ✨ Yeni Özellikler:

- **Custom EVM Queries:** Approvals ve invariants için read-only EVM sorguları
- **Tokenization Precompile İyileştirmeleri:** Return type'lar basitleştirildi, daha kolay entegrasyon
- **Collection Stats Tracking:** Yeni GetCollectionStats query ve precompile method

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v24 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı

# Kalan blok sayısı
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Kalan blok: $((8944000 - CURRENT))"
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Current link değiştirme ve restart

---

## ⚠️ v25 Özel Notları:

- **HIZLI UPGRADE:** <48 saat içinde gerçekleşecek, hemen hazırlanın!
- **IBC Düzeltmesi:** Relayer çalıştırıyorsanız mutlaka upgrade edin
- **Coin Type Değişti (118 → 60):** Eski adresler için özel flag gerekli
- **Gov Query Düzeltmesi:** Explorer'lar düzgün çalışacak
- **EVM İyileştirmeleri:** Daha kolay entegrasyon

---

## 🚀 v25 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v25 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v25 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v25
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v25`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v25" NEEDED at height: 8944000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v25 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v25`

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
# Çıktı: v25 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind

# IBC çalışıyor mu (Relayer çalıştırıyorsanız)
bitbadgeschaind query ibc client states --limit 5
```

---

## 🔑 Coin Type Değişikliği - ÖNEMLİ!

### Coin Type: 118 → 60 (KALICI DEĞİŞİKLİK)

Bu değişiklik **kalıcıdır** ve tüm gelecek versiyonlarda geçerli olacak.

### Eski Adres Kurtarma (Coin Type 118)

Eğer eski mnemonic'lerle adres kurtarmak istiyorsanız:

```bash
# Eski coin type (118) ile adres kurtarma
bitbadgeschaind keys add cuzdan-adi \
  --recover \
  --hd-path "m/44'/118'/0'/0/0"

# Mnemonic'i girin ve eski adresiniz oluşacak
```

### Yeni Adres Oluşturma (Coin Type 60)

v25 ve sonrası için yeni adresler:

```bash
# Yeni coin type (60) ile adres oluşturma - varsayılan
bitbadgeschaind keys add yeni-cuzdan

# Veya açıkça belirtmek isterseniz
bitbadgeschaind keys add yeni-cuzdan \
  --hd-path "m/44'/60'/0'/0/0"
```

### Adres Kontrolü

```bash
# Tüm adreslerinizi listeleyin
bitbadgeschaind keys list

# Belirli bir adresin bilgilerini görün
bitbadgeschaind keys show cuzdan-adi
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v25 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v25 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v25_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=8944000
TARGET_VERSION="v25"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v25 FAST PATCH in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v25_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v25_monitor
./v25_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v25
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v25/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v25/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v25/
cp $HOME/.bitbadgeschain/config/app.toml $HOME/bitbadges_backup_v25/
cp $HOME/.bitbadgeschain/config/config.toml $HOME/bitbadges_backup_v25/

# Keyring backup (önerilir)
bitbadgeschaind keys export cuzdan-adi > $HOME/bitbadges_backup_v25/wallet_export.key
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/
cd $HOME/bitbadgeschain
git checkout v25
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind
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

### Problem: IBC Relayer "route not found" Hatası

Bu sorun v25 ile düzeltildi. Upgrade sonrası:

```bash
# Relayer'ı yeniden başlatın
systemctl restart relayer-service

# IBC client'ları kontrol edin
bitbadgeschaind query ibc client states

# IBC bağlantıları kontrol edin
bitbadgeschaind query ibc connection connections
```

### Problem: Eski Adres Kurtarılamıyor

Coin type değişti (118 → 60). Eski adres kurtarma için:

```bash
bitbadgeschaind keys add eski-cuzdan \
  --recover \
  --hd-path "m/44'/118'/0'/0/0"
```

### Problem: Gov Proposal Query Hatası

v25 ile düzeltildi. Eski proposal'ları sorgulamak için:

```bash
# Tüm proposal'ları listele
bitbadgeschaind query gov proposals

# Belirli bir proposal
bitbadgeschaind query gov proposal 32
```

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((8944000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
echo "Kalan blok: $REMAINING | Tahmini süre: $HOURS saat"

# Tahmini upgrade zamanı
date -d "+${HOURS} hours" "+%Y-%m-%d %H:%M:%S"
```

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v25 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v25/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 8944000 | Remaining: $((8944000 - CURRENT))"

echo "=== Coin Type Check ==="
echo "New coin type: 60 (default)"
echo "Old coin type: 118 (use --hd-path flag)"
```

---

## 📝 v25 Özellik Notları

### 🔥 Kritik Düzeltmeler (Hızlı Patch Nedeni):

1. **IBC Düzeltmesi**
   - IBC v8 → v10 geçişi sırasındaki eski client kayıt sorunu düzeltildi
   - "route not found" hataları giderildi
   - Relayer'lar sorunsuz çalışacak

2. **Gov Query Düzeltmesi**
   - Eski message type'lı proposal'ları sorgularken olan hata düzeltildi
   - Explorer'lar artık tüm proposal'ları gösterebilecek

3. **Coin Type Değişikliği (KALICI)**
   - 118 → 60'a geçiş **kalıcı**
   - Eski adresler için: `--hd-path "m/44'/118'/0'/0/0"`
   - Yeni adresler için: `--hd-path "m/44'/60'/0'/0/0"` (varsayılan)

### ✨ Yeni Özellikler:

1. **Custom EVM Queries**
   - Read-only EVM sorguları approvals için
   - Read-only EVM sorguları invariants için
   - Daha iyi EVM entegrasyonu

2. **Tokenization Precompile İyileştirmeleri**
   - Return type'lar basitleştirildi
   - Daha kolay EVM entegrasyonu
   - Daha temiz API

3. **Collection Stats Tracking**
   - Yeni `GetCollectionStats` query
   - Yeni precompile method
   - Daha detaylı istatistikler

### ⏱️ Upgrade Zamanlaması:

- **Normal upgrade:** 4 gün sürer
- **v25 Fast Patch:** <48 saat
- **Neden hızlı:** Kritik IBC ve gov query sorunları

### 🎯 Etkilenen Kullanıcılar:

- **Relayer çalıştıranlar:** Mutlaka upgrade edin (IBC düzeltmesi)
- **Explorer operatörleri:** Mutlaka upgrade edin (gov query düzeltmesi)
- **Eski adres kullananlar:** Coin type değişikliğini not edin
- **EVM geliştiricileri:** Yeni query ve precompile özelliklerinden faydalanın

---

## 🚨 Relayer Operatörleri İçin Özel Notlar

### IBC Düzeltmesi

v24'te IBC v8'den v10'a geçiş sırasında sorun yaşandıysa:

```bash
# 1. v25'e upgrade edin
# 2. Relayer'ı yeniden başlatın
sudo systemctl restart relayer

# 3. IBC durumunu kontrol edin
bitbadgeschaind query ibc client states
bitbadgeschaind query ibc connection connections
bitbadgeschaind query ibc channel channels

# 4. Relayer loglarını kontrol edin
journalctl -u relayer -f
```

### Relayer Yapılandırması

Hermes veya rly kullanıyorsanız:

```bash
# Hermes
hermes health-check
hermes query channels --chain bitbadges-1

# rly
rly chains list
rly paths list
```

---

## 📚 Ek Kaynaklar

- **Release Notes:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v25
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/32
- **Discord:** Destek için BitBadges Discord kanalına katılın
- **Telegram:** BitBadges Telegram grubuna katılın

---

**⚠️ HATIRLATMA: Bu hızlı bir patch upgrade'dir (<48 saat). Hazırlıklarınızı hemen yapın!**

**Not:** Bu rehber, v24'ten v25'e geçiş için hazırlanmıştır. Kritik IBC ve gov query düzeltmeleri içerir. Tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı tamamlayın.
