# SPDX-License-Identifier: AGPL-3.0
#
# Maintainer: Truocolo <truocolo@aol.com>
# Maintainer: Pellegrino Prevete (tallero) <pellegrinoprevete@gmail.com>
# Maintainer: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer: David Runge <dvzrv@archlinux.org>
# Contributor: Christian Rebischke <Chris.Rebischke@archlinux.org>
# Contributor: Daniel Micay <danielmicay@gmail.com>
# Contributor: <kang@insecure.ws>
# Contributor: Massimiliano Torromeo <massimiliano.torromeo@gmail.com>
# Contributor: Connor Behan <connor.behan@gmail.com>
# Contributor: henning mueller <henning@orgizm.net>

_systemd="yes"
_ldap="libldap"
_apparmor="yes"
_python="yes"
_py="python"
_os="$( \
  uname \
  -o)"
[[ "${_os}" == "Android" ]] && \
  _apparmor="no" \
  _ldap="openldap" \
  _systemd="no"
_pkg="audit"
pkgbase="${_pkg}"
_name="${_pkg}-userspace"
pkgname=(
  "${_pkg}"
)
[[ "${_python}" == "yes" ]] && \
  pkgname+=(
    "${_py}-${_pkg}"
  )
pkgver=4.0
pkgrel=1
pkgdesc='Userspace components of the audit framework'
url="https://people.redhat.com/sgrubb/${_pkg}"
arch=(
  x86_64
  arm
  aarch64
)
license=(
  GPL-2.0-or-later
  LGPL-2.0-or-later
)
makedepends=(
  glibc
  krb5
  libcap-ng
  "${_ldap}"
  linux-api-headers
  "${_py}"
  swig
)
[[ "${_apparmor}" == yes ]] && \
  makedepends+=(
    apparmor
  )
options=(
  emptydirs
)
_url="https://github.com/linux-${_pkg}/${_name}"
source=(
  "${_url}/archive/v${pkgver}/${_name}-v${pkgver}.tar.gz"
  $pkgbase.tmpfiles
  $pkgbase-4.0-executable_paths.patch
)
sha512sums=(
  'f001e3f466e012dc9c41e8aefd19ffac0db73c8510cd20467e2eb78dcf7a0fdde64279ec2ff0c370ecba692cda2ae228543b8e7ecdae06863ef37f6fa3a7c7c3'
  '1750741755f58d0ae19ed2c30e136d05560dc12ec613a502bad12f47c6b70432d30b3a16f3f1574c8433ad2701428d1c1d567a4d3b55be19abac300310c831d9'
  '5c1b524bf86234eac690cbe073e5b5459d3f74ec58ef57d50250261b0b1ca4656f295f410bf0727242ed852e725e6acd4438a4a02993c21a6fc80c132eb745b1'
)
b2sums=(
  'c8ab1b241134dfc16f4a440358203b095b376a5f2042c6434a22d4127b3ae0751759b2f457bab50b81969368daa218bedbcd4cce815aec5fe6d13a62e9363d57'
  '549ebbbc9e43277d44d0dc5bfd8ca2926628322d898479171b2707dd004968d036ef792b442548af90ad56dea868a72c88b5cf3bb93ea70cb8bbed82747ad9b5'
  '36e9c74eeccda534a019f780d7aa5337fc24118129c2766203a8bb028a81ba2bdf057cbc78194bcaeeda28c7ae9ef4ef8bb9cd34a37e47655a4ed2b64381935d'
)

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

prepare() {
  # use /usr and /bin merge compatible paths in configs and services
  patch \
    -Np1 \
    -d \
    $pkgbase-userspace-$pkgver \
    -i \
    ../$pkgbase-4.0-executable_paths.patch
  cd \
    $_name-$pkgver
  libtoolize
  autoreconf \
    -fiv
}

build() {
  local \
    _configure_opts=()
  _configure_opts=(
    --enable-gssapi-krb5=yes
    --enable-zos-remote
    --libexecdir=/usr/lib/audit
    --prefix=/usr
    --sbindir=/usr/bin
    --sysconfdir=/etc
    --with-apparmor="${_apparmor}"
    --with-io_uring=yes
    --with-libcap-ng=yes
    --with-python3="${_python}"
  )
  if [[ "${_systemd}" == "yes" ]]; then
    _configure_opts+=(
     --enable-systemd
   )
  fi
  cd \
    $_name-$pkgver
  ./configure \
    "${_configure_opts[@]}"
  # prevent excessive overlinking due to libtool
  sed \
    -i \
    -e \
      's/ -shared / -Wl,-O1,--as-needed\0/g' \
    libtool
  make
}

package_audit() {
  depends=(
    glibc
    krb5
    libkrb5.so
    libgssapi_krb5.so
    libcap-ng
    libcap-ng.so
  )
  optdepends=(
    'libldap: for audispd-zos-remote'
    'sh: for augenrules'
  )
  provides=(
    libaudit.so
    libauparse.so
  )
  backup=(
    etc/libaudit.conf
    etc/audit/audit-stop.rules
    etc/audit/auditd.conf
    etc/audit/audisp-remote.conf
    etc/audit/zos-remote.conf
    etc/audit/plugins.d/af_unix.conf
    etc/audit/plugins.d/au-remote.conf
    etc/audit/plugins.d/audispd-zos-remote.conf
    etc/audit/plugins.d/syslog.conf
  )

  make \
    DESTDIR="$pkgdir" \
    install \
    -C \
      $_name-$pkgver
  install \
    -vDm644 \
    $_name-$pkgver/{{README,SECURITY}.md,ChangeLog} \
    -t \
    "$pkgdir/usr/share/doc/$pkgname/"
  # add log dir
  install \
    -vdm 755 \
    "$pkgdir/var/log/$pkgname/"
  # add rules.d dir to satisfy augenrules
  install \
    -vdm 755 \
    "$pkgdir/etc/audit/rules.d/"
  # add config dir for audisp
  install \
    -vdm 755 \
    "$pkgdir/etc/audisp"
  # add factory files
  install \
    -vdm 755 \
    "$pkgdir/usr/share/factory/"
  cp \
    -av \
    "$pkgdir/etc" \
    "$pkgdir/usr/share/factory/"
  # add tmpfiles.d integration for factory files and file permissions
  install \
    -vDm 644 \
    $pkgbase.tmpfiles \
    "$pkgdir/usr/lib/tmpfiles.d/$pkgbase.conf"
  # remove legacy files
  rm \
    -frv \
    "$pkgdir/usr/lib/audit"
  (
    cd \
      "$pkgdir"
    _pick \
      python-audit \
      usr/lib/python*
  )
}

package_python-audit() {
  pkgdesc+=' - Python bindings'
  depends=(
    audit
    libaudit.so
    libauparse.so
    glibc
    python
  )
  mv \
    -v \
    "${pkgname}/"* \
    "${pkgdir}"
}

# vim: ts=2 sw=2 et:
