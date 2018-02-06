# Maintainer: Det <nimetonmaili g-mail>
# Based on [extra]'s thunderbird: https://git.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/thunderbird

pkgname=thunderbird-beta
pkgver=59.0b1
_major=${pkgver/[br]*}
_build=${pkgver/*rc}
pkgrel=1
pkgdesc="Standalone mail and news reader from mozilla.org - Bleeding edge version"
arch=('x86_64')
license=('MPL' 'GPL' 'LGPL')
url="https://www.mozilla.org/thunderbird/"
depends=(gtk3 mozilla-common libxt startup-notification mime-types dbus-glib alsa-lib ffmpeg
         nss sqlite ttf-font icu libvpx)
makedepends=(gtk2 unzip zip diffutils python2 yasm mesa imake gconf libpulse inetutils xorg-server-xvfb
             autoconf2.13 cargo clang llvm hunspell)
optdepends=('hunspell: Spell checking'
            'hyphen: Hyphenation'
            'libcanberra: Sound support')
options=(!emptydirs !makeflags)
install=$pkgname.install
source=(
    "https://ftp.mozilla.org/pub/mozilla.org/thunderbird/releases/$pkgver/source/thunderbird-$pkgver.source.tar.xz"
    "$pkgname.desktop"
    'thunderbird-install-dir.patch'
    'no-crmf.diff'
)
sha512sums=('384d78346413c66b02573a6138bde223b6388a306ba535eaf5498867a4dd7bb44b73dda4e7811b728202df0283813bc3365317007a170696bdba0703beb83215'
            'e5649ddee3ca9cfdcf56652e9c8e6160d52c69d1439f9135b0c0d436ce61a25f17758afc0dd6cac3434c26234c584828eb07fdf9604797f7dd3f617ec194b79a'
            '08f14d57db962d4c5060daf6ff3413bf7e2b1d79c60ea522235acbd3ccdfa1955e75fb9d0520aaca8a96fb33cb5093e13b09f51aa116dec999dee34224df9f8b'
            '951667941520e66e7b6aad55619ec2b38364da58c5cf8a71775a3032921cfc0a8e5c7ba14e0df35588175f94a6b4785566d39177ff536ab9cefcbd19a03dc065')
# RC
if [[ $_build = ? ]]; then
  source[0]="https://ftp.mozilla.org/pub/thunderbird/candidates/$_major-candidates/build$_build/source/thunderbird-$_major.source.tar.xz"
fi

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

# Mozilla API keys (see https://location.services.mozilla.com/api)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact heftig@archlinux.org for
# more information.
_mozilla_api_key=16674381-f021-49de-8622-3021c5942aff

prepare() {
  # Link Python2
  mkdir -p path
  ln -sf /usr/bin/python2 path/python

  cd thunderbird-$pkgver

  msg2 "thunderbird-install-dir.patch"
  patch -Np1 -i ../thunderbird-install-dir.patch

  msg2 "no-crmf.diff: https://bugzilla.mozilla.org/show_bug.cgi?id=1371991"
  patch -Np1 -i ../no-crmf.diff

  # API keys
  echo -n "$_google_api_key" >google-api-key
  echo -n "$_mozilla_api_key" >mozilla-api-key

  # mozconfig
  cat >.mozconfig <<END
ac_add_options --enable-application=mail
ac_add_options --enable-calendar

ac_add_options --prefix=/opt
ac_add_options --libdir=/opt
ac_add_options --enable-release
ac_add_options --enable-gold
ac_add_options --enable-pie
ac_add_options --enable-optimize="-O2"
#ac_add_options --enable-rust

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=beta
ac_add_options --with-distribution-id=org.archlinux

# Keys
ac_add_options --with-google-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-mozilla-api-keyfile=${PWD@Q}/mozilla-api-key

# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss
ac_add_options --with-system-icu
ac_add_options --with-system-jpeg
ac_add_options --with-system-zlib
ac_add_options --with-system-bz2
ac_add_options --with-system-libvpx
ac_add_options --enable-system-hunspell
ac_add_options --enable-system-sqlite
ac_add_options --enable-system-ffi
ac_add_options --enable-system-pixman

# Features
ac_add_options --enable-startup-notification
ac_add_options --disable-crashreporter
ac_add_options --enable-alsa
ac_add_options --disable-updater
ac_add_options --disable-tests ###
ac_add_options --disable-debug-symbols ###

STRIP_FLAGS="--strip-debug"
END
}

build() {
  cd thunderbird-$pkgver

  # _FORTIFY_SOURCE causes configure failures
  CPPFLAGS+=" -O2"

  # Hardening
  LDFLAGS+=" -Wl,-z,now"

  # Export build path
  export PATH="$srcdir/path:$PATH"

  # Do PGO
  #xvfb-run -a -n 95 -s "-extension GLX -screen 0 1280x1024x24" \
  #  make -f client.mk build MOZ_PGO=1
  
  # Build
  msg2 "Running make -f client.mk build.."
  make -f client.mk build
}

package() {
  cd thunderbird-$pkgver

  # Install
  msg2 "Running make -f client.mk install.."
  make -f client.mk DESTDIR="$pkgdir" INSTALL_SDK= install

  # vendor.js
  _vendorjs="$pkgdir/opt/$pkgname/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.matchOS", true);

// Disable default mailer checking.
pref("mail.shell.checkDefaultMail", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);
END

  _distini="$pkgdir/usr/lib/$pkgname/distribution/distribution.ini"
  install -Dm644 /dev/stdin "$_distini" <<END
[Global]
id=archlinux
version=1.0
about=Mozilla Thunderbird for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$pkgname
app.partner.archlinux=archlinux
END

  # Icons
  for i in 16 22 24 32 48 256; do
    install -Dm644 other-licenses/branding/thunderbird/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgname.png"
  done

  install -Dm644 ../$pkgname.desktop \
    "$pkgdir/usr/share/applications/$pkgname.desktop"

  # Use system-provided dictionaries
  rm -r "$pkgdir"/opt/$pkgname/dictionaries
  ln -Ts /usr/share/hunspell "$pkgdir/opt/$pkgname/dictionaries"
  ln -Ts /usr/share/hyphen "$pkgdir/opt/$pkgname/hyphenation"

  # Install a wrapper to avoid confusion about binary path
  install -Dm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" <<END
#!/bin/sh
exec /opt/$pkgname/thunderbird "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srf "$pkgdir/usr/bin/$pkgname" \
    "$pkgdir/opt/$pkgname/thunderbird-bin"

  # Use system certificates
  ln -srf "$pkgdir/usr/lib/libnssckbi.so" \
    "$pkgdir/opt/$pkgname/libnssckbi.so"
}
