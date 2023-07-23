pkgname=nushell
pkgver=0.82.0
pkgrel=1
pkgdesc='A new type of shell operating on structured data'
arch=('aarch64')
url='https://www.nushell.sh'
license=('MIT')
depends=(openssl zlib)
source=("$pkgname-$pkgver.tar.gz::https://github.com/nushell/nushell/archive/$pkgver.tar.gz")
sha256sums=('587847feeb9fc06eb2a9da5ff05ffea5238fe5928ebea944c042838d8ad136e8')

TERMUX_PREFIX='/data/data/com.termux/files/usr'

prepare() {
  cd "$pkgname-$pkgver"

  doas find '/opt/android-ndk/' -name 'libunwind.a' -execdir sh -c 'echo "INPUT(-lunwind)" > libgcc.a' \;

  yes | doas pacman --config "${HOME}/pacman_aarch64.conf" --arch aarch64 --sync openssl zlib
  doas chown --recursive build "${TERMUX_PREFIX}"

  rustup target add aarch64-linux-android
  cargo fetch --target aarch64-linux-android --locked

  # Patches
  for d in ~/.cargo/registry/src/github.com-*/pwd-*; do
    patch --silent --strip 1 --directory ${d} < "${startdir}/crates-pwd-for-android.diff" || :
  done
  patch --forward --strip 1 --input "${startdir}/user-home-dir.patch"
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
    --workspace \
    --features=extra,dataframe
}

package() {
  cd "$pkgname-$pkgver"
  install -Dm700 -t "${pkgdir}${TERMUX_PREFIX}/bin" target/aarch64-linux-android/release/nu
  install -Dm600 -t "${pkgdir}${TERMUX_PREFIX}/share/doc/${pkgname}" LICENSE
}
