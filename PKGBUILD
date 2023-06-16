pkgname=nushell
pkgver=0.79.0
_commit=a1b72611215dbfca257351003204a80c83859e05
pkgrel=1
pkgdesc='A new type of shell'
arch=('aarch64')
url='https://www.nushell.sh'
license=('MIT')
depends=(openssl zlib)
makedepends=(cargo git)
source=("git+https://github.com/nushell/nushell.git#commit=$_commit")
sha256sums=('SKIP')

TERMUX_PREFIX='/data/data/com.termux/files/usr'

prepare() {
  cd "$pkgname"

  sudo find '/opt/android-ndk/' -name 'libunwind.a' -execdir sh -c 'echo "INPUT(-lunwind)" > libgcc.a' \;

  yes | sudo pacman --config "${HOME}/pacman_aarch64.conf" --arch aarch64 --sync openssl zlib
  sudo chown --recursive build "${TERMUX_PREFIX}"

  rustup target add aarch64-linux-android
  cargo fetch --target aarch64-linux-android --locked

  # Patches
  patch --forward --strip 1 --input "${startdir}/extern-with-blocks.patch"

  for d in ~/.cargo/registry/src/github.com-*/pwd-*; do
    patch --silent --strip 1 --directory ${d} < "${startdir}/crates-pwd-for-android.diff" || :
  done
  patch --forward --strip 1 --input "${startdir}/user-home-dir.patch"
}

pkgver() {
  cd "$pkgname"
  git describe --tags
}

build() {
  cd "$pkgname"

  export PATH="/opt/android-ndk/toolchains/llvm/prebuilt/linux-x86_64/bin:$PATH"
  export AR_aarch64_linux_android='/opt/android-ndk/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-ar'

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
    --features dataframe
}

package() {
  cd "$pkgname"
  install -Dm700 -t "${pkgdir}${TERMUX_PREFIX}/bin" target/aarch64-linux-android/release/nu
  install -Dm600 -t "${pkgdir}${TERMUX_PREFIX}/share/doc/${pkgname}" LICENSE
}
