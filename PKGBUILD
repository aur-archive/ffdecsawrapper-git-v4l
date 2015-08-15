# Submitter:   Wessel Dirksen "p-we" <wdirksen at gmail dot com>
# Contributor: Tycho Lürsen "bas-t" (responsible for hosting and development of FFdecsawrapper and more)
# Contributor: Petr Vacek "vaca" (providing cardslot.conf for serial port SC readers)
# Contributor: J.P. van Best (implementing new procfs API for kernels >= 3.10 in FFdecsawrapper kernel module)
# Contributor: "Sunday" (tweak for speeding up gzip compression)
# Contributor: Oliver Endress (providing improved mutex patch for dvbdev.c)

# !! Important:
# If you use more than 4 DVB tuners, you must change the line below to the actual number of DVB tuners you have :

_dvbtuners=4

pkgname=ffdecsawrapper-git-v4l
pkgver=3
pkgrel=2
pkgdesc="FFdecsa empowered softcam - compiled with V4L development lib's"
url="https://github.com/bas-t/ffdecsawrapper.git"
arch=('i686' 'x86_64')
license=('GPLv3')
makedepends=('git' 'patchutils' 'perl-proc-processtable' 'linux-headers')
depends=('v4l-utils')
optdepends=('oscam-svn: smartcard reader support' 'linuxtv-dvb-apps: handy DVB tools')
conflicts=('ffdecsawrapper' 'sasc-ng')
provides=('ffdecsawrapper')
backup=('etc/camdir/cardclient.conf' 'etc/conf.d/ffdecsawrapper' 'etc/camdir/cardslot.conf')
install='ffdecsawrapper.install'

# Alternately you can use "oldstable" or "master" branch by changing line below:

source=('git://github.com/bas-t/ffdecsawrapper.git#branch=master' \
	'git://linuxtv.org/media_build.git' \
	'cardclient.conf' 'cardslot.conf' 'ffdecsawrapper.conf' 'ffdecsawrapper.install' \
	'ffdecsawrapper.lr' 'ffdecsawrapper.service' 'ffdecsawrapper.rc' 'v4l-mutex-new2.patch')

sha256sums=('SKIP'
            'SKIP'
            '5c23db2b93d1accdc0b3f1612766de38bf7ede5658f6ef973706988dd71d1b81'
            '436eb5a612aa3cb9e45bb2031429f3d41eb596ed65d18659d3bd708919c61253'
            '3e4c28a68d312783761150c9bc5e8239261f9fba11f76f1ab8d072b71391c55c'
            'd6ceef7a559ad49722663b4f0b2762ff1ebce3801dd56cd801c24112c9a0cba9'
            'f435344dc9f1c0ed7c2e0de74ec434cd73e2130a0d7589a4d38338e45925d8db'
            'e798aacd050c078083a477b1fc393fa2dacf9b413ab8fea5a80449d3423c2b22'
            'a4c1169df845608596c1f20d8c5ca1cc7d9f9a03a8c2e49f5391f942007fe75d'
            '413ec60fd459a67bd527679a72024991bf74fb506db1e0e36468b5a5f253a6e6')

pkgver() {

	cd $srcdir/ffdecsawrapper
 	_gitffdecsawrapper=`git describe --always | sed 's|-|.|g'`
	cd $srcdir/media_build 
	_gitv4l=`git describe --always | sed 's|-|.|g'`
	_kernel=`uname -r | sed -r 's/-/_/g'`
	echo "$_gitffdecsawrapper"_"$_gitv4l"_"$_kernel"
}

prepare() {

	cd $srcdir/media_build
	sed -i 's/system ("make") == 0 or die "build failed";/#system ("make") == 0 or die "build failed";/' build
	./build

	_v4lconfig="s/CONFIG_DVB_MAX_ADAPTERS=8/CONFIG_DVB_MAX_ADAPTERS=$(($_dvbtuners * 2))/"
	sed -i $_v4lconfig v4l/.config
	
	msg "Compiling with number of DVB tuners set to $(($_dvbtuners * 2))..."
	msg "Applying patches..."
	patch -p1 < $srcdir/v4l-mutex-new2.patch
	sleep 4

	make
}

build() {

	cd $srcdir/ffdecsawrapper      
	./configure --dvb_dir=$srcdir/media_build/linux --update=no --tsbuffer=32
}

package() {

	mkdir -p $pkgdir/usr/bin
	mkdir -p $pkgdir/usr/lib/modules/`uname -r`/updates/{ffdecsawrapper,v4l}
	mkdir -p $pkgdir/etc/conf.d
	mkdir -p $pkgdir/etc/rc.d
	mkdir -p $pkgdir/etc/camdir
	mkdir -p $pkgdir/etc/logrotate.d
	mkdir -p $pkgdir/usr/lib/systemd/system

	install -m0644 $srcdir/cardclient.conf  $pkgdir/etc/camdir/cardclient.conf
	install -m0644 $srcdir/cardslot.conf  $pkgdir/etc/camdir/cardslot.conf
	install -m0755 $srcdir/ffdecsawrapper.rc  $pkgdir/etc/rc.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.conf  $pkgdir/etc/conf.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.lr  $pkgdir/etc/logrotate.d/ffdecsawrapper-git-v4l.lr
	install -m0644 $srcdir/ffdecsawrapper.service  $pkgdir/usr/lib/systemd/system/ffdecsawrapper.service      
	install -m0755 $srcdir/ffdecsawrapper/ffdecsawrapper  $pkgdir/usr/bin
	install -m0755 $srcdir/ffdecsawrapper/dvbloopback.ko  $pkgdir/usr/lib/modules/`uname -r`/updates/ffdecsawrapper
	find "$srcdir/media_build" -name '*.ko' -exec cp {} $pkgdir/usr/lib/modules/`uname -r`/updates/v4l \;

	msg "Compressing modules, this will take awhile..."
	find "$pkgdir" -name '*.ko' -print0 | xargs -0 -P`nproc` -n10 gzip -9

	chmod 755 -R $pkgdir/usr/lib/modules/`uname -r`/updates
}
