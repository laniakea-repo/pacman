# Maintainer: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer: Morten Linderud <foxboron@archlinux.org>

pkgname=pacman
pkgver=7.0.0.r6.gc685ae6
pkgrel=1
# use annotated tag and patch level commit from release branch (can be empty for no patches)
_git_tag=7.0.0
_git_patch_level_commit=c685ae6412af04cae1eaa5d6bda8c277c7ffb8c8
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="https://www.archlinux.org/pacman/"
license=('GPL-2.0-or-later')
depends=(
  bash
  coreutils
  curl
  gawk
  gettext
  glibc
  gnupg
  gpgme
  grep
  libarchive
  systemd
)
makedepends=(
  asciidoc
  doxygen
  git
  meson
)
checkdepends=(
  fakechroot
  python
)
optdepends=(
  'base-devel: required to use makepkg'
  'perl-locale-gettext: translation support in makepkg-template'
)
provides=('libalpm.so')
backup=(etc/pacman.conf
        etc/makepkg.conf
        etc/makepkg.conf.d/rust.conf)
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121') # Andrew Gregory (pacman) <andrew@archlinux.org>
source=("git+https://gitlab.archlinux.org/pacman/pacman.git#tag=v${_git_tag}?signed"
        revertme-makepkg-remove-libdepends-and-libprovides.patch::https://gitlab.archlinux.org/pacman/pacman/-/commit/354a300cd26bb1c7e6551473596be5ecced921de.patch
        pacman.conf
        makepkg.conf
        alpm.sysusers
        rust.conf)
sha256sums=('06d082c3ce6f0811ca728515aa82d69d372800bd3ada99f5c445ef9429b6e3a6'
            'b3bce9d662e189e8e49013b818f255d08494a57e13fc264625f852f087d3def2'
            '0e84952e4b8eacbb38c018608d152ddd6f98e205c4c6c7d3cdca854d4b7d4179'
            'd99c1f9608362fff9ab3a2ca0a3096a317927b42a6725bc86599da6849c9c67c'
            'c8760d7ebb6c9817d508c691c67084be251cd9c8811ee1ccf92c1278bad74c1c'
            '59a8bbdc32830c90dbce5ead9f90074a2bb31cd85ec92bc5f92251bb4bcf8a1d')

pkgver() {
  cd "$pkgname"
  git describe --abbrev=7 --match 'v*' | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
  cd "$pkgname"

  # apply patch level commits on top of annotated tag
  if [[ -n ${_git_patch_level_commit} ]]; then
    if [[ v${_git_tag} != $(git describe --tags --abbrev=0 "${_git_patch_level_commit}") ]] then
      error "patch level commit ${_git_patch_level_commit} is not a descendant of v${_git_tag}"
      exit 1
    fi
    git rebase "${_git_patch_level_commit}"
  fi

  # handle patches
  local -a patches
  patches=($(printf '%s\n' "${source[@]}" | grep '.patch'))
  patches=("${patches[@]%%::*}")
  patches=("${patches[@]##*/}")

  if (( ${#patches[@]} != 0 )); then
    for patch in "${patches[@]}"; do
      if [[ $patch =~ revertme-* ]]; then
        msg2 "Reverting patch $patch..."
        patch -RNp1 < "../$patch"
      else
        msg2 "Applying patch $patch..."
        patch -Np1 < "../$patch"
      fi
    done
  fi
}

build() {
  cd "$pkgname"

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Ddoxygen=enabled \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        build

  meson compile -C build
}

check() {
  cd "$pkgname"

  meson test -C build
}

package() {
  cd "$pkgname"

  DESTDIR="$pkgdir" meson install -C build

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"
  install -D -m644 "$srcdir/alpm.sysusers" "${pkgdir}"/usr/lib/sysusers.d/alpm.conf
  install -m644 "$srcdir/rust.conf" "$pkgdir/etc/makepkg.conf.d"

  local wantsdir="$pkgdir/usr/lib/systemd/system/sockets.target.wants"
  install -dm755 "$wantsdir"

  local unit
  for unit in dirmngr gpg-agent gpg-agent-{browser,extra,ssh} keyboxd; do
    ln -s "../${unit}@.socket" "$wantsdir/${unit}@etc-pacman.d-gnupg.socket"
  done
}

# vim: set ts=2 sw=2 et:
