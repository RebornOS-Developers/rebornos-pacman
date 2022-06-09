# Maintainer: Philip Müller <philm[at]manjaro[dot]org>
# Maintainer: Bernhard Landauer <bernhard[at]manjaro[dot]org>
# Maintainer: Mark Wagie <mark at manjaro dot org>
# Contributor: Helmut Stult

# Arch credits:
# Dan McGee <dan@archlinux.org>
# Dave Reisner <dreisner@archlinux.org>

pkgname=pacman
pkgver=6.0.1
_pkgver=1.5.3
pkgrel=9
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base-devel')
depends=('bash' 'fakeroot' 'glibc' 'libarchive' 'curl' 'perl' 'gpgme' 'archlinux-keyring'
         'manjaro-keyring' 'pacman-mirrors>=4.1.0')
checkdepends=('python' 'fakechroot')
makedepends=('asciidoc' 'pacman>=6.0.0' 'meson' 'doxygen')
optdepends=('haveged: for pacman-init.service'
            'perl-locale-gettext: translation support in makepkg-template'
            'diffutils: for pacdiff'
            'findutils: for pacdiff --find'
            'mlocate: for pacdiff --locate'
            'sudo: privilege elevation for several scripts'
            'vim: default merge program for pacdiff')
provides=('pacman-contrib' 'pacman-init' 'libalpm.so')
conflicts=('pacman-contrib' 'pacman-init')
replaces=('pacman-contrib' 'pacman-init')
backup=(etc/pacman.conf etc/makepkg.conf)
#options=('strip' 'debug')
install=pacman.install
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121'  # Andrew Gregory (pacman) <andrew@archlinux.org>
              '5134EF9EAF65F95B6BB1608E50FB9B273A9D0BB5' # Johannes Löthberg <johannes@kyriasis.com>
              '04DC3FB1445FECA813C27EFAEA4F7B321A906AD9') # Daniel M. Capella <polyzen@archlinux.org>

source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.xz{,.sig}
        https://gitlab.archlinux.org/pacman/pacman-contrib/-/archive/v$_pkgver/pacman-contrib-v$_pkgver.tar.gz
        "fix-wkd-lookup.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/e1246baddd14ec6f4b6270b59bea0e1b639472a7.patch"
        'add-flto-to-LDFLAGS-for-clang.patch'
        makepkg-use-ffile-prefix-map-instead-of-fdebug-prefi.patch
        libmakepkg-add-extra-buildflags-only-when-buildflags.patch
        make-link-time-optimization-flags-configurable.patch
        pacman.conf
        makepkg.conf
        pacman-sync-first-option.patch
        etc-pacman.d-gnupg.mount
        pacman-init.service)
sha256sums=('0db61456e56aa49e260e891c0b025be210319e62b15521f29d3e93b00d3bf731'
            'SKIP'
            '56c0b1ae41ff8e72ae352739e80df0e9f0403a1c66b52d358c46e04f65a82057'
            '8ab5b1338874d7d58e11c5d1185ea3454fcc89755f9c18faf87ff348ad1ed16c'
            '82ff91b85f4c6ceba19f9330437e2a22aabc966c2b9e2a20a53857f98a42c223'
            'b940e6c0c05a185dce1dbb9da0dcbebf742fca7a63f3e3308d49205afe5a6582'
            '7d0aee976c9c71fcf7c96ef1d99aa76efe47d8c1f4451842d6d159ec7deb4278'
            '5b43e26a76be3ed10a69d4bfb2be48db8cce359baf46583411c7f124737ebe6a'
            'a71fabbf3cce40c8df7b2d9897d01c77bcc51c692258224acf492a4440f0feb7'
            '0fd86997eba2fce40425a20ef7515a7cc1f3ba9aedfeea7dfadeb06dec60dd57'
            '8167155d3a3e15fc4a1b1e989fdb826779e7b3690a52e2ca9d307ae0b1550e1d'
            'b6d14727ec465bb66d0a0358163b1bbfafcb4eaed55a0f57c30aabafae7eed68'
            '65d8bdccdcccb64ae05160b5d1e7f3e45e1887baf89dda36c1bd44c62442f91b')

prepare() {
  cd $pkgname-$pkgver
  patch -Np1 -i ../'add-flto-to-LDFLAGS-for-clang.patch'
  patch -Np1 -i ../makepkg-use-ffile-prefix-map-instead-of-fdebug-prefi.patch
  patch -Np1 -i ../libmakepkg-add-extra-buildflags-only-when-buildflags.patch
  patch -Np1 -i ../make-link-time-optimization-flags-configurable.patch
  patch -Np1 -i ../fix-wkd-lookup.patch

  # Manjaro patches
  patch -Np1 < "$srcdir"/pacman-sync-first-option.patch

  cd $srcdir/pacman-contrib-v$_pkgver
  ./autogen.sh
}

build() {
  cd "$pkgname-$pkgver"

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Ddoxygen=enabled \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        build

  meson compile -C build

  cd $srcdir/pacman-contrib-v$_pkgver

  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var
  make
}

check() {
  cd "$pkgname-$pkgver"
  meson test -C build

  make -C ../pacman-contrib-v$_pkgver check
}

package() {
  cd "$pkgname-$pkgver"

  DESTDIR="$pkgdir" meson install -C build

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"

  # install pacman-init
  install -dm755 $pkgdir/usr/lib/systemd/system/
  install -m644 $srcdir/etc-pacman.d-gnupg.mount $pkgdir/usr/lib/systemd/system/etc-pacman.d-gnupg.mount
  install -m644 $srcdir/pacman-init.service $pkgdir/usr/lib/systemd/system/pacman-init.service

  cd $srcdir/pacman-contrib-v$_pkgver

  make DESTDIR="$pkgdir" install

  # replace rankmirrors
  rm "$pkgdir/usr/bin/rankmirrors"
  ln -sfv "/usr/bin/pacman-mirrors" "$pkgdir/usr/bin/rankmirrors"
}
