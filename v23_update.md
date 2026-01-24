# BitBadges v23 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v23 Upgrade Bilgileri

- **Upgrade Adı:** v23
- **Upgrade Blok Yüksekliği:** 8400000
- **Tahmini Zaman:** 28 Ocak 2026, 13:58:28 EST (20:58:28 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/30
- **Mevcut Versiyon:** v22
- **Hedef Versiyon:** v23
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v23

---

## 🆕 v23 Upgrade Özeti

Bu upgrade şu değişiklikleri içerir:

- **Alt-chain imza desteği kaldırıldı**
- **allowBackedMinting** ve **allowSpecialWrapping** bayrakları eklendi (özel işlemler için aktive edilmeli)
- **ownershipCheckParty** dinamik store'lar için eklendi
- **v22 patch otomatik uygulanıyor** - ekstra işlem gerekmez

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v22 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı
```

---

## ✅ İki Aşamalı Yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Sadece current link değiştirme ve restart

---

## ✅ Önceki Sorunlardan Dersler:

- Manuel current link oluşturma adımı standart prosedür olarak eklendi
- Cosmovisor'ın otomatik geçiş yapamayacağı varsayılarak hazırlandı
- Hızlı komut özeti eklendi
- v22 patch otomatik uygulanıyor, ekstra işlem gerekmez

---

## ✅ Monitoring ve Uyarı Sistemi:

- Upgrade bloğunu izleme script'i
- Blok yüksekliğini sürekli kontrol
- 100 ve 50 blok kala uyarı

---

## 🚀 v23 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v23 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v23 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v23
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v23`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v23" NEEDED at height: 8400000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v23 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v23`

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
# Çıktı: v23 olmalı

# Node durumunu kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo

# Servis durumunu kontrol edin
sudo systemctl status bitbadgeschaind
```

---

## 📊 Hızlı Komut Özeti

### Upgrade Öncesi Hazırlık (ŞİMDİ yapın):

```bash
cd $HOME && rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain && git checkout v23 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v23 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v23_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=8400000
TARGET_VERSION="v23"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v23 Upgrade in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v23_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v23_monitor
./v23_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v23
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v23/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v23/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v23/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/
cd $HOME/bitbadgeschain
git checkout v23
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind
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

---

## 📈 Upgrade Zamanlaması

```bash
# Kalan blok ve tahmini süre
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
REMAINING=$((8400000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
echo "Kalan blok: $REMAINING | Tahmini süre: $HOURS saat"
```

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v23 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v23/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 8400000 | Remaining: $((8400000 - CURRENT))"
```

---

## 📝 v23 Özellik Notları

### Yeni Özellikler:
- **Alt-chain imza desteği kaldırıldı** - Artık sadece native chain imzaları kullanılıyor
- **allowBackedMinting bayrağı** - Backed token basımı için özel izin gerekiyor
- **allowSpecialWrapping bayrağı** - Özel wrapping işlemleri için özel izin gerekiyor
- **ownershipCheckParty** - Dinamik store'lar için ownership kontrolü eklendi
- **v22 patch otomatik uygulanıyor** - v22'den gelen tüm düzeltmeler dahil

### Önemli:
Bu upgrade v22 patch'i de içerdiği için, v22'den v23'e geçiş yaparken ek işlem yapmanıza gerek yoktur. Standart upgrade prosedürünü takip edin.

---

**Not:** Bu rehber, v22'den v23'e geçiş için hazırlanmıştır. Tüm adımları dikkatlice takip edin ve upgrade bloğuna ulaşmadan önce hazırlıklarınızı tamamlayın.
