# $Id$
# Maintainer: Paul Mattal <paul@archlinux.org>
# Contribuildor : lh <jarryson@gmail.com>

pkgname=lirc-utils-oss
pkgver=0.8.3
pkgrel=1
pkgdesc="Linux Infrared Remote Control utils without ALSA dependence"
arch=(i686 x86_64)
url="http://www.lirc.org/"
license=('GPL')
_kernver=2.6.26-ARCH
depends=('libusb' 'libx11' 'libsm')
makedepends=('help2man')
replaces=('lirc+pctv' 'lirc-utils')
provides=("lirc-utils=$pkgver")
conflicts=("lirc-utils")
backup=('etc/lircd.conf' 'etc/lircmd.conf'\
        'etc/conf.d/lircd')
options=('!libtool' '!makeflags')
source=(http://umn.dl.sf.net/sourceforge/lirc/lirc-$pkgver.tar.bz2 \
	lircd lircmd lirc.logrotate lircd.conf.d kernel-2.6.26.patch)
md5sums=('8e78eeded7b31e5ad02e328970437c0f' '909ad968afa10e4511e1da277bb23c3b'\
         '85f7fdac55e5256967241864049bf5e9' '3deb02604b37811d41816e9b4385fcc3'\
         '5b1f8c9cd788a39a6283f93302ce5c6e' '1753acd774f50b638e6173d364de53fd')

build() {
	export MAKEFLAGS="-j1"
	# configure
	cd $startdir/src/lirc-$pkgver || return 1
	patch -Np1 -i ../kernel-2.6.26.patch || return 1

      # Disabling lirc_gpio driver as it does no longer work Kernel 2.6.22+
	sed -i -e "s:lirc_gpio\.o::" drivers/lirc_gpio/Makefile.am || return 1

	autoreconf || return 1
	libtoolize || return 1

	./configure --enable-sandboxed --prefix=/usr \
	--with-driver=all --with-kerneldir=/usr/src/linux-${_kernver} \
	--with-moduledir=/lib/modules/${_kernver}/kernel/drivers/misc \
	--with-transmitter \
		|| return 1
	# disable parallel and bt829
        # because of incompatibility with smp systems
        sed -i -e "s:lirc_parallel::" -e "s:lirc_bt829::" \
		Makefile drivers/Makefile drivers/*/Makefile tools/Makefile \
                || return 1

  	# build
	make || return 1
	make DESTDIR=$startdir/pkg install || return 1
	mkdir -p $startdir/pkg/usr/share/lirc $startdir/pkg/etc/rc.d \
		|| return 1
	cp $startdir/src/{lircd,lircmd} $startdir/pkg/etc/rc.d/ \
		|| return 1
	cp -rp remotes $startdir/pkg/usr/share/lirc || return 1
	chmod -R go-w $startdir/pkg/usr/share/lirc/ || return 1

	# install the logrotate config
	install -D -m644 $startdir/src/lirc.logrotate \
		$startdir/pkg/etc/logrotate.d/lirc || return 1
	
	# install conf.d file
	install -D -m644 $startdir/src/lircd.conf.d \
		$startdir/pkg/etc/conf.d/lircd || return 1

	# remove built modules
	rm -r $startdir/pkg/lib/
}
