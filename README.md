# nextcloud desktop client
:computer: theme and build instructions for the nextcloud dekstop client

Based on https://github.com/owncloud/client/blob/master/doc/building.rst

## Getting repository ready

Run:
```bash
git submodule update --init
cd client
git submodule update --init
cd ...
```

## Building on Linux

Run:

```bash
mkdir build-linux
cd build-linux
cmake -D OEM_THEME_DIR=`pwd`/../nextcloudtheme ../client
make
make install
```

## Building on OSX

*Attention:* When building make sure to use an old Core 2 Duo build machine running OS X 10.10. Otherwise the resulting binary won't work properly for users of an older device.

### Install dependencies

1. Install [HomeBrew](http://brew.sh/)
2. `brew tap owncloud/owncloud`
3. `brew install qtkeychain cmake`
4. `brew install openssl`
5. Download the newest Sparkle from https://sparkle-project.org/
6. `mv Sparkle.framework ~/Library/Frameworks/`
7. Install XCode 6.4 from http://adcdownload.apple.com/Developer_Tools/Xcode_6.4/Xcode_6.4.dmg
8. sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
9. Generate Sparkle keys: `./bin/generate_keys`. Keep those, if you loose it you won't be able to deploy updates anymore.
10. Store the keys in `osx/`. Make sure to not make the `dsa_priv.pem` publicly available.
11. Install http://s.sudre.free.fr/Software/Packages/about.html

### Install older XCode SDK

TODO: Check if can be removed

First of all we need to install the OSX 10.8 SDK, this can be done using xcodelegacy. To be able to do that you need to have an Apple Developer Membership.

1. `cd /tmp`
2. Download https://developer.apple.com/devcenter/download.action?path=/Developer_Tools/xcode_5.1.1/xcode_5.1.1.dmg and put into /tmp/


```bash
wget https://raw.githubusercontent.com/devernay/xcodelegacy/master/XcodeLegacy.sh
sudo bash XcodeLegacy.sh -osx108 buildpackages
sudo bash XcodeLegacy.sh -osx108 install
```

### Compile Qt

Because the desktop client comes with a lot of custom patches you have to download the Qt 5.4.0 source and then apply all of them.

In addition to below patches you also need to apply https://codereview.qt-project.org/#/c/121545/

```bash
cd /tmp/
wget http://download.qt.io/official_releases/qt/5.4/5.4.0/single/qt-everywhere-opensource-src-5.4.0.tar.gz
tar -xf qt-everywhere-opensource-src-5.4.0.tar.gz
cd /tmp/qt-everywhere-opensource-src-5.4.0/qtbase
git apply /Users/lukasreschke/client/admin/qt/patches/0001-Fix-crash-on-Mac-OS-if-PAC-URL-contains-non-URL-lega.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0002-Fix-possible-crash-when-passing-an-invalid-PAC-URL.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0003-Fix-crash-if-PAC-script-retrieval-returns-a-null-CFD.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0004-Cocoa-Fix-systray-SVG-icons.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0005-OSX-Fix-disapearing-tray-icon.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0006-Fix-force-debug-info-with-macx-clang_NOUPSTREAM.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0007-QNAM-Fix-upload-corruptions-when-server-closes-conne.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0007-X-Network-Fix-up-previous-corruption-patch.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0008-QNAM-Fix-reply-deadlocks-on-server-closing-connectio.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0009-QNAM-Assign-proper-channel-before-sslErrors-emission.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0010-Don-t-let-closed-http-sockets-pass-as-valid-connecti.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0011-Make-sure-to-report-correct-NetworkAccessibility.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0012-Make-sure-networkAccessibilityChanged-is-emitted.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0013-Make-UnknownAccessibility-not-block-requests.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0015-Remove-legacy-platform-code-in-QSslSocket-for-OS-X-1.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0016-QSslSocket-evaluate-CAs-in-all-keychain-categories.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0017-Win32-Re-init-system-proxy-if-internet-settings-chan.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0018-Windows-Do-not-crash-if-SSL-context-is-gone-after-ro.patch
git apply /Users/lukasreschke/client/admin/qt/patches/0019-Ensure-system-tray-icon-is-prepared-even-when-menu-bar.patch
cd ..
./configure -sdk macosx10.9
make -j7
sudo make -j1 install
```

### Build the client

```bash
sh osx/build.sh
```

## Building on Windows

### Building the docker image

The docker image contains the toolchain to build the windows binary.
Build it:

```bash
docker build -t nextcloud-client-win32:<version> client/admin/win/docker/
```

### Building the binary

```bash
docker run -v "$PWD:/home/user/" nextcloud-client-win32:2.2.2 /home/user/win/build.sh $(id -u)
```
