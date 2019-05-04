# Maintainer: Hal Gentz <zegentzy@protonmail.com>
# Contributor: AndyRTR <andyrtr@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>

pkgbase=xorg-server-git
pkgname=('xorg-server-git' 'xorg-server-xephyr-git' 'xorg-server-xdmx-git' 'xorg-server-xvfb-git' 'xorg-server-xnest-git'
         'xorg-server-xwayland-git' 'xorg-server-common-git' 'xorg-server-devel-git')
pkgver=1.20.0.326.r16960.gdd1aebccf
pkgrel=1
arch=('x86_64')
license=('custom')
groups=('xorg')
url="http://xorg.freedesktop.org"
makedepends=(
    'xorgproto' 'pixman' 'libx11' 'mesa' 'mesa-libgl' 'xtrans' 'libxkbfile' 
    'libxfont2' 'libpciaccess' 'libxv' 'libxmu' 'libxrender' 'libxi' 'libxaw' 
    'libdmx' 'libxtst' 'libxres' 'xorg-xkbcomp' 'xorg-util-macros' 
    'xorg-font-util' 'libepoxy' 'xcb-util' 'xcb-util-image' 
    'xcb-util-renderutil' 'xcb-util-wm' 'xcb-util-keysyms' 'libxshmfence' 
    'libunwind' 'systemd' 'wayland-protocols' 'egl-wayland' 'meson' 'git'
) 
source=(
    git://anongit.freedesktop.org/xorg/xserver
    xvfb-run # with updates from FC master
    xvfb-run.1
)
validpgpkeys=(
    '7B27A3F1A6E18CD9588B4AE8310180050905E40C'
    'C383B778255613DFDB409D91DB221A6900000011'
    'DD38563A8A8224537D1F90E45B8A2D50A0ECD0D3'
    '995ED5C8A6138EB0961F18474C09DD83CAAA50B2'
)
sha512sums=('SKIP'
            '55bbf520333f6e818b0125b37179a7039b69a0d3d2242b80a08da003d94cbf6c1fb912d880abcce318a85d7947e3eff8fbc4cdf57d7118572e8ebc56c4569af6'
            'de5e2cb3c6825e6cf1f07ca0d52423e17f34d70ec7935e9dd24be5fb9883bf1e03b50ff584931bd3b41095c510ab2aa44d2573fd5feaebdcb59363b65607ff22')
groups=('gentz_custom')

#mesonFlags="-D b_ndebug=true"

pkgver() {
    cd xserver

    echo $(git describe --long | cut -d "-" -f3-4 | tr - .).r$(git rev-list HEAD --count).$(git describe --long | cut -d "-" -f5)
}

prepare() {
    # Since pacman 5.0.2-2, hardened flags are now enabled in makepkg.conf
    # With them, module fail to load with undefined symbol.
    # See https://bugs.archlinux.org/task/55102 / https://bugs.archlinux.org/task/54845
    export CFLAGS=${CFLAGS/-fno-plt}
    export CXXFLAGS=${CXXFLAGS/-fno-plt}
    export LDFLAGS=${LDFLAGS/,-z,now}

    export CFLAGS="$CFLAGS -fplt -fno-lto"
    export CXXFLAGS="$CXXFLAGS -fplt -fno-lto"
    export LDFLAGS="$LDFLAGS,-fno-lto"

    arch-meson xserver build \
        $mesonFlags \
        -D os_vendor="Arch Linux" \
        -D ipv6=true \
        -D dmx=true \
        -D xvfb=true \
        -D xnest=true \
        -D xcsecurity=true \
        -D xorg=true \
        -D xephyr=true \
        -D xwayland=true \
        -D xwayland_eglstream=true \
        -D glamor=true \
        -D udev=true \
        -D systemd_logind=true \
        -D suid_wrapper=true \
        -D xkb_dir=/usr/share/X11/xkb \
        -D xkb_output_dir=/var/lib/xkb

    # Print config
    meson configure build
}

build() {
    msg2 "Please confirm"
    for VAR in VIDEODRV XINPUT EXTENSION; do
        echo "X-ABI-${VAR}_VERSION=$(grep -Po "${VAR}_V.*\(\K[^)]*" xserver/hw/xfree86/common/xf86Module.h |& sed 's/, /./')"
    done

    export CFLAGS=${CFLAGS/-fno-plt}
    export CXXFLAGS=${CXXFLAGS/-fno-plt}
    export LDFLAGS=${LDFLAGS/,-z,now}
    ninja -C build

    # fake installation to be seperated into packages
    DESTDIR="${srcdir}/fakeinstall" ninja -C build install
}

_install() {
    local src f dir
    for src; do
        f="${src#fakeinstall/}"
        dir="${pkgdir}/${f%/*}"
        install -m755 -d "${dir}"
        mv -v "${src}" "${dir}/"
    done
}

