# Contributor: Andreas B. Wagner <AndreasBWagner@pointfree.net>
pkgname=jsawk-git
pkgver=1.4.13.g5a14c4a
pkgrel=1
pkgdesc="like awk, but for JSON"
arch=('i686' 'x86_64')
url="http://github.com/micha/jsawk"
source=('jsawk-git::git+https://github.com/micha/jsawk.git#branch=master')
md5sums=('SKIP')
license=('unknown')
depends=('js')
makedepends=('git')
provides=('jsawk')
conflicts=('jsawk')

pkgver() {
  cd "${srcdir}/${pkgname}"
  git describe | sed 's/^v//;s/-/./g'
}

package() {
  cd "$SRCDEST"
  install -dm755 "${pkgdir}"/usr/bin/
  install -Dm755 jsawk "${pkgdir}"/usr/bin/jsawk

  install -dm755 "${pkgdir}"/etc
  echo "export JS=/usr/bin/js24" >> "${pkgdir}"/etc/jsawkrc
}
