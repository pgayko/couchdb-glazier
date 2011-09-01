#Glazier - automating building CouchDB on Windows #############################

Glazier is a set of scripts designed to help automate as much as practicable the
build of CouchDB on Windows, from XP/2003 to Windows 7 or Server 2008. It
assumes you're starting from a vanilla state.

# Using Glazier  ##############################################################

Glazier requires 6 things to run successfully

1. you are logged in as a local user with admin permissions
2. Windows XP, 2003, Vista, Windows 7 or 2008, either 32 or 64-bit platforms
3. internet connectivity
4. approx 12GiB of disk space during fetch & build stages
5. download and unzipped [glazier latest zip](http://github.com/dch/glazier/zipball/master)
6. an optional environment variable, `%RELAX%`, set to where you wish to have CouchDB
 and Erlang built within. If none is selected, `c:\relax` will be used.

## Current State ##############################################################

* The steps below, manually or automated, should produce a working CouchDB build
    & self-installing .exe from Erlang/OTP R14B01, and CouchDB 1.0.1 onwards.
* The build environment should be fully functional on both 32,64, desktop and
    server versions of windows from XP/2003 onwards
* Fetching dependent binaries is described and automated
* Installation of compilers and development environment is described and automated
* Downloads are not small -
[get_bits.cmd](https://github.com/dch/glazier/blob/master/bin/get_bits.cmd)
    retrieves approx 7GiB of DVD ISOs for Microsoft's Visual Studio 2008
    compiler, related SDKs, the smaller cygwin and mozilla build frameworks,
    source and misc tools
* Glazier tries to be self-contained so that it is both repeatable and also
    easy to clean up.
* Compilation stages are not fully automated but are all now command-line driven
* Visual Studio 2010 is not supported yet by all source code so use Visual
    Studio Express 2008 instead

# Running Automatically #######################################################

* run `%GLAZIER%\bin\glaze.cmd` to fetch & cache the bits, install compiilers
* you will need to select the cygwin modules individually
* close any open command prompts or shells
* run `%GLAZIER%\bin\relax.cmd` to start a valid build environment
* select the erlang version you wish to build from
* the first time around you will need to unpack your erlang and couchdb tarballs

#TODO needs work

        cd /relax
        CYGWIN="nontsec nodosfilewarning"
        CL=/D_BIND_TO_CURRENT_VCLIBS_VERSION=1
        tar xzf /relax/bits/apache-couchdb-1.1.0.tar.gz &
        tar xzf /relax/bits/curl-7.21.7.tar.gz &
        tar xzf /relax/bits/otp_src_R14B03.tar.gz &
        cd /relax/otp_src_R14B03 && tar xzf /relax/bits/tcltk85_win32_bin.tar.gz &

* then run the following 4 scripts in order

        erl_config.sh
        erl_build.sh
        couchdb_config.sh
        couchdb_build.sh

* each of these scripts leaves logfiles in the root folder. If you have issues
    during compilation phase, load these onto <http://friendpaste.com/>
    before emailing your issues to the mailing list

# Installing the Build Environment ############################################

* Building Erlang & CouchDB on Windows requires a custom build environment,
    which is very sensitive to path order amongst the three different
    compilers used to build wxWidgets, erlang, javascript, and couchdb
* Each component is built via standard Makefiles in the Cygwin unix/posix
    emulation layer, and then handed over to the appropriate compiler.
* This is further complicated by different install locations on 32 vs 64 bit
    windows versions, and which Microsoft C compiler and Windows SDKs
    installed.

## Cygwin #####################################################################

The full Cygwin install comprises several GiB of data. Run [cygwin]'s setup.exe
using defaults, optionally installing all components if you have the
bandwidth, or alternatively with the following additional modules at a
minimum:

* devel: ALL
* editors: vim
* utils: file

After install, set up a link to where you plan to install related binaries,
build erlang, and couchdb. I am using `C:\relax` so:

        setx RELAX c:\relax
        mkdir %RELAX%
        junction.exe c:\cygwin\relax %RELAX%

## Mozilla Build ##############################################################

The mozilla build toolchain is needed solely for building a javascript engine.

* Download it from [mozbuild] and install per defaults

## Microsoft Visual C++ #######################################################

* Erlang and CouchDB can be built using the free VS2008 Express C++ edition
    from [MSDN](http://msdn.microsoft.com/en-gb/vstudio/)
* install Visual C++ 9 only, to the default locations, using the DVD ISO
    [msvc++] excluding optional MSSSQL & Silverlight, or alternatively the
    web installer [msvc++webstart]

## Windows 7 SDK ##############################################################

* The windows 7 SDK is required, as the free VS2008 install is missing the
    message compiler. Download one of the following version per your
    requirements & install
* [win7sdk_32bit]
* [win7sdk_64bit]

# Supporting Tools ############################################################

Both CouchDB and Erlang have dependencies on other opensource tools.

## OpenSSL ####################################################################

* use the 32-bit version even if you are using a 64-bit OS
* download [openssl_bits] and unzip to `c:\openssl`

## Innosoft Installer #########################################################

* download the installer [inno_bits] and install to `c:\relax\inno5`

## NSIS Installer #############################################################

* download the installer [nsis_bits] and install to `c:\relax\nsis`

## Microsoft Visual C++ runtime ###############################################

* download the runtime installer [vcredist] and copy to `c:\relax\` - note this
    is the same as the one provided with VS2008 SP1 - this is *critical*
    to ensure that your compiled erlang, spidermonkey, couchdb all require
    the same runtime that is bundled with your distribution. The MD5 should
    be 2c79302d42817d54bb621216a6d4dc7f.

## set up hard links ##########################################################

* to keep our paths clean later, and largely independent of compiler
    installations if you have pre-existing ones, start a new cmd.exe
    prompt with a fresh environment
* this should have both VS90ComnTools and ProgramFiles environment vars
    defined from the previous install of Visual Studio
* setup the following hard links (junction points), using either the included
    mklink tool (Windows 7 and later), or SysInternal's
    [junction](http://live.sysinternals.com/junction.exe)

        junction c:\relax\openssl c:\openssl
        junction c:\relax\vs90 "%VS90COMNTOOLS%\..\.."
        junction c:\relax\SDKs "%programfiles%\Microsoft SDKs\Windows"

or using mklink.exe

        mklink /j c:\relax\openssl c:\openssl
        [etc...]

# Building pre-requisites for Erlang ##########################################

## wxWidgets ##################################################################

* two components are used for building Erlang's graphical shell, `werl.exe`
    on windows
* download [wxwidgets_bits] from [WxWidgets website](http://wxwidgets.org/)
    & unzip using cygwin into /relax/
* the Erlang build expects to see wxWidgets in /opt/local/pgm/wxWidgets-2.8.11
    by default

        mkdir c:\cygwin\opt\local\pgm
        junction c:\cygwin\opt\local\pgm\wxWidgets-2.8.11 c:\relax\wxMSW-2.8.11

* A number of manual changes are required to get WxWidgets to build cleanly.
    These can be done manually or simply apply the overlay from [wxoverlay] &
    skip through to the build instructions lower down.

* Using a suitable editor (vi in the cygwin suite, or install
    [notepadplus_bits] for windows users) and
* Edit `c:\relax\wxMSW-2.8.11\include\wx\msw\setup.h` to enable
    `wxUSE_GLCANVAS, wxUSE_POSTSCRIPT` and `wxUSE_GRAPHICS_CONTEXT`

### wx.dsw ####################################################################

* open VSC++ & the project  `%RELAX%\wxMSW-2.8.11\build\msw\wx.dsw`,
    accepting the automatic conversion to the newer VC++ format and save
    as `\relax\wxMSW-2.8.11\build\msw\wx.sln`
* right-click on the project, and set up the dependencies for wx.dsw to
    achieve the below build order:
`jpeg, png, tiff, zlib, regex, expat, base, net, odbc, core,
 gl, html, media, qa, adv, dbgrid, xrc, aui, richtext, xml`
* Launch a new prompt from somewhere like Start -> Programs -> Microsoft
    Visual C++ -> Visual Studio Tools -> VS2008 Cmd Prompt
* Then build all unicode release (and unicode debug) packages:

        pushd %RELAX%\wxMSW*\build\msw
        set CL=/D_BIND_TO_CURRENT_VCLIBS_VERSION=1
        vcbuild /useenv  /platform:Win32 /M4 wx.sln "Unicode Release|Win32"
        vcbuild /useenv  /platform:Win32 /M4 wx.sln "Unicode Debug|Win32"
        popd

### stc.dsw ###################################################################

* open VSC++ & convert `%RELAX%\wxMSW-2.8.11\contrib\build\stc\stc.dsw`
    to `%RELAX%\wxMSW-2.8.11\contrib\build\stc\stc.sln`

        pushd %RELAX%\wxMSW*\contrib\build\stc
        set CL=/D_BIND_TO_CURRENT_VCLIBS_VERSION=1
        vcbuild /useenv /platform:Win32 /M4 stc.sln "Unicode Release|Win32"
        vcbuild /useenv /platform:Win32 /M4 stc.sln "Unicode Debug|Win32"
        popd


# Building Erlang #############################################################

* after installing VC++ 2008 Express, and most other Visual Studio solutions,
    `call "%vs90comntools%\..\..\vc\vcvarsall.bat" x86` will automatically
    set up our 32-bit build environment correctly, independently if
    you have installed on 32 or 64bit windows, with the exception of the
    Windows v7.0 SDK.

* in a cmd.exe shell

        junction.exe %RELAX%\bin %GLAZIER%\bin
        junction.exe %RELAX%\bits %GLAZIER%\bits
        mkdir %RELAX%\release

* in a cygwin shell, using these new junction points:

        cd /relax && tar xzf /relax/bits/otp_src_R14B03.tar.gz &

* then run from explorer, `%GLAZIER%\bin\relax.cmd`

## Tk/Tcl #####################################################################

* optional components - used for debugger and java interfaces

        cd $ERL_TOP && tar xvzf /relax/bits/tcltk85_win32_bin.tar.gz

* or skip the whole damn lot this way

        echo "skipping gs" > lib/gs/SKIP
        echo "skipping jinterface" > lib/jinterface/SKIP

* check that `which cl; which link; which mc` return the MS ones, if not then
 sort them out manually. Refer to
 [relax.cmd](http://github.com/dch/glazier/bin/relax.cmd) and
 [relax.sh](http://github.com/dch/glazier/bin/relax.sh)

* Build Erlang using `/relax/glazier/bin/erl_config.sh`
  and `/relax/glazier/bin/erl_build.sh`, or manually as follows

        eval `./otp_build env_win32`
        ./otp_build autoconf
        ./otp_build configure
        ./otp_build boot -a
        ./otp_build release -a
        ./otp_build installer_win32
        # we need to set up erlang to run from this new source build to build CouchDB
        ./release/win32/Install.exe -s

* More details are at [erlang INSTALL-Win32.md on github](http://github.com/erlang/otp/blob/dev/INSTALL-WIN32.md)

* or using the relax tools:

        start %glazier%\bin\relax.cmd
        :: [select an erlang build]
        erl_config.sh; erl_build.sh


# CouchDB #####################################################################

CouchDB has been built & tested against the following components successfully

* Erlang OTP R14B03 including source
* ICU 4.4.2
* OpenSSL 1.0.0d
* Mozilla SpiderMonkey 1.8.5 or Firefox 4.0 release
* libcurl 7.21.7

## Javascript #################################################################

The Javascript engine used by CouchDB is Mozilla Spidermonkey. As there is no
formal release for it, you can build from anywhere on trunk. The 1.8.5 source
below is also used on the Mac OS X homebrew build of CouchDB.

* to build and install from SpiderMonkey get [spidermonkey_bits]
* run `c:\mozilla-build\start-msvc9.bat` even if you are on a 64-bit platform.

        cd $RELAX
        tar xzf bits/57a6ad20eae9.tar.gz
        cd ./tracemonkey-57a6ad20eae9/js/src
        autoconf-2.13
        export CXXFLAGS='-D_BIND_TO_CURRENT_VCLIBS_VERSION=1'
        ./configure --enable-static --enable-shared-js
        make

## Inno Installer #############################################################

* Download from [inno_bits]
* Install to c:\relax\inno5 & ensure its in the path
* Install ispack-5.4.2-unicode.exe, optionally including additional components

## OpenSSL ####################################################################

* already installed into `C:/OpenSSL/` no further steps required

## LibCURL ####################################################################

* Extract from cygwin shell if not already done

::        cd /relax && tar xf /relax/bits/curl-7*

* run from a Visual Studio command shell:

        pushd %RELAX%\curl-7*
        set OPENSSL_PATH=%relax%\openssl
        set USE_SSLEAY=1
        set USE_OPENSSL=1
        set INCLUDE=%INCLUDE%;%OPENSSL_PATH%\include;%OPENSSL_PATH%\include\openssl;
        set LIBPATH=%LIBPATH%;%OPENSSL_PATH%\lib;
        set LIB=%LIB%;%OPENSSL_PATH%\lib;

        set CL=/D_BIND_TO_CURRENT_VCLIBS_VERSION=1
        vcbuild /useenv /upgrade /platform:Win32 lib\libcurl.vcproj
        vcbuild /useenv /platform:Win32 lib\libcurl.vcproj "Release|Win32"
        xcopy lib\Release\libcurl.lib lib\ /y
        popd

## ICU ########################################################################

* Download and unzip the compiled libraries from [icu_bits_curr]

        cd %RELAX%\
        7z x bits\icu*.zip

## Make & Build ###############################################################

* The generic configure script looks like this:

        ./configure \
        --with-js-include=/cygdrive/c/path_to_spidermonkey/dist/include \
        --with-js-lib=/cygdrive/c/path_to_spidermonkey/dist/lib \
        --with-win32-icu-binaries=/cygdrive/c/path_to_icu_binaries_root \
        --with-erlang=$ERL_TOP/release/win32/usr/include \
        --with-win32-curl=/cygdrive/c/path/to/curl/root/directory \
        --with-openssl-bin-dir=/cygdrive/c/path/to/openssl/bin \
        --with-msvc-redist-dir=/cygdrive/c/dir/with/vcredist_platform_executable \
        --prefix=$ERL_TOP/release/win32

## using spidermonkey 1.8.x   ###################################################

* This is the recommended config if you have used the above steps:

        ./configure \
        --prefix=$ERL_TOP/release/win32 \
        --with-erlang=$ERL_TOP/release/win32/usr/include \
        --with-win32-icu-binaries=/relax/icu \
        --with-win32-curl=/relax/curl-7.21.5 \
        --with-openssl-bin-dir=/relax/openssl/bin \
        --with-msvc-redist-dir=/relax \
        --with-js-lib=/relax/57a6ad20eae9/js/src/dist/lib \
        --with-js-include=/relax/57a6ad20eae9/js/src/dist/include \
        2>&1 | tee $COUCH_TOP/build_configure.txt


********************************************************************************
# Appendices
********************************************************************************

## Licences

* the core tools & scripts used in glazier are released or included under the
    same Apache Licence used for CouchDB
* curl and the included openssl libraries are the only ones built into glazier
* the silent installation of each component assumes your implicit acceptance~
* curl <http://curl.haxx.se/docs/copyright.html>
* openssl <http://www.openssl.org/source/license.html>

## Download URLs - visible only in raw text view

[7zip_bits]:		http://downloads.sourceforge.net/sevenzip/7z465.exe
[bitvise_sshd_bits]:	http://dl.bitvise.com/WinSSHD5-Inst.exe
[curl]:			http://curl.haxx.se/download.html
[cygwin]:		http://www.cygwin.com/setup.exe
[DEP]:			http://support.microsoft.com/kb/875352
[erlang_R14B01]:	http://www.erlang.org/download/otp_src_R14B01.tar.gz
[erlang_R14B03]:	http://www.erlang.org/download/otp_src_R14B03.tar.gz
[icu_bits_curr]:	http://download.icu-project.org/files/icu4c/4.2/icu4c-4_2-Win32-msvc9.zip
[inno_bits]:		http://www.jrsoftware.org/download.php/is-unicode.exe
[inno_help]:		http://www.jrsoftware.org/ishelp/
[libcurl_bits]:		http://curl.haxx.se/download/libcurl-7.19.3-win32-ssl-msvc.zip
[libcurl_src]:		http://curl.haxx.se/download/curl-7.21.7.tar.gz
[msvc++]:		http://download.microsoft.com/download/E/8/E/E8EEB394-7F42-4963-A2D8-29559B738298/VS2008ExpressWithSP1ENUX1504728.iso
[mozbuild]:		http://ftp.mozilla.org/pub/mozilla.org/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe
[notepadplus_bits]:	http://sourceforge.net/projects/notepad-plus/files/notepad%2B%2B%20releases%20binary/npp%205.8.7%20bin/npp.5.8.7.Installer.exe/download
[nsis_bits]:		http://download.sourceforge.net/project/nsis/NSIS%202/2.46/nsis-2.46-setup.exe
[openssl_bits]:		http://cloud.github.com/downloads/wincouch/openssl/openssl-1.0.0d.zip
[ramdisk]:		http://www.ltr-data.se/files/imdiskinst.exe
[spidermonkey_bits]:	http://hg.mozilla.org/tracemonkey/archive/57a6ad20eae9.tar.gz
[SEHOP]:		http://support.microsoft.com/kb/956607
[vcredist]:
# NB this is the same version as supplied with VS2008sp1 - EXEs and DLLs built against older vcredists can use the newer one successfully
http://download.microsoft.com/download/d/d/9/dd9a82d0-52ef-40db-8dab-795376989c03/vcredist_x86.exe
[win7sdk_32bit]:	http://download.microsoft.com/download/2/E/9/2E911956-F90F-4BFB-8231-E292A7B6F287/GRMSDK_EN_DVD.iso
[win7sdk_64bit]:	http://download.microsoft.com/download/2/E/9/2E911956-F90F-4BFB-8231-E292A7B6F287/GRMSDKX_EN_DVD.iso
[wxwidgets_bits]:	http://sourceforge.net/projects/wxwindows/files/2.8.11/wxMSW-2.8.11.zip
[wxoverlay]:            https://github.com/dch/glazier/raw/master/bits/wxMSW-2.8.11_erlang_overlay.zip

## notes for next release

## zlib from vs2008 prompt please
- get http://zlib.net/zlib-1.2.5.tar.gz
7z x %relax%\bits\zlib-1.2.5.tar.gz -o%relax%\bits\
7z x %relax%\bits\zlib-1.2.5.tar -o%relax%
pushd %relax%\zlib*\contrib\masmx86\
bld_ml32.bat
cd ..\..
vcbuild /rebuild contrib\vstudio\vc9\zlibvc.sln "Release|Win32"
xcopy %relax%\zlib-1.2.5\contrib\vstudio\vc9\x86\ZlibStatRelease\zlibstat.lib %relax%\libraries\ /y
xcopy %relax%\zlib-1.2.5\contrib\vstudio\vc9\x86\ZlibDllRelease\zlibwapi.lib %relax%\libraries\ /y
popd

# bugger that
http://zlib.net/zlib125-dll.zip & unzip

// TODO fix curl and zlib build scripts

compile OpenSSL from scratch

7z x %glazier%\bits\nasm-2.09.07-win32.zip -o%relax%\
move %relax%\nasm* nasm
7z x %glazier%\bits\strawberry-perl-5.12.2.0-portable.zip -o%relax%\strawberry
7z x %glazier%\bits\openssl-1.0.0d.tar.gz -o%relax%\bits\

[open a VC++ shell]
path=%path%;%relax%\nasm;%relax%\strawberry\perl\bin;
pushd %relax%\openssl-*
perl Configure VC-WIN32 --prefix=c:\openssl --
ms\do_nasm
nmake -f ms\ntdll.mak

nmake -f ms\ntdll.mak install

- get http://www.nasm.us/pub/nasm/releasebuilds/2.09.07/win32/nasm-2.09.07-win32.zip
- unzip to $RELAX/nasm & add to path
- get http://strawberryperl.com/download/5.12.2.0/strawberry-perl-5.12.2.0-portable.zip
- unzip to $RELAX/perl & add to path
- download OpenSSL http://www.openssl.org/source/openssl-1.0.0d.tar.gz
- unpack
- do INSTALL.W32 stuff
- install to /openssl

compile ICU from scratch?
no thanks..

 aria2c.exe --force-sequential=false --max-connection-per-server=4  --check-certificate=false --auto-file-renaming=false bits.txt --max-concurrent-downloads=5 --dir=%GLAZIER%/bits --save-session=%GLAZIER%/a2session.txt
 
 
# AWS
 
Must use 2008R2 64-bit edition ami-ee926087
m1.xlarge spot instances are a good deal
create a 30GiB volume from relaxing snapshot
this will provide backing store for the ram disk

 
