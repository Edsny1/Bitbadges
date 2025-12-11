Görüntüdeki formata göre hazırlıyorum. İşte tek tıklamayla kopyalayabileceğiniz, GitHub'a ekleyebileceğiniz tam dokümantasyon:

---

# BitBadges v21 Upgrade Hazırlık Rehberi

## ⚠️ ÖNEMLİ: Upgrade Bloğuna Ulaşmadan Önce Hazırlık Yapın!

Bu talimatları **upgrade bloğuna ulaşmadan ÖNCE** tamamlayın. Upgrade bloğuna ulaştığında sadece manuel geçiş adımlarını (Adım 6-9) uygulayın.

---

## 📋 v21 Upgrade Bilgileri

- **Upgrade Adı:** v21
- **Upgrade Blok Yüksekliği:** 7510000
- **Tahmini Zaman:** 14 Aralık 2025, 10:52:18 AM EST (17:52:18 Türkiye Saati)
- **Oylama Dönemi:** Şimdi + 24 saat
- **Governance Proposal:** https://explorer.oshvank.xyz/BitBadges%20Mainnet/gov/28
- **Mevcut Versiyon:** v20
- **Hedef Versiyon:** v21
- **Release Bilgileri:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v21

---

## 🔍 Ön Kontroller

### Mevcut Durumu Kontrol Edin

```bash
# Mevcut versiyonu kontrol edin
bitbadgeschaind version
# Çıktı: v20 olmalı

# Mevcut blok yüksekliğini kontrol edin
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'

# Node senkronize mi kontrol edin
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
# Çıktı: false olmalı
```

---

## ✅ İki aşamalı yaklaşım:

### ✅ Önceden hazırlık (node çalışırken) - Binary'yi hazırlayıp upgrade dizinine koyma

### ✅ Upgrade bloğunda geçiş (node durduğunda) - Sadece current link değiştirme ve restart

---

## ✅ Önceki sorunlardan dersler:

- Manuel current link oluşturma adımı standart prosedür olarak eklendi
- Cosmovisor'ın otomatik geçiş yapamayacağı varsayılarak hazırlandı
- Hızlı komut özeti eklendi

---

## ✅ Monitoring ve uyarı sistemi:

- Upgrade bloğunu izleme script'i
- Blok yüksekliğini sürekli kontrol
- 100 ve 50 blok kala uyarı

---

## 🚀 v21 Upgrade Hazırlık Adımları

### ⚠️ DİKKAT: Upgrade bloğuna ulaşmadan ÖNCESİNDE yapın!

### Adım 1: v21 Binary'sini Hazırlayın

**Not:** Servisi DURDURMADAN yapın, node çalışmaya devam etsin.

```bash
# v21 kaynak kodunu indirin
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v21
make build-linux/amd64
```

### Adım 2: Upgrade Dizinini Oluşturun

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin
```

### Adım 3: Binary'yi Kopyalayın

```bash
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
```

### Adım 4: Binary Versiyonunu Doğrulayın

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v21`

### Adım 5: Hazırlığın Tamamlandığını Doğrulayın

```bash
# Upgrade dizinini kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/

# Binary'nin çalıştırılabilir olduğunu doğrulayın
$HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind version --long
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
error during handshake: error on replay: UPGRADE "v21" NEEDED at height: 7510000
```

**Bu normaldir!** Şimdi manuel geçişi yapın:

### Adım 6: Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

### Adım 7: Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v21 $HOME/.bitbadgeschain/cosmovisor/current
```

### Adım 8: Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v21`

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
# Çıktı: v21 olmalı

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
cd bitbadgeschain && git checkout v21 && make build-linux/amd64
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
$HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind version
```

### Upgrade Bloğunda (Node durduğunda yapın):

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v21 $HOME/.bitbadgeschain/cosmovisor/current
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

---

## 🔔 Monitoring Script (Opsiyonel)

```bash
cat > $HOME/v21_upgrade_monitor.sh << 'EOF'
#!/bin/bash

UPGRADE_HEIGHT=7510000
TARGET_VERSION="v21"

while true; do
    CURRENT_HEIGHT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
    CURRENT_VERSION=$(bitbadgeschaind version 2>/dev/null)
    
    echo "[$(date)] Height: $CURRENT_HEIGHT | Version: $CURRENT_VERSION"
    
    if [ "$CURRENT_HEIGHT" -ge $((UPGRADE_HEIGHT - 100)) ] && [ "$CURRENT_HEIGHT" -lt "$UPGRADE_HEIGHT" ]; then
        BLOCKS_LEFT=$((UPGRADE_HEIGHT - CURRENT_HEIGHT))
        echo "⚠️  WARNING: v21 Upgrade in $BLOCKS_LEFT blocks!"
        
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

chmod +x $HOME/v21_upgrade_monitor.sh

# Tmux içinde çalıştırın
tmux new -s v21_monitor
./v21_upgrade_monitor.sh
# CTRL+B sonra D ile detach
```

---

## 🛡️ Güvenlik ve Backup

### Upgrade Öncesi Backup (Önerilen)

```bash
# Önemli dosyaları yedekleyin
mkdir -p $HOME/bitbadges_backup_v21
cp $HOME/.bitbadgeschain/config/priv_validator_key.json $HOME/bitbadges_backup_v21/
cp $HOME/.bitbadgeschain/config/node_key.json $HOME/bitbadges_backup_v21/
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/bitbadges_backup_v21/
```

---

## ⚠️ Sorun Giderme

### Problem: "binary not found" Hatası

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/
cd $HOME/bitbadgeschain
git checkout v21
make build-linux/amd64
cp build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
```

### Problem: "permission denied" Hatası

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind
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
REMAINING=$((7510000 - CURRENT))
HOURS=$((REMAINING * 6 / 3600))
echo "Kalan blok: $REMAINING | Tahmini süre: $HOURS saat"
```

---

## 📞 Destek ve İletişim

- **Discord:** https://discord.gg/bitbadges
- **GitHub:** https://github.com/BitBadges/bitbadgeschain
- **Release:** https://github.com/BitBadges/bitbadgeschain/releases/tag/v21
- **Governance:** https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/28

---

## 📝 Checklist

- [ ] Mevcut versiyon v20
- [ ] Node senkronize (catching_up: false)
- [ ] v21 binary'si hazır
- [ ] Binary versiyonu doğrulandı (v21)
- [ ] Upgrade dizini oluşturuldu
- [ ] Priv validator key yedeklendi
- [ ] Monitoring script kuruldu (opsiyonel)
- [ ] Manuel upgrade komutları hazır

---

## 🎯 Son Kontrol

```bash
echo "=== Version Check ==="
bitbadgeschaind version

echo "=== v21 Binary Ready ==="
$HOME/.bitbadgeschain/cosmovisor/upgrades/v21/bin/bitbadgeschaind version

echo "=== Blocks Until Upgrade ==="
CURRENT=$(bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height')
echo "Current: $CURRENT | Target: 7510000 | Remaining: $((7510000 - CURRENT))"
```

---

## 🚀 Son Hatırlatmalar

1. ✅ **Hazırlığı ŞİMDİ yapın**
2. ✅ **Logları izleyin**
3. ✅ **Manuel müdahaleye hazır olun**
4. ✅ **Sakin kalın**
5. ✅ **Hızlı hareket edin**

---
