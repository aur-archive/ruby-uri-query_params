# Generated by pacgem
_gemname="uri-query_params"
_gembuilder=("install"
             "man"
             "license"
             "fix")
_ruby="/usr/bin/ruby"
_gem="/usr/bin/gem"
pkgname="ruby-uri-query_params"
pkgver="0.7.0"
pkgrel=1
pkgdesc="Access the query parameters of a URI, just like $_GET in PHP."
arch=("any")
url="http://github.com/postmodern/uri-query_params"
license=("custom:MIT")
_licensefile=("LICENSE.txt")
groups=("pacgem")
makedepends=("ruby"
             "binutils")
depends=("ruby")
conflicts=()
optdepends=("ruby-rspec: rspec-2.13.0"
            "ruby-yard: Documentation tool for consistent and usable documentation in Ruby.")
source=("http://rubygems.org/gems/$_gemname-$pkgver.gem")
sha256sums=("d83dadfc9369e354d5de3fb1ca9dbe4221fa435a9e754c5fcbd3632f27b6128c")
noextract=("$_gemname-$pkgver.gem")
options=("!emptydirs")

_gem_install() {
  msg 'Installing gem...'

  # Install the gem
  install -d -m755 $_bindir $_gemdir
  $_gem install --no-ri --no-rdoc --ignore-dependencies --no-user-install \
                --bindir $_bindir --install-dir $_gemdir "$srcdir/$_gemname-$pkgver.gem"
}

_gem_man() {
  msg 'Installing man pages...'

  # Find man pages and move them to the correct directory
  local mandir="$_gemdir/gems/$_gemname-$pkgver/man"
  if [[ -d $mandir ]]; then
    install -d -m755 $_mandir
    local file
    for file in $(find $mandir -type f -and -name *.[0-9]); do
      local dir=$_mandir/man${file##*.}
      install -d -m755 $dir
      mv $file $dir
    done
    rm -rf $mandir
  fi
}

_gem_license() {
  if [[ "${#_licensefile[@]}" -ne 0 ]]; then
    msg "Installing license $license..."
    install -d -m755 "$pkgdir/usr/share/licenses/$pkgname"
    local file
    for file in ${_licensefile[@]}; do
      ln -s "../../../..$_gemdestdir/gems/$_gemname-$pkgver/$file"  "$pkgdir/usr/share/licenses/$pkgname/$file"
    done
  fi
}

_gem_fix() {
  msg 'Fixing gem installation...'

  # Set mode of executables to 755
  [[ -d "$_gemdir/bin" ]] && find "$_gemdir/bin" -type f -exec chmod 755 -- '{}' ';'

  # Remove cached gem file
  rm -f "$_gemdir/cache/$_gemname-$pkgver.gem"

  # Sometimes there are files which are not world readable. Fix this.
  find $pkgdir -type f '!' -perm '-004' -exec chmod o+r -- '{}' ';'
}

_gem_cleanext() {
  msg 'Removing native build leftovers...'
  local extdir="$_gemdir/gems/$_gemname-$pkgver/ext"
  [[ -d $extdir ]] && find "$extdir" -name '*.o' -exec rm -f -- '{}' ';'
}

# Check if dependency is already satisfied
_dependency_satisfied() {
  local dep=$1 deps="${depends[@]}"
  [[ $(type -t in_array) == 'function' ]] || error "in_array should be provided by makepkg"
  while true; do
    in_array $dep ${deps[@]} && return 0
    local found=0 pkg
    # Warning: This could break easily if the pacman output format changes.
    for pkg in $(LC_ALL=C pacman -Qi ${deps[@]} 2>/dev/null | sed '/Depends On/!d;s/.*: //;s/None\|[<>]=\?[^ ]*\|=[^ ]*//g'); do
      if ! in_array $pkg ${deps[@]}; then
        deps=(${deps[@]} $pkg) && found=1
      fi
    done
    (( $found )) || break
  done
  return 1
}

_gem_autodepends() {
  msg 'Automatic dependency resolution...'

  # Find all referenced shared libraries
  local deps=$(find $pkgdir -type f -name '*.so')
  [[ -n $deps ]] || return 0

  deps=$(readelf -d $deps | sed -n 's/.*Shared library: \[\(.*\)\].*/\1/p' | sort | uniq)

  # Find referenced libraries on the library search path
  local libs=() lib path
  for lib in $deps; do
    for path in /lib /usr/lib; do
      [[ -f "$path/$lib" ]] && libs=(${libs[@]} "$path/$lib")
    done
  done
  (( ${#libs} )) || return 0

  msg2 "Referenced libraries: ${libs[*]}"

  # Find matching packages with pacman -Qo
  # and add them to the depends array
  local pkg
  for pkg in $(pacman -Qqo ${libs[@]}); do
    _dependency_satisfied $pkg || depends=(${depends[@]} $pkg)
  done
  msg2 "Referenced packages: ${depends[*]}"
}

_rbconfig() {
  $_ruby -e "require 'rbconfig'; puts RbConfig::CONFIG['$1']"
}

build() {
  # Directories defined inside build() because if ruby is not installed on the system
  # makepkg will barf when sourcing the PKGBUILD
  _gemdestdir=$($_gem environment gemdir)
  _gemdir=$pkgdir$_gemdestdir
  _bindir=$pkgdir$(_rbconfig bindir)
  _mandir=$pkgdir$(_rbconfig mandir)

  local i
  for i in ${_gembuilder[@]}; do
    _gem_$i
  done
}
