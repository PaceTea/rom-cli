# Maintainer: PaceTea <dimip.2006@gmail.com>
pkgname=rom-cli
pkgver=1.1.0
pkgrel=1
pkgdesc="Browse and download legal ROMs, homebrew and public-domain games from the Internet Archive (ani-cli style)"
arch=('any')
url="https://github.com/PaceTea/rom-cli"
license=('GPL3')
depends=('curl' 'jq' 'fzf' 'aria2')
source=("$pkgname::git+https://github.com/PaceTea/rom-cli.git")
sha256sums=('SKIP')

package() {
    cd "$srcdir/$pkgname"
    install -Dm755 rom-cli "$pkgdir/usr/bin/rom-cli"
    install -Dm644 README.md "$pkgdir/usr/share/doc/$pkgname/README.md"
    install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
