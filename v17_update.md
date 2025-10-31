# BitBadges v16 → v17 Güncelleme Rehberi

## 📋 Güncelleme Bilgileri

- **Upgrade Adı:** v17
- **Upgrade Blok Yüksekliği:** #6624000
- **Tahmini Zaman:** 31 Ekim 2025, 15:22:42
- **Mevcut Versiyon:** v16
- **Hedef Versiyon:** v17

## 🔍 Güncelleme İçeriği

v17 güncellemesi aşağıdaki iyileştirmeleri içermektedir:

- v16 upgrade'den sonra invariant'larla ilgili minor bug düzeltmeleri
- Transaction serialization'da x/group mesajları için destek eklenmesi

## ⚠️ Önemli Notlar

- **Cosmovisor kullanan node'lar** için bu güncelleme **OTOMATIK** olarak gerçekleşecektir
- Binary'nin doğru konuma yerleştirilmesi **zorunludur**
- Doğru kurulum yapılmazsa node #6624000 bloğunda **durur** ve **slash** riski oluşur
- Güncelleme öncesi mutlaka binary'nin hazır olduğundan emin olun

---

## 🚀 Güncelleme Adımları

### Adım 1: Mevcut Durumu Kontrol Edin

#### Mevcut Blok Yüksekliğini Görüntüleme

```bash
bitbadgeschaind status 2>&1 | jq -r '.SyncInfo.latest_block_height'
```

#### Mevcut Versiyonu Kontrol Etme

```bash
bitbadgeschaind version
```

Çıktı: `v16` olmalıdır.

#### Node Senkronizasyon Durumunu Kontrol Etme

```bash
bitbadgeschaind status 2>&1 | jq .SyncInfo.catching_up
```

Çıktı: `false` olmalıdır (node senkronize).

---

### Adım 2: v17 Binary İndirme ve Derleme

#### Kaynak Kodunu İndirme

```bash
cd $HOME
rm -rf bitbadgeschain
git clone https://github.com/BitBadges/bitbadgeschain.git
cd bitbadgeschain
git checkout v17
```

#### Binary'yi Derleme

```bash
make build-linux/amd64
```

#### Derleme Başarısını Kontrol Etme

```bash
ls -la build/bitbadgeschain-linux-amd64
```

Dosya görünüyor olmalıdır.

---

### Adım 3: Cosmovisor İçin Upgrade Dizini Hazırlama

#### Upgrade Dizinini Oluşturma

```bash
mkdir -p $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin
```

#### Dizin Yapısını Doğrulama

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/
```

`v17` klasörü görünmelidir.

---

### Adım 4: Binary'yi Doğru Konuma Yerleştirme

#### Binary'yi Kopyalama

```bash
cp $HOME/bitbadgeschain/build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind
```

#### Çalıştırılabilir İzinleri Verme

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind
```

---

### Adım 5: Binary Versiyonunu Doğrulama

