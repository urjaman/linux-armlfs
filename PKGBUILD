# ARMv7 ex-multi-platform -> C201 -> ARMLFS
# Original by:
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Modified by:
# Maintainer: Urja Rannikko <urjaman@gmail.com>

buildarch=4

# This is the "extension" to the base linux-c201 name -- used in zImage 
# dtb paths too
_xname=

pkgbase=linux-armlfs${_xname}
_srcname=linux-6.10
_kernelname=${pkgbase#linux}
_desc="Veyron Speedy"
pkgver=6.10.2
pkgrel=3
arch=('armv7h')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'dtc' 'vboot-utils' 'uboot-tools')
options=('!strip')
source=(
	"https://www.kernel.org/pub/linux/kernel/v${pkgver::1}.x/${_srcname}.tar.xz"
        "https://www.kernel.org/pub/linux/kernel/v${pkgver::1}.x/patch-${pkgver}.xz"
	# Those are good until 9.x (10.x will fail), fix this then :P
	'armlfs.patch'
        'kernel.its'
        'kernel.keyblock'
        'kernel_data_key.vbprivk'
        '60-linux.hook'
        'config')

prepare() {
  cd "${srcdir}/${_srcname}"

  # hack to allow git apply to work on non-git files while in a git directory
  export GIT_DIR=/var/empty

  # add upstream patch
  git apply --whitespace=nowarn ../patch-${pkgver}

  # ARMLFS patchset
  git apply ../armlfs.patch

  unset GIT_DIR
  
  cat "${srcdir}/config" > ./.config

  # Append (if -rc*) or set ${_xname}-${pkgrel} as extraversion
  sed -ri "s/^(EXTRAVERSION =) ?(-rc[^-]*|).*/\1 \2${_xname}-${pkgrel}/" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

}

build() {
  cd "${srcdir}/${_srcname}"

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  if [ -n "$_menuconfig" ]; then
    make menuconfig # CLI menu for configuration
    cp ./.config ../../config
  else
    make olddefconfig
  fi

  # ... or manually edit .config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  make -j2 ${MAKEFLAGS} zImage modules dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")

  cd "${srcdir}/${_srcname}"

  KARCH=arm

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs${_xname}" dtbs_install
  cp arch/$KARCH/boot/zImage "${pkgdir}/boot/zImage${_xname}"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build link
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/build

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
}

_package-chromebook() {
  pkgdesc="The Linux Kernel - ${_desc} - Chromebooks"
  depends=("linux-armlfs${_xname}")
  install=chromebook.install

  cd "${srcdir}/${_srcname}"

  cp ../kernel.its .
  mkimage -D "-I dts -O dtb -p 2048" -f kernel.its vmlinux.uimg || true
  dd if=/dev/zero of=bootloader.bin bs=512 count=1
  echo 'console=ttyS2,115200n8 console=tty0 init=/sbin/init root=PARTUUID=%U/PARTNROFF=1 rootwait rw noinitrd' > cmdline
  vbutil_kernel \
    --pack vmlinux.kpart \
    --version 1 \
    --vmlinuz vmlinux.uimg \
    --arch arm \
    --keyblock ../kernel.keyblock \
    --signprivate ../kernel_data_key.vbprivk \
    --config cmdline \
    --bootloader bootloader.bin

  mkdir -p "${pkgdir}/boot"
  cp vmlinux.kpart "${pkgdir}/boot"
}

pkgname=("${pkgbase}" "${pkgbase}-chromebook")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done

sha256sums=('774698422ee54c5f1e704456f37c65c06b51b4e9a8b0866f34580d86fef8e226'
            'f3166b9b9f6a7dbae9ed7e92e373c8ddb672c5bd2da3991207aa30f52ceda7fa'
            'cb7604e70e47c32840e3555871ae7b85193d4348834e8c7baf0b0c28796f68cf'
            '994aee74b13313bdc7c47df4d621c890f5ee52bc18f6c7b658de215c17423b2a'
            '4e708c9ec43ac4a5d718474c9431ba6b6da3e64a9dda6afd2853a9e9e3079ffb'
            'bc9e707a86e55a93f423e7bcdae4a25fd470b868e53829b91bbe2ccfbc6da27b'
            'ae2e95db94ef7176207c690224169594d49445e04249d2499e9d2fbc117a0b21'
            '91b60d81760bf349abc4fcc8e7660269ed5fc996a636a1b88b5328256b5c6772')
