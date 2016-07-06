# Maintainer: Apokalypzx <apokalypzx at gmail dot com>
# Contributor: Kevin Mihelich <kevin@archlinuxarm.org>
# Contributor: Sergej Pupykin <pupykin.s+arch@gmail.com>
# Contributor: BlackIkeEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Brad Fanella <bradfanella@archlinux.us>
# Contributor: [vEX] <niechift.dot.vex.at.gmail.dot.com>
# Contributor: Zeqadious <zeqadious.at.gmail.dot.com>
# Contributor: BlackIkeEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Maxime Gauduin <alucryd@gmail.com>
# Contributor: Owersun <shiva at maill dot ru> 
# Contributor: Holzhaus <holthuis dot jan at googlemail dot com>
# Contributor: Mdrjr <mauro dot ribeiro at hardkernel dot com>

buildarch=4

_prefix=/usr

pkgbase=kodi-c1-fb
pkgname=('kodi-c1-fb' 'kodi-c1-eventclients-fb')
_commit='cdb7704d1174395f399657fd3f562fe236516b9f'
pkgver=16.1
pkgrel=1
arch=('armv7h')
url="http://kodi.tv"
license=('GPL2')
makedepends=(
    'hicolor-icon-theme' 'fribidi' 'lzo2' 'smbclient' 'libtiff'
    'libva' 'libpng' 'libcdio' 'yajl' 'libmariadbclient'
    'libjpeg-turbo' 'libsamplerate' 'libssh' 'libmicrohttpd'
    'sdl_image' 'python2' 'python2-pillow' 'python2-pybluez'
    'python2-simplejson' 'libass' 'libmpeg2' 'libmad' 'libmodplug'
    'jasper' 'rtmpdump' 'unzip' 'xorg-xdpyinfo' 'libbluray'
    'libnfs' 'afpfs-ng' 'avahi' 'bluez-libs' 'tinyxml' 'libcec'
    'libplist' 'swig' 'taglib' 'libxslt' 'shairplay' 'boost'
    'cmake' 'gperf' 'nasm' 'zip' 'udisks' 'upower' 'git'
    'autoconf' 'java-environment' 'odroid-c1-libgl-fb'
    'aml-libs-c1'
)
source=("https://github.com/owersun/xbmc/archive/${_commit}.tar.gz"
        "git+https://github.com/mdrjr/c1_mali_libs.git"
        'kodi.service'
        'polkit.rules'
        'kodi_permissions.conf'
        'gcc6_fix.patch'
	'squish_build.patch')

sha256sums=('a56165257af0ee6899317093c412603b01967068536ae200dfcab0b74bdc501d'
            'SKIP'
            'b4a8bcef695b6c53805a076d11528552f5ad053059216bc31343cffa284e4221'
            'c68ed2bd377f80b606b8815d78239b9132b479eafc1d19797cee5824debe1800'
            '1bc168887b08a9b957f6770627437ed03b00891b3fd61e3475aed0d0cd05819e'
            'b0fe75d10b2678894d1dec48f3258c0bec2a4a170f33d76a9a8334bb1969b18f'
            'f68eb917544e0c646f47b1c2a4fa55f6139a0024f4f89716d130f4d6ff7ec5e8')

prepare() {
  cd xbmc-${_commit}

  # Use Python 2 instead of Python 3
  find -type f -name *.py -exec sed 's|^#!.*python$|#!/usr/bin/python2|' -i "{}" +
  sed 's|^#!.*python$|#!/usr/bin/python2|' -i tools/depends/native/rpl-native/rpl
  sed 's/python/python2/' -i tools/Linux/kodi.sh.in
  patch -Np1 -i ${srcdir}/gcc6_fix.patch
  cd ..
  patch xbmc-${_commit}/tools/depends/native/libsquish-native/Makefile squish_build.patch
}

