pkgname=deskflow
pkgver=1.27.0
pkgrel=2.2
pkgdesc='Share one mouse and keyboard between multiple computers (local: Wayland data-control clipboard fallback + per-monitor edge barriers for Hyprland)'
url='https://deskflow.org/'
arch=(x86_64)
license=('GPL-2.0-only WITH LicenseRef-OpenSSL-Exception')
depends=(
  glibc
  hicolor-icon-theme
  libei
  libgcc
  libglvnd
  libice
  libportal
  libsm
  libstdc++
  libx11
  libxext
  libxi
  libxinerama
  libxkbcommon
  libxkbcommon-x11
  libxkbfile
  libxrandr
  libxtst
  openssl
  qt6-base
  qt6-declarative
  qt6-svg
  wayland
)
makedepends=(
  cmake
  git
  ninja
  python
  qt6-tools
  wayland-protocols
  xorgproto
)
source=("git+https://github.com/deskflow/deskflow.git#tag=v${pkgver}"
        0001-wayland-data-control-clipboard.patch
        0002-portal-barriers-on-exterior-edges.patch)
sha256sums=('6842734f9edef62540910f6b18a42cc485f17b37445fc84266132502ea641e4c'
            'a2df842fa41dc92b9a36d66e2b682dd4574239ee64856a8b8235edd37b86ec7d'
            '171d9533064ee7da524e58c31396e5b5d0fb516e9da5af6b2356e951a4d13be2')
b2sums=('bec3dacaae27bd9bf1a2e16a77d8bba2fde6352e3d834130172e0c58fd0a98afa02ed7e2925da46f56b6422b2fb21d1a400e9daafdd3215b9f0ed14f363e24a9'
        'ca08764d3ebb0850bc67a5a47d87fc499db76640778da1f00118d8833f9584510b4819eff0e426ddafe077a3f25962d57ac158ab324f02bcb8ce6296d2d06e0e'
        '229284854db743e1e578f9adf0c37f0680e0ffe9e6f5da9bcafc394fde00adab3b76a367280b0fe64af45487e27ffc354aab2cbbce6d52fbd85af6b4b2b68903')

prepare() {
  cd "${pkgname}"
  git apply "${srcdir}/0001-wayland-data-control-clipboard.patch"
  git apply "${srcdir}/0002-portal-barriers-on-exterior-edges.patch"
}

build() {
  cd "${pkgname}"
  cmake \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=None \
    -DCMAKE_C_FLAGS="${CFLAGS}" \
    -DCMAKE_CXX_FLAGS="${CXXFLAGS}" \
    -DCMAKE_EXE_LINKER_FLAGS="${LDFLAGS}" \
    -DCMAKE_SHARED_LINKER_FLAGS="${LDFLAGS}" \
    -Wno-dev \
    -G Ninja \
    -B build \
    -S .
  cmake --build build --verbose
}

check() {
  cd "${pkgname}"
  export QT_QPA_PLATFORM=offscreen
  ctest --test-dir  "./build/src/unittests" --output-on-failure
}

package() {
  cd "${pkgname}"
  DESTDIR="${pkgdir}" cmake --install build
}

# vim: ts=2 sw=2 et:
