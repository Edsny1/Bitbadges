```
#!/bin/bash

# BitBadges Binary Builder Helper

if [ -z "$1" ]; then
    echo "Kullanım: ./build_upgrade.sh v20"
    exit 1
fi

VERSION=$1
REPO="https://github.com/BitBadges/bitbadgeschain.git"
COSMOVISOR_DIR="$HOME/.bitbadgeschain/cosmovisor"

echo "=================================================="
echo "BitBadges Binary Builder"
echo "Version: $VERSION"
echo "=================================================="

cd $HOME
rm -rf bitbadgeschain
git clone $REPO
cd bitbadgeschain
git checkout $VERSION
make build-linux/amd64
mkdir -p $COSMOVISOR_DIR/upgrades/$VERSION/bin
cp build/bitbadgeschain-linux-amd64 $COSMOVISOR_DIR/upgrades/$VERSION/bin/bitbadgeschaind
chmod +x $COSMOVISOR_DIR/upgrades/$VERSION/bin/bitbadgeschaind

echo ""
echo "✅ Binary hazır!"
$COSMOVISOR_DIR/upgrades/$VERSION/bin/bitbadgeschaind version
echo ""
echo "Auto-upgrade script otomatik olarak tespit edecek."
```


```
# Kullanımı:
chmod +x ~/build_upgrade.sh
~/build_upgrade.sh v20
~/build_upgrade.sh v21
# vs...
```
