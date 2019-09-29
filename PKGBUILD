# Maintainer: Dave Higham <pepedog@archlinuxarm.org>
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: Oleg Rakhmanov <oleg@archlinuxarm.org>

buildarch=20

pkgbase=linux-raspberrypi-${EXT_PKGBASE}
_commit=${EXT_GIT_HASH}
_srcname=linux-${_commit}
_kernelname=${pkgbase#linux}
_desc="Raspberry Pi"
# the real_pkgver is the actual kernel version of the package
# normally this should be the same as pkgver, but if we for some reason need to downgrade
# we can do so by using a 'fake' pkgver of a higher kernel than is actually provided by the real_pkgver
pkgver=4.19.75
real_pkgver=${EXT_PKG_VER}
pkgrel=1
arch=('armv7h')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git')
options=('!strip')
source=("https://github.com/raspberrypi/linux/archive/${_commit}.tar.gz"
        'config.txt'
        'cmdline.txt'
        'config'
	'config_rpi2_rpi3'
	'config_rpi4'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook'
	'kernel-usb-native-dsd-generic-detection.patch'
        'kernel-usb-native-dsd-quirks.patch'
	'kernel-alsa-support-for-384khz-sample-rates.patch'
	'kernel-sound-pcm5102a-add-support-for-384k.patch'
	'kernel-drivers-net-usb-ax88179_178a.patch'
	'kernel-add-rtl8812au-network-driver.patch')
md5sums=('dad130907d153516df6079f569ef6d1f'
         '7c6b37a1353caccf6d3786bb4161c218'
         'fcd90122a2621d0a7d6cdd020da8723d'
         'c7df4140ea594658116f7e556ebc2e75'
         '77993211f980846c8fc245b8f6aa2917'
         '9ab7ea6376ab9de17c11b7d6b0db7ccc'
         '86d4a35722b5410e3b29fc92dae15d4b'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         'ba6ee1d0a4c28fc35748013b4468c3d3'
         '59723235d523b774488ae5a5bf03f7c9'
         '1da0d841aae10c9b0711f521d8a0e282'
         'ec0778debc64a779fb674aa1231d5a58'
         '0c7adc3f558065e2f2343b973830a51e'
         '2bc8928dad8202d9acba5c1e81a0803f'
         '2d7b6bd883af73a8987c58f20c591391')

abort() {
   msg2 "$1"
   exit 1
}

prepare() {
  cd "${srcdir}/${_srcname}"

  [[ -z "$RPI_CONFIG" ]] && abort "RPI_CONFIG variable not set!"
  msg2 "setting up config: ${RPI_CONFIG}"
  cat "${srcdir}/${RPI_CONFIG}" > ./.config
#  cat "${srcdir}/bcm2709_defconfig" > ./.config

  msg2 "patching: 384k support"
  patch -Np1 -i ../kernel-alsa-support-for-384khz-sample-rates.patch
  patch -Np1 -i ../kernel-sound-pcm5102a-add-support-for-384k.patch

  msg2 "patching: kernel USB native DSD quirks"
#  patch -Np1 -i ../kernel-usb-native-dsd-generic-detection.patch
  patch -Np1 -i ../kernel-usb-native-dsd-quirks.patch

  msg2 "patching: kernel driver used by the Allo USBridge Signature"
  patch -Np1 -i ../kernel-drivers-net-usb-ax88179_178a.patch

  msg2 "Patching: add kernel driver RTL 8812AU"
  patch -Np1 -i ../kernel-add-rtl8812au-network-driver.patch

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

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
  #make oldconfig # using old config from previous kernel version
  #make bcmrpi_defconfig # using RPi defconfig
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config ../${pkgver}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  make ${MAKEFLAGS} zImage modules dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7' 'firmware-raspberrypi')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${real_pkgver}")
  conflicts=('kernel26' 'linux')
  install=${pkgname}.install
  backup=('boot/config.txt' 'boot/cmdline.txt')
  replaces=('linux-raspberrypi-latest')

  cd "${srcdir}/${_srcname}"

  KARCH=arm

  # image name
  [[ "$RPI_CONFIG" == "config_rpi2_rpi3" ]] && KERNEL_IMAGE="kernel7.img"
  [[ "$RPI_CONFIG" == "config_rpi4" ]]      && KERNEL_IMAGE="kernel7l.img"

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot" dtbs_install

  [[ $RPI_CONFIG == "config_rpi2_rpi3" ]] && cp arch/$KARCH/boot/zImage "${pkgdir}/boot/$KERNEL_IMAGE" \
                                          && rm -f "${pkgdir}"/boot/bcm{2835,2836,2837,2838,2708,2711}*.dtb

  [[ $RPI_CONFIG == "config_rpi4" ]]      && cp arch/$KARCH/boot/zImage "${pkgdir}/boot/$KERNEL_IMAGE" \
                                          && rm -f "${pkgdir}"/boot/bcm{2835,2836,2837,2838,2708,2709,2710}*.dtb


  # cp arch/$KARCH/boot/dts/overlays/README "${pkgdir}/boot/overlays"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"

  # install boot files
  install -m644 ../config.txt ../cmdline.txt "${pkgdir}/boot"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${real_pkgver}")
  conflicts=('linux-headers')
  replaces=('linux-raspberrypi-latest-headers')

  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

  cd "${srcdir}/${_srcname}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

  for i in acpi asm-generic config crypto drm generated keys linux math-emu \
    media net pcmcia scsi soc sound trace uapi video xen; do
    cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
  done

  # copy arch includes for external modules
  mkdir -p ${pkgdir}/usr/lib/modules/${_kernver}/build/arch/$KARCH
  cp -a arch/$KARCH/include ${pkgdir}/usr/lib/modules/${_kernver}/build/arch/$KARCH/

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

  # add dm headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
  cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

  # add inotify.h
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
  cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

  # add wireless headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
  cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/11194
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
  cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"

  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
  cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

  # add dvb headers
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
  cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
  cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
  cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

  # add xfs and shmem for aufs building
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"

  # copy in Kconfig files
  for i in $(find . -name "Kconfig*"); do
    mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
    cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
  done

  chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
  find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

  # strip scripts directory
  find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
      *application/x-sharedlib*) # Libraries (.so)
        /usr/bin/strip ${STRIP_SHARED} "${binary}";;
      *application/x-archive*) # Libraries (.a)
        /usr/bin/strip ${STRIP_STATIC} "${binary}";;
      *application/x-executable*) # Binaries
        /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
  done

  # remove unneeded architectures
  rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arc,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,x86,xtensa}
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
