# Based on the file created for Arch Linux by:
# Dan McGee <dan@archlinux.org>
# Dave Reisner <dreisner@archlinux.org>

# Maintainer: Philip Müller <philm@manjaro.org>
# Maintainer: Guinux <guillaume@manjaro.org>

# Note: since we include pacman-contrib with this package we have to build pacman twice
# Example: pacsort: error while loading shared libraries

pkgname=pacman
pkgver=5.1.1
_pkgver=1.1.0
pkgrel=2
pkgdesc="A library-based package manager with dependency support"
arch=('i686' 'x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base' 'base-devel')
depends=('bash>=4.2.042-2' 'glibc>=2.17-2' 'libarchive>=3.1.2' 'curl>=7.39.0'
         'gpgme' 'archlinux-keyring' 'manjaro-keyring' 'pacman-mirrors>=4.1.0')
checkdepends=('python2' 'fakechroot')
makedepends=('asciidoc' 'pacman>=5.1')
optdepends=('haveged: for pacman-init.service'
            'perl-locale-gettext: translation support in makepkg-template')
provides=('pacman-contrib' 'pacman-init')
conflicts=('pacman-contrib' 'pacman-init')
replaces=('pacman-contrib' 'pacman-init')
backup=(etc/pacman.conf etc/makepkg.conf)
install=pacman.install
options=('strip' 'debug')
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121'  # Andrew Gregory (pacman) <andrew@archlinux.org>
              '5134EF9EAF65F95B6BB1608E50FB9B273A9D0BB5') # Johannes Löthberg <johannes@kyriasis.com>
source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.gz{,.sig}
        https://sources.archlinux.org/other/community/pacman-contrib/pacman-contrib-$_pkgver.tar.gz{,.asc}
        pacman.conf.i686
        pacman.conf.x86_64
        makepkg.conf
        pacman-sync-first-option.patch
        etc-pacman.d-gnupg.mount
        pacman-init.service)

prepare() {
  cd $srcdir/$pkgname-$pkgver

  # Manjaro patches
  patch -p1 -i $srcdir/pacman-sync-first-option.patch

  ./configure --prefix=/usr --sysconfdir=/etc \
    --localstatedir=/var --enable-doc \
    --with-scriptlet-shell=/usr/bin/bash \
    --with-ldconfig=/usr/bin/ldconfig
}

build() {
  cd $srcdir/$pkgname-$pkgver
  make V=1

  cd $srcdir/pacman-contrib-$_pkgver

  ./configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --localstatedir=/var
  make
}

check() {
  make -C "$pkgname-$pkgver" check
  make -C pacman-contrib-$_pkgver check
}

package() {
  cd $srcdir/$pkgname-$pkgver
  make DESTDIR=$pkgdir install

  # install Arch specific stuff
  install -dm755 $pkgdir/etc
  install -m644 $srcdir/pacman.conf.$CARCH $pkgdir/etc/pacman.conf
  
  case "$CARCH" in
    i686)    
      mycarch="i686"
      mychost="i686-pc-linux-gnu"
      myflags="-march=i686"
      ;;
    x86_64)
      mycarch="x86_64"
      mychost="x86_64-pc-linux-gnu"
      myflags="-march=x86-64"
      ;;
  esac
  install -m644 $srcdir/makepkg.conf $pkgdir/etc/
  # set things correctly in the default conf file
  sed -i $pkgdir/etc/makepkg.conf \
    -e "s|@CARCH[@]|$mycarch|g" \
    -e "s|@CHOST[@]|$mychost|g" \
    -e "s|@CARCHFLAGS[@]|$myflags|g"
    
  # put bash_completion in the right location
  install -dm755 ${pkgdir}/usr/share/bash-completion/completions
  mv ${pkgdir}/etc/bash_completion.d/pacman \
    ${pkgdir}/usr/share/bash-completion/completions
  rmdir ${pkgdir}/etc/bash_completion.d

  for f in makepkg pacman-key; do
    ln -s pacman "$pkgdir/usr/share/bash-completion/completions/$f"
  done

  # install pacman-init
  install -dm755 $pkgdir/usr/lib/systemd/system/
  install -m644 $srcdir/etc-pacman.d-gnupg.mount $pkgdir/usr/lib/systemd/system/etc-pacman.d-gnupg.mount
  install -m644 $srcdir/pacman-init.service $pkgdir/usr/lib/systemd/system/pacman-init.service

  cd $srcdir/pacman-contrib-$_pkgver

  make DESTDIR="$pkgdir" install

  # replace rankmirrors
  rm "$pkgdir/usr/bin/rankmirrors"
  ln -sfv "/usr/bin/pacman-mirrors" "$pkgdir/usr/bin/rankmirrors"
}

sha256sums=('be04b9162d62d2567e21402dcbabb5bedfdb03909fa5ec6e8568e02ab325bd8d'
            'SKIP'
            '308c3b8dc15ed8bd419cba1eb3103afe9250cf415626334a0c3a753b550549a6'
            'SKIP'
            '4421dc5d63a24e926852c1ea83b575355772aaa2add71cc522cd04ca22b131d6'
            'a9f21ebe59f1a2c8309145930ee96b8e3dc6e96e150fb77269d9958a80550950'
            'eec71e0b23f2ac1f06ad6ec5890d9de62effe0883ffd16573eed2153d0c87475'
            'fe0901a02d34ccf288743397fa32526f6dd878db8337e666082bced10e24e754'
            'b6d14727ec465bb66d0a0358163b1bbfafcb4eaed55a0f57c30aabafae7eed68'
            '65d8bdccdcccb64ae05160b5d1e7f3e45e1887baf89dda36c1bd44c62442f91b')