build() {
  cd xbmc-${_commit}
  MALI_INCLUDE=$srcdir/c1_mali_libs/fbdev/mali_headers
  FLAGS="-mcpu=cortex-a5 -mtune=cortex-a5 -mfpu=neon-vfpv4 \
         -Ofast -fexcess-precision=fast -mfloat-abi=hard \
         -pipe -fstack-protector --param=ssp-buffer-size=4 \
         -mvectorize-with-neon-quad -I$MALI_INCLUDE"
  export CFLAGS=$FLAGS
  export CXXFLAGS=$FLAGS
  export LDFLAGS="$LDFLAGS -L/usr/lib/mali-egl \
                  -L/usr/lib/aml_libs"

  # Bootstrapping
  MAKEFLAGS=-j1 ./bootstrap

  #./configure --help
  #return 1

  # Configuring KODI
  export PYTHON_VERSION=2  # external python v2
  ./configure --prefix=$_prefix \
    gl_cv_func_gettimeofday_clobber=no ac_cv_lib_bluetooth_hci_devid=no \
    --disable-static --enable-shared \
    --with-lirc-device=/run/lirc/lircd \
    --enable-optimizations \
    --enable-libbluray \
    --enable-rsxs \
    --enable-codec=amcodec \
    --enable-pulse \
    --enable-gles \
    --enable-airplay \
    --enable-airtunes \
    --enable-avahi \
    --enable-dvdcss \
    --enable-nfs \
    --enable-rtmp \
    --enable-optical-drive \
    --enable-libcec \
    --disable-debug \
    --disable-x11 \
    --disable-xrandr \
    --disable-gl \
    --disable-openmax \
    --disable-vdpau \
    --disable-vaapi \
    --disable-tegra \
    --disable-texturepacker \
    --disable-sdl \
    --disable-goom \
    --disable-joystick \
    --disable-mid \
    --disable-profiling \
    --disable-projectm

# Build KODI
# Make outputs to build.log instead of stdout.
make -k > build.log 2>&1
}

package_kodi-c1-fb() {
  pkgdesc="A software media player and entertainment hub for digital media (ODROID-C1 Framebuffer)"

  # depends expected for kodi plugins:
  # 'python2-pillow' 'python2-pybluez' 'python2-simplejson'
  # depends expeced in FEH.py
  # 'mesa-demos' 'xorg-xdpyinfo'
  depends=(
    'hicolor-icon-theme' 'fribidi' 'lzo2' 'smbclient' 'libtiff'
    'libva' 'libpng' 'libcdio' 'yajl' 'libmariadbclient' 
    'libjpeg-turbo' 'libsamplerate' 'libssh' 'libmicrohttpd'
    'sdl_image' 'python2' 'python2-pillow' 'python2-pybluez'
    'python2-simplejson' 'libass' 'libmpeg2' 'libmad' 
    'libmodplug' 'jasper' 'rtmpdump' 'unzip' 'xorg-xdpyinfo'
    'libbluray' 'libnfs' 'avahi' 'bluez-libs' 'tinyxml' 'libcec'
    'libplist' 'swig' 'taglib' 'libxslt' 'shairplay'
    'odroid-c1-libgl-fb' 'aml-libs-c1'
  )
  optdepends=(
    'afpfs-ng: Apple shares support'
    'lirc: Remote controller support'
    'udisks: Automount external drives'
    'unrar: Archives support'
    'upower: Display battery level'
    'polkit: Power menu functionality.'
    'lsb-release: log distro information in crashlog'
    'pulseaudio: Pulseaudio support.'
  )
  install="kodi.install"
  provides=('xbmc' 'kodi')
  conflicts=('xbmc' 'kodi')
  replaces=('xbmc')

  cd xbmc-${_commit}
  # Running make install
  make DESTDIR="$pkgdir" install

  # Licenses
  install -dm755 ${pkgdir}${_prefix}/share/licenses/${pkgname}
  for licensef in LICENSE.GPL copying.txt; do
    mv ${pkgdir}${_prefix}/share/doc/kodi/${licensef} \
      ${pkgdir}${_prefix}/share/licenses/${pkgname}
  done

install -Dm0644 $srcdir/kodi_permissions.conf $pkgdir/etc/tmpfiles.d/kodi_permissions.conf
install -Dm0644 $srcdir/kodi.service $pkgdir/usr/lib/systemd/system/kodi.service
install -Dm0644 $srcdir/polkit.rules $pkgdir/etc/polkit-1/rules.d/10-kodi.rules
}

package_kodi-c1-eventclients-fb() {
  pkgdesc="Kodi Event Clients (ODROID-C1)"
  provides=('kodi-eventclients')
  conflicts=('kodi-eventclients')
  depends=('cwiid')

  cd xbmc-${_commit}

  make DESTDIR="$pkgdir" eventclients WII_EXTRA_OPTS=-DCWIID_OLD

  install -dm755 "$pkgdir/usr/lib/python2.7/$pkgbase"
  mv "$pkgdir/kodi"/* "$pkgdir/usr/lib/python2.7/$pkgbase"
  rmdir "$pkgdir/kodi"
}
