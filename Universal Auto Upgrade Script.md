```
#!/bin/bash

#############################################
# BitBadges Universal Auto Upgrade Script
# Tüm gelecek upgrade'ler için çalışır
#############################################

# Konfigürasyon
DAEMON_HOME="$HOME/.bitbadgeschain"
COSMOVISOR_DIR="$DAEMON_HOME/cosmovisor"
UPGRADE_INFO_FILE="$DAEMON_HOME/data/upgrade-info.json"
RPC_ENDPOINT="http://localhost:26657"
CHECK_INTERVAL=30  # Normal kontrol aralığı (saniye)
FAST_CHECK_INTERVAL=5  # Yaklaşınca kontrol aralığı (saniye)
FAST_CHECK_THRESHOLD=10  # Kaç blok kala hızlı kontrole geçilecek

# Renkli output için
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Log fonksiyonu
log() {
    echo -e "${BLUE}[$(date '+%Y-%m-%d %H:%M:%S')]${NC} $1"
}

log_success() {
    echo -e "${GREEN}[$(date '+%Y-%m-%d %H:%M:%S')] ✓${NC} $1"
}

log_warning() {
    echo -e "${YELLOW}[$(date '+%Y-%m-%d %H:%M:%S')] ⚠${NC} $1"
}

log_error() {
    echo -e "${RED}[$(date '+%Y-%m-%d %H:%M:%S')] ✗${NC} $1"
}

# Mevcut blok yüksekliğini al
get_current_height() {
    local height=$(curl -s $RPC_ENDPOINT/status 2>/dev/null | jq -r '.result.sync_info.latest_block_height' 2>/dev/null)
    if [[ -z "$height" || "$height" == "null" ]]; then
        echo "0"
    else
        echo "$height"
    fi
}

# Upgrade info dosyasını oku
read_upgrade_info() {
    if [ ! -f "$UPGRADE_INFO_FILE" ]; then
        echo "NOT_FOUND"
        return
    fi
    
    local upgrade_name=$(jq -r '.name' "$UPGRADE_INFO_FILE" 2>/dev/null)
    local upgrade_height=$(jq -r '.height' "$UPGRADE_INFO_FILE" 2>/dev/null)
    
    if [[ -z "$upgrade_name" || "$upgrade_name" == "null" || -z "$upgrade_height" || "$upgrade_height" == "null" ]]; then
        echo "INVALID"
        return
    fi
    
    echo "${upgrade_name}:${upgrade_height}"
}

# Upgrade binary'sinin hazır olup olmadığını kontrol et
check_upgrade_binary() {
    local upgrade_name=$1
    local binary_path="$COSMOVISOR_DIR/upgrades/$upgrade_name/bin/bitbadgeschaind"
    
    if [ ! -f "$binary_path" ]; then
        return 1
    fi
    
    if [ ! -x "$binary_path" ]; then
        return 2
    fi
    
    return 0
}

# Upgrade işlemini gerçekleştir
perform_upgrade() {
    local upgrade_name=$1
    
    echo ""
    echo "=================================================="
    log "🚀 UPGRADE İŞLEMİ BAŞLIYOR: $upgrade_name"
    echo "=================================================="
    echo ""
    
    # 1. Node'u durdur
    log "[1/5] Node durduruluyor..."
    sudo systemctl stop bitbadgeschaind
    
    # Node'un tamamen durmasını bekle
    sleep 5
    
    if sudo systemctl is-active --quiet bitbadgeschaind; then
        log_error "Node durdurulamadı!"
        return 1
    fi
    log_success "Node durduruldu"
    
    # 2. Eski current symlink'i yedekle ve kaldır
    log "[2/5] Eski symlink temizleniyor..."
    if [ -L "$COSMOVISOR_DIR/current" ]; then
        local old_target=$(readlink "$COSMOVISOR_DIR/current")
        log "Eski target: $old_target"
        rm -rf "$COSMOVISOR_DIR/current"
        log_success "Eski symlink kaldırıldı"
    else
        log_warning "Current symlink bulunamadı (ilk upgrade olabilir)"
    fi
    
    # 3. Yeni symlink oluştur
    log "[3/5] Yeni symlink oluşturuluyor: $upgrade_name"
    ln -s "$COSMOVISOR_DIR/upgrades/$upgrade_name" "$COSMOVISOR_DIR/current"
    
    if [ ! -L "$COSMOVISOR_DIR/current" ]; then
        log_error "Symlink oluşturulamadı!"
        return 1
    fi
    
    local new_target=$(readlink "$COSMOVISOR_DIR/current")
    log_success "Yeni symlink oluşturuldu: $new_target"
    
    # 4. Binary'yi kontrol et ve versiyonu göster
    log "[4/5] Binary kontrol ediliyor..."
    local binary_path="$COSMOVISOR_DIR/current/bin/bitbadgeschaind"
    
    if [ ! -x "$binary_path" ]; then
        log_error "Binary bulunamadı veya executable değil: $binary_path"
        return 1
    fi
    
    local version=$($binary_path version 2>/dev/null)
    log_success "Binary hazır - Version: $version"
    
    # 5. Node'u başlat
    log "[5/5] Node başlatılıyor..."
    sudo systemctl start bitbadgeschaind
    
    # Node'un başlamasını bekle
    sleep 10
    
    # Status kontrolü
    local retry=0
    local max_retry=6
    while [ $retry -lt $max_retry ]; do
        if sudo systemctl is-active --quiet bitbadgeschaind; then
            log_success "Node başarıyla başlatıldı"
            break
        fi
        retry=$((retry + 1))
        if [ $retry -lt $max_retry ]; then
            log_warning "Node henüz aktif değil, bekleniyor... ($retry/$max_retry)"
            sleep 5
        fi
    done
    
    if ! sudo systemctl is-active --quiet bitbadgeschaind; then
        log_error "Node başlatılamadı! Manuel kontrol gerekli."
        log "Kontrol için: sudo systemctl status bitbadgeschaind"
        log "Log için: journalctl -u bitbadgeschaind -n 100 --no-pager"
        return 1
    fi
    
    echo ""
    echo "=================================================="
    log_success "✅ UPGRADE TAMAMLANDI!"
    echo "=================================================="
    log "Upgrade: $upgrade_name"
    log "Version: $version"
    log "Target: $new_target"
    echo "=================================================="
    log "Log takip: journalctl -u bitbadgeschaind -f"
    echo "=================================================="
    echo ""
    
    return 0
}

# Ana script başlangıcı
clear
echo "=================================================="
echo "  BitBadges Universal Auto Upgrade Monitor"
echo "=================================================="
echo ""

# Ön kontroller
log "Sistem kontrolleri yapılıyor..."

# jq kontrolü
if ! command -v jq &> /dev/null; then
    log_error "jq yüklü değil! Lütfen yükleyin: sudo apt install jq -y"
    exit 1
fi

# Node durumu kontrolü
if ! sudo systemctl is-active --quiet bitbadgeschaind; then
    log_error "Node şu anda çalışmıyor!"
    exit 1
fi
log_success "Node aktif ve çalışıyor"

# RPC endpoint kontrolü
CURRENT_HEIGHT=$(get_current_height)
if [ "$CURRENT_HEIGHT" == "0" ]; then
    log_error "RPC endpoint'e bağlanılamadı: $RPC_ENDPOINT"
    log_warning "Node senkronize oluyor olabilir veya RPC kapalı olabilir"
    exit 1
fi
log_success "RPC bağlantısı OK - Mevcut yükseklik: $CURRENT_HEIGHT"

echo ""
log "✅ Tüm ön kontroller başarılı"
echo ""
echo "=================================================="
log "⏳ Upgrade bilgisi bekleniyor..."
echo "=================================================="
echo ""

# Ana döngü
LAST_UPGRADE_INFO=""
MONITORING_UPGRADE=false
UPGRADE_NAME=""
UPGRADE_HEIGHT=0

while true; do
    # Upgrade info dosyasını oku
    UPGRADE_INFO=$(read_upgrade_info)
    
    # Upgrade info değişti mi kontrol et
    if [ "$UPGRADE_INFO" != "$LAST_UPGRADE_INFO" ]; then
        if [ "$UPGRADE_INFO" == "NOT_FOUND" ]; then
            if [ "$MONITORING_UPGRADE" == true ]; then
                log_warning "upgrade-info.json dosyası bulunamadı (upgrade tamamlandı olabilir)"
                MONITORING_UPGRADE=false
            fi
        elif [ "$UPGRADE_INFO" == "INVALID" ]; then
            log_error "upgrade-info.json dosyası geçersiz formatta"
        else
            # Yeni upgrade bilgisi geldi
            UPGRADE_NAME=$(echo $UPGRADE_INFO | cut -d':' -f1)
            UPGRADE_HEIGHT=$(echo $UPGRADE_INFO | cut -d':' -f2)
            
            echo ""
            echo "=================================================="
            log "🎯 YENİ UPGRADE TESPİT EDİLDİ!"
            echo "=================================================="
            log "Upgrade Adı: $UPGRADE_NAME"
            log "Hedef Blok: $UPGRADE_HEIGHT"
            log "Mevcut Blok: $CURRENT_HEIGHT"
            log "Kalan Blok: $(($UPGRADE_HEIGHT - $CURRENT_HEIGHT))"
            echo "=================================================="
            echo ""
            
            # Binary kontrolü
            log "Binary kontrol ediliyor: $UPGRADE_NAME"
            check_upgrade_binary "$UPGRADE_NAME"
            case $? in
                0)
                    log_success "✅ Binary hazır: $UPGRADE_NAME"
                    MONITORING_UPGRADE=true
                    ;;
                1)
                    log_error "Binary bulunamadı: $COSMOVISOR_DIR/upgrades/$UPGRADE_NAME/bin/bitbadgeschaind"
                    log "Lütfen binary'yi hazırlayın ve script tekrar kontrol edecek"
                    MONITORING_UPGRADE=false
                    ;;
                2)
                    log_error "Binary executable değil, chmod +x gerekli"
                    log "Düzeltme için: chmod +x $COSMOVISOR_DIR/upgrades/$UPGRADE_NAME/bin/bitbadgeschaind"
                    MONITORING_UPGRADE=false
                    ;;
            esac
            echo ""
        fi
        
        LAST_UPGRADE_INFO="$UPGRADE_INFO"
    fi
    
    # Eğer upgrade monitoring aktifse
    if [ "$MONITORING_UPGRADE" == true ]; then
        CURRENT_HEIGHT=$(get_current_height)
        
        if [ "$CURRENT_HEIGHT" == "0" ]; then
            log_warning "RPC'ye bağlanılamadı, tekrar deneniyor..."
            sleep $CHECK_INTERVAL
            continue
        fi
        
        BLOCKS_REMAINING=$(($UPGRADE_HEIGHT - $CURRENT_HEIGHT))
        
        # Upgrade yüksekliğine ulaşıldı mı?
        if [ $CURRENT_HEIGHT -ge $UPGRADE_HEIGHT ]; then
            echo ""
            echo "=================================================="
            log "🎯 UPGRADE YÜKSEKLIĞINE ULAŞILDI!"
            echo "=================================================="
            log "Upgrade: $UPGRADE_NAME"
            log "Mevcut yükseklik: $CURRENT_HEIGHT"
            log "Upgrade yüksekliği: $UPGRADE_HEIGHT"
            echo "=================================================="
            echo ""
            
            # Binary son kontrol
            check_upgrade_binary "$UPGRADE_NAME"
            if [ $? -ne 0 ]; then
                log_error "HATA: Binary hazır değil! Upgrade yapılamıyor."
                log "Binary: $COSMOVISOR_DIR/upgrades/$UPGRADE_NAME/bin/bitbadgeschaind"
                exit 1
            fi
            
            # Upgrade işlemini gerçekleştir
            perform_upgrade "$UPGRADE_NAME"
            
            if [ $? -eq 0 ]; then
                log_success "Upgrade başarıyla tamamlandı!"
                log "Bir sonraki upgrade için bekleniyor..."
                MONITORING_UPGRADE=false
                LAST_UPGRADE_INFO=""
            else
                log_error "Upgrade başarısız! Manuel müdahale gerekli."
                exit 1
            fi
            
            echo ""
        else
            # İlerleme göster
            if [ $(($BLOCKS_REMAINING % 100)) -eq 0 ] || [ $BLOCKS_REMAINING -le $FAST_CHECK_THRESHOLD ]; then
                log "📊 [$UPGRADE_NAME] Mevcut: $CURRENT_HEIGHT | Hedef: $UPGRADE_HEIGHT | Kalan: $BLOCKS_REMAINING blok"
            fi
            
            # Yaklaştıkça daha sık kontrol et
            if [ $BLOCKS_REMAINING -le $FAST_CHECK_THRESHOLD ]; then
                sleep $FAST_CHECK_INTERVAL
            else
                sleep $CHECK_INTERVAL
            fi
        fi
    else
        # Henüz upgrade bilgisi yok veya binary hazır değil, daha seyrek kontrol et
        sleep $CHECK_INTERVAL
    fi
done
```


```
# Script'i oluştur
nano ~/bitbadges_universal_upgrade.sh

# Yukarıdaki kodu yapıştır ve kaydet

# Executable yap
chmod +x ~/bitbadges_universal_upgrade.sh
```


```