#### v17 Binary Versiyonunu Kontrol Etme

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v17`

#### Binary'nin Çalıştırılabilir Olduğunu Doğrulama

```bash
$HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind version --long
```

Detaylı versiyon bilgisi görüntülenmelidir.

---

### Adım 6: Cosmovisor Yapılandırmasını Kontrol Etme

#### Cosmovisor Servis Dosyasını Kontrol Etme

```bash
cat /etc/systemd/system/bitbadgeschaind.service
```

Aşağıdaki ortam değişkenlerinin olduğundan emin olun:

```
Environment="DAEMON_NAME=bitbadgeschaind"
Environment="DAEMON_HOME=$HOME/.bitbadgeschain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
```

#### Upgrade Dizin Yapısını Son Kontrol

```bash
tree -L 3 $HOME/.bitbadgeschain/cosmovisor/
```

veya

```bash
find $HOME/.bitbadgeschain/cosmovisor/ -type f -name "bitbadgeschaind"
```

Çıktıda şunları görmelisiniz:
- `$HOME/.bitbadgeschain/cosmovisor/genesis/bin/bitbadgeschaind` (v16)
- `$HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind` (v17)

---

### Adım 7: Manuel Upgrade Link Oluşturma (Cosmovisor Otomatik Geçiş Yapmazsa)

Bazı durumlarda Cosmovisor otomatik geçiş yapamayabilir. Bu durumda manuel olarak current link'i oluşturmanız gerekir:

#### Servisi Durdurun

```bash
sudo systemctl stop bitbadgeschaind
```

#### Current Link'i Manuel Oluşturun

```bash
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v17 $HOME/.bitbadgeschain/cosmovisor/current
```

#### Link'i Doğrulayın

```bash
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind
```

#### Versiyonu Kontrol Edin

```bash
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version
```

**Beklenen Çıktı:** `v17`

#### Servisi Başlatın

```bash
sudo systemctl start bitbadgeschaind
```

#### Logları İzleyin

```bash
journalctl -u bitbadgeschaind -f
```

---

## 📊 Güncelleme Öncesi Kontrol Listesi

Aşağıdaki tüm maddeleri kontrol edin:

- [ ] Mevcut versiyon v16
- [ ] Node tamamen senkronize (`catching_up: false`)
- [ ] v17 binary indirildi ve derlendi
- [ ] v17 binary `$HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/` dizinine kopyalandı
- [ ] Binary adı `bitbadgeschaind` olarak ayarlandı
- [ ] Binary çalıştırılabilir izinlere sahip (`chmod +x`)
- [ ] Binary versiyon kontrolü yapıldı (`v17` çıktısı alındı)
- [ ] Cosmovisor servis dosyası doğru yapılandırılmış
- [ ] **ÖNEMLİ:** Blok #6624000'e ulaşıldığında Adım 7'deki manuel link oluşturma işlemini yapın

---

## 🔄 Güncelleme Sırasında İzleme

### Logları Canlı Takip Etme

Blok yüksekliği #6624000'e yaklaştığında logları izleyin:

```bash
journalctl -u bitbadgeschaind -f
```

### Belirli Kelimeleri Filtreleyerek İzleme

```bash
journalctl -u bitbadgeschaind -f | grep -i "upgrade\|halt\|panic\|error"
```

### Blok Yüksekliğini Sürekli İzleme

Başka bir terminal penceresinde:

```bash
watch -n 5 'bitbadgeschaind status 2>&1 | jq -r ".SyncInfo.latest_block_height"'
```

---

## ✅ Güncelleme Sonrası Doğrulama

### Adım 1: Servis Durumunu Kontrol Etme

```bash
sudo systemctl status bitbadgeschaind
```

Servis `active (running)` durumunda olmalıdır.

### Adım 2: Versiyon Kontrolü

```bash
bitbadgeschaind version
```

**Beklenen Çıktı:** `v17`

### Adım 3: Node Senkronizasyon Durumu

```bash
bitbadgeschaind status 2>&1 | jq .SyncInfo
```

Node'un blokları üretmeye/takip etmeye devam ettiğini doğrulayın.

### Adım 4: Validator Durumunu Kontrol Etme (Validator İseniz)

```bash
bitbadgeschaind query staking validator $(bitbadgeschaind keys show cüzdan-adı --bech val -a)
```

Validator'ınızın jailed olmadığından emin olun.

### Adım 5: Log Kontrolü

```bash
journalctl -u bitbadgeschaind -n 100 --no-pager
```

Son 100 log satırında hata olmadığını kontrol edin.

---

## 🐛 Sorun Giderme

### Problem: Node Blok #6624000'de Durdu

**Neden:** Cosmovisor otomatik geçiş yapamadı ve current link'i güncellemedi.

**Çözüm:**

```bash
# Servisi durdurun
sudo systemctl stop bitbadgeschaind

# Current link'i manuel oluşturun
rm -rf $HOME/.bitbadgeschain/cosmovisor/current
ln -s $HOME/.bitbadgeschain/cosmovisor/upgrades/v17 $HOME/.bitbadgeschain/cosmovisor/current

# Link'i doğrulayın
ls -la $HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind

# Versiyonu kontrol edin
$HOME/.bitbadgeschain/cosmovisor/current/bin/bitbadgeschaind version

# Servisi başlatın
sudo systemctl start bitbadgeschaind

# Logları izleyin
journalctl -u bitbadgeschaind -f
```

### Problem: "permission denied" Hatası

**Çözüm:**

```bash
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind
sudo systemctl restart bitbadgeschaind
```

### Problem: "binary not found" Hatası

**Çözüm:**

```bash
# Binary'nin varlığını kontrol edin
ls -la $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/

# Eğer yoksa tekrar kopyalayın
cp $HOME/bitbadgeschain/build/bitbadgeschain-linux-amd64 $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind
chmod +x $HOME/.bitbadgeschain/cosmovisor/upgrades/v17/bin/bitbadgeschaind
```

### Problem: Node Panic Veriyor

**Çözüm:**

```bash
# Logları kontrol edin
journalctl -u bitbadgeschaind -n 200 --no-pager

# Gerekirse snapshot ile sıfırlama
bitbadgeschaind tendermint unsafe-reset-all --home $HOME/.bitbadgeschain --keep-addr-book

# Snapshot indirin (opsiyonel)
SNAP_NAME=$(curl -s https://ss.bitbadges.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.bitbadges.nodestake.org/${SNAP_NAME} | lz4 -c -d - | tar -x -C $HOME/.bitbadgeschain

# Node'u yeniden başlatın
sudo systemctl restart bitbadgeschaind
```

### Problem: Validator Jailed Oldu

**Çözüm:**

```bash
# Unjail komutu
bitbadgeschaind tx slashing unjail \
  --chain-id bitbadges-1 \
  --from cüzdan-adı \
  --gas auto \
  --gas-adjustment 1.5 \
  --fees 5000ubadge \
  -y
```

---

## 📝 Notlar

- **Cosmovisor kullanmıyorsanız** manual olarak güncelleme yapmanız gerekir
- Güncelleme sırasında node'un durmasını önlemek için önceden hazırlık yapın
- Validator operatörler için: Slashing riskini azaltmak için monitoring sistemlerinizi kontrol edin
- Testnet'te önce test etmek isterseniz, testnet endpoint'lerini kullanabilirsiniz

---

## ✨ Güncelleme Başarılı!

v17 güncellemesi tamamlandığında:

```bash
bitbadgeschaind version
# Çıktı: v17
```

Node'unuz artık v17 ile çalışıyor demektir. 🎉

---

**Son Güncelleme:** 30 Ekim 2025