package_xorg-server-common-git() {
    pkgdesc="Xorg server common files"
    depends=(xkeyboard-config xorg-xkbcomp xorg-setxkbmap)
    conflicts=('xorg-server-common')
    provides=('xorg-server-common')

    _install fakeinstall/usr/lib/xorg/protocol.txt
    _install fakeinstall/usr/share/man/man1/Xserver.1

    install -m644 -Dt "${pkgdir}/var/lib/xkb/" xserver/xkb/README.compiled
    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-git() {
    pkgdesc="Xorg X server"
    depends=(
        libepoxy libxfont2 pixman xorg-server-common libunwind dbus libgl 
        xf86-input-libinput nettle libpciaccess libdrm libxshmfence
    ) # FS#52949

    # see xorg-server-*/hw/xfree86/common/xf86Module.h for ABI versions -
    # we provide major numbers that drivers can depend on
    # and /usr/lib/pkgconfig/xorg-server.pc in xorg-server-devel pkg
    provides=(
        'X-ABI-VIDEODRV_VERSION=25.0' 'X-ABI-XINPUT_VERSION=24.1' 
        'X-ABI-EXTENSION_VERSION=10.0' 'x-server' 'xorg-server'
    )
    conflicts=(
        'nvidia-utils<=331.20' 'glamor-egl' 'xf86-video-modesetting' 
        'xorg-server'
    )
    replaces=('glamor-egl' 'xf86-video-modesetting')
    install=xorg-server.install

    _install fakeinstall/usr/bin/{Xorg,cvt,gtf}
    ln -s /usr/bin/Xorg "${pkgdir}/usr/bin/X"
    _install fakeinstall/usr/lib/Xorg{,.wrap}
    _install fakeinstall/usr/lib/xorg/modules/*
    _install fakeinstall/usr/share/X11/xorg.conf.d/10-quirks.conf
    _install fakeinstall/usr/share/man/man1/{Xorg,Xorg.wrap,cvt,gtf}.1
    _install fakeinstall/usr/share/man/man4/{exa,fbdevhw,modesetting}.4
    _install fakeinstall/usr/share/man/man5/{Xwrapper.config,xorg.conf,xorg.conf.d}.5

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -m755 -d "${pkgdir}/etc/X11/xorg.conf.d"

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-xephyr-git() {
    pkgdesc="A nested X server that runs as an X application"
    depends=(
        libxfont2 libgl libepoxy libunwind systemd-libs libxv pixman
        xorg-server-common xcb-util-image xcb-util-renderutil xcb-util-wm 
        xcb-util-keysyms nettle libtirpc
    )
    conflicts=('xorg-server-xephyr')
    provides=('xorg-server-xephyr')

    _install fakeinstall/usr/bin/Xephyr
    _install fakeinstall/usr/share/man/man1/Xephyr.1

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-xvfb-git() {
    pkgdesc="Virtual framebuffer X server"
    depends=(
        libxfont2 libunwind pixman xorg-server-common xorg-xauth libgl nettle
    )
    conflicts=('xorg-server-xvfb')
    provides=('xorg-server-xvfb')

    _install fakeinstall/usr/bin/Xvfb
    _install fakeinstall/usr/share/man/man1/Xvfb.1

    install -m755 "${srcdir}/xvfb-run" "${pkgdir}/usr/bin/"
    install -m644 "${srcdir}/xvfb-run.1" "${pkgdir}/usr/share/man/man1/" # outda

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-xnest-git() {
    pkgdesc="A nested X server that runs as an X application"
    depends=(libxfont2 libxext pixman xorg-server-common nettle libtirpc)
    conflicts=('xorg-server-xnest')
    provides=('xorg-server-xnest')

    _install fakeinstall/usr/bin/Xnest
    _install fakeinstall/usr/share/man/man1/Xnest.1

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-xdmx-git() {
    pkgdesc="Distributed Multihead X Server and utilities"
    depends=(
        libxfont2 libxi libxaw libxrender libdmx libxfixes pixman 
        xorg-server-common nettle
    )
    conflicts=('xorg-server-xdmx')
    provides=('xorg-server-xdmx')

    _install fakeinstall/usr/bin/{Xdmx,dmx*,vdltodmx,xdmxconfig}
    _install fakeinstall/usr/share/man/man1/{Xdmx,dmxtodmx,vdltodmx,xdmxconfig}.1

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-xwayland-git() {
    pkgdesc="run X clients under wayland"
    depends=(
        libxfont2 libepoxy libunwind systemd-libs libgl pixman 
        xorg-server-common nettle libtirpc
    )
    conflicts=('xorg-server-xwayland')
    provides=('xorg-server-xwayland')

    _install fakeinstall/usr/bin/Xwayland

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING
}

package_xorg-server-devel-git() {
    pkgdesc="Development files for the X.Org X server"
    depends=('xorgproto' 'mesa' 'libpciaccess' 'xorg-util-macros')
    conflicts=('xorg-server-devel')
    provides=('xorg-server-devel')

    _install fakeinstall/usr/include/xorg/*
    _install fakeinstall/usr/lib/pkgconfig/xorg-server.pc
    _install fakeinstall/usr/share/aclocal/xorg-server.m4

    # license
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver/COPYING

    # make sure there are no files left to install
    find fakeinstall -depth -print0 | xargs -0 rmdir
}
