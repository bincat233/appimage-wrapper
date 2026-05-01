# Maintainer: bincat233
pkgname=appimage-wrapper
pkgver=0.0.1
pkgrel=1
pkgdesc='Wrapper and desktop entry generator for versioned AppImage launches'
arch=('any')
url='https://github.com/bincat233/appimage-wrapper'
license=('WTFPL')
depends=('bash' 'coreutils' 'findutils')
optdepends=(
  'bash-completion: Bash completions'
  'zsh: Zsh completions'
  'imagemagick: detect icon dimensions when installing PNG icons'
  'desktop-file-utils: validate generated desktop files'
)
source=("${pkgname}-${pkgver}.tar.gz::${url}/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=('SKIP')

package() {
  cd "${srcdir}/${pkgname}-${pkgver}"

  install -Dm755 bin/appimage-wrapper "${pkgdir}/usr/bin/appimage-wrapper"
  install -Dm755 bin/appimage-desktop "${pkgdir}/usr/bin/appimage-desktop"

  install -Dm644 bash/appimage-wrapper "${pkgdir}/usr/share/bash-completion/completions/appimage-wrapper"
  install -Dm644 bash/appimage-desktop "${pkgdir}/usr/share/bash-completion/completions/appimage-desktop"
  install -Dm644 zsh/_appimage-wrapper "${pkgdir}/usr/share/zsh/site-functions/_appimage-wrapper"
  install -Dm644 zsh/_appimage-desktop "${pkgdir}/usr/share/zsh/site-functions/_appimage-desktop"

  install -Dm644 README.md "${pkgdir}/usr/share/doc/${pkgname}/README.md"
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
