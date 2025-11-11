# BitBadges v17 → v18 Güncelleme Rehberi

## 📋 Güncelleme Bilgileri

- **Upgrade Adı:** v18  
- **Upgrade Blok Yüksekliği:** #6930000  
- **Tahmini Zaman:** [Explorer Linki](https://explorer.bitbadges.io/BitBadges%20Mainnet/gov/25)  
- **Voting Period:** Şimdi + 24 saat  
- **Mevcut Versiyon:** v17  
- **Hedef Versiyon:** v18  

## 🔍 Güncelleme İçeriği

v18 güncellemesi aşağıdaki değişiklikleri içermektedir:

- **Arayüz Güncellemesi:** “badges” terminolojisi yerine artık **“tokens”** ifadesi kullanılacaktır. Bu, daha global bir dil birliği sağlamak için yapılmıştır.  
- **Likidite Havuzu Mantığı Güncellemesi:** Likidite havuzundan yapılan transferlerde artık işlemi teknik olarak **pool adresi** yerine **işlemi başlatan kullanıcı (tx creator)** gerçekleştirmektedir.  

📦 **Binary ve Talimatlar:**  
🔗 [https://github.com/BitBadges/bitbadgeschain/releases/tag/v18](https://github.com/BitBadges/bitbadgeschain/releases/tag/v18)

⚠️ **Not:**  
GitHub üzerinde **v19** testnet için denenmektedir.  
Mainnet upgrade için **mutlaka v18 binary’sini** kullanın!

---

## 🚀 Güncelleme Adımları

### Adım 1: Mevcut Durumu Kontrol Edin

#### Blok Yüksekliği Kontrolü

```bash
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'
```

#### Mevcut Versiyon

```bash
bitbadgeschaind version
```

**Beklenen çıktı:** `v17`

#### Node Senkronizasyonu

```bash
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
```

**Beklenen çıktı:** `false`

---

### Adım 2: v18 Binary İndirme ve Derleme

#### Kaynak Kodunu İndirme

```bash
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v18
```

#### Derleme

```bash
make build-linux/amd64
```

#### Derleme Kontrolü

```bash
ls -la build/bitbadgeschain-linux-amd64
```

---

### Adım 3: Cosmovisor Dizinini Hazırlama

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin
```

---

### Adım 4: Binary’yi Yerleştirme

```bash
cp $HOME/bitbadgeschain/build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind
```

---

### Adım 5: Binary Versiyonunu Doğrulama

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v18`

---

### Adım 6: Cosmovisor Servis Kontrolü

```bash
cat /etc/systemd/system/bitbadgeschaind.service
```

Aşağıdaki satırların mevcut olduğundan emin olun:

```
Environment="DAEMON_NAME=bitbadgeschaind"
Environment="DAEMON_HOME=$HOME/.bitbadgeschain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
```

---

### Adım 7: Manuel Link (Gerekirse)

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v18 $HOME/.bitbadgeschain/cosmovisor/current
sudo systemctl start bitbadgeschaind
```

---

## 📊 Güncelleme Öncesi Kontrol Listesi

- [ ] Mevcut versiyon `v17`
- [ ] Node senkronize (`catching_up: false`)
- [ ] `v18` binary indirildi ve derlendi
- [ ] Binary doğru dizine kopyalandı
- [ ] Binary çalıştırılabilir (`chmod +x`)
- [ ] Cosmovisor doğru yapılandırıldı
- [ ] `bitbadgeschaind version` çıktısı `v18`

---

## 🔄 Güncelleme Sırasında İzleme

```bash
journalctl -u bitbadgeschaind -f
```

Filtreli izleme:

```bash
journalctl -u bitbadgeschaind -f | grep -i "upgrade\|halt\|panic\|error"
```

Blok takibi:

```bash
watch -n 5 'bitbadgeschaind status 2>&1 | jq -r ".SyncInfo.latest_block_height"'
```

---

## ✅ Güncelleme Sonrası Doğrulama

### Servis Durumu

```bash
sudo systemctl status bitbadgeschaind
```

### Versiyon Kontrolü

```bash
bitbadgeschaind version
```

**Beklenen Çıktı:** `v18`

### Node Senkronizasyonu

```bash
bitbadgeschaind status 2>&1 | jq .SyncInfo
```

---

## 🐛 Sorun Giderme

### Node Upgrade Sonrası Durduysa

```bash
sudo systemctl stop bitbadgeschaind
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v18 $HOME/.bitbadgeschain/cosmovisor/current
sudo systemctl start bitbadgeschaind
journalctl -u bitbadgeschaind -f
```

### Permission Denied

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind
sudo systemctl restart bitbadgeschaind
```

### Binary Not Found

```bash
cp $HOME/bitbadgeschain/build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v18/bin/bitbadgeschaind
```

---

## 📝 Notlar

- **Cosmovisor** kullananlar için upgrade otomatik gerçekleşecektir.  
- **v19** şu anda **testnet** denemeleri için kullanılmaktadır.  
  → Mainnet için **yalnızca v18** binary’sini kullanın.  
- Güncelleme sırasında validator’lar **slash** riskine karşı node’larını izlemelidir.

---

## ✨ Güncelleme Başarılı!

```bash
bitbadgeschaind version
# Çıktı: v18
```

Node’unuz artık **v18** sürümünde çalışıyor! 🎉

---

**Son Güncelleme:** 11 Kasım 2025
