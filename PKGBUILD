pkgname=nushell
pkgver=0.106.0
pkgrel=1
pkgdesc='A new type of shell operating on structured data'
arch=('aarch64')
url='https://www.nushell.sh'
license=('MIT')
depends=(openssl zlib)
source=("$pkgname-$pkgver.tar.gz::https://github.com/nushell/nushell/archive/$pkgver.tar.gz")
sha256sums=('3b0f26b293e76b4d6a9f593184ceca18e9853c6c5e81fcc958405040f0792bcc')

TERMUX_PREFIX='/data/data/com.termux/files/usr'

prepare() {
  cd "$pkgname-$pkgver"

  doas find '/opt/android-ndk/' -name 'libunwind.a' -execdir sh -c 'echo "INPUT(-lunwind)" > libgcc.a' \;

  yes | doas pacman --config "${HOME}/pacman_aarch64.conf" --arch aarch64 --sync openssl zlib
  doas chown --recursive build "${TERMUX_PREFIX}"

  rustup target add aarch64-linux-android
  cargo fetch --target aarch64-linux-android --locked
}

build() {
  cd "$pkgname-$pkgver"

  export AR_aarch64_linux_android='llvm-ar'

  export CFLAGS=''
  CFLAGS+=' -fstack-protector-strong'
  CFLAGS+=' -Oz'
  CFLAGS+=" -I${TERMUX_PREFIX}/include"

  export RUSTFLAGS="-C link-args=-Wl,-rpath=${TERMUX_PREFIX}/lib"

  export OPENSSL_DIR="${TERMUX_PREFIX}"

  cargo build \
    --target aarch64-linux-android \
    --locked \
    --release \
    --workspace
}

package() {
  cd "$pkgname-$pkgver"
  install -Dm700 -t "${pkgdir}${TERMUX_PREFIX}/bin" target/aarch64-linux-android/release/nu
  install -Dm600 -t "${pkgdir}${TERMUX_PREFIX}/share/doc/${pkgname}" LICENSE
}
