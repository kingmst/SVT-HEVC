# SVT-HEVC ffmpeg plugin installation

These instructions build a version of ffmpeg with the SVT-HEVC plugin.  The steps differ depending
on whether using a Linux or Windows build machine.

## For Linux:

1\. Build and install SVT-HEVC

``` bash
git clone https://github.com/OpenVisualCloud/SVT-HEVC
cd SVT-HEVC/Build/linux
./build.sh release
cd Release
make install
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib
export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/usr/local/lib/pkgconfig
```
## For Windows:
0\. Setup environment
``` bash
Install Visual Studio 2017
Install msys2-x86_64 to C:\msys64 following instructions found on http://www.msys2.org/
Run "C:\msys64\mingw64.exe" to open a msys2 mingw-w64 shell.  All subsequent build steps should be done in this shell.
Execute "pacman -S --needed make mingw-w64-x86_64-cmake mingw-w64-x86_64-gcc mingw-w64-x86_64-yasm mingw-w64-x86_64-SDL2 perl diffutils pkg-config git tar" in the console
```
1\. Build and install SVT-HEVC
``` bash
git clone https://github.com/OpenVisualCloud/SVT-HEVC
```
Option 1: use msvc compiler
``` bash
cd SVT-HEVC/Build/windows
./build.bat 2017 release
cmake --install . --config Release
```
Option 2: use mingw-w64 gcc
``` bash
mkdir release
cd release
cmake .. -G"Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=C:/svt-encoders -DBUILD_SHARED_LIBS=off
make -j $(nproc) install
cd ../..
```
The location c:/svt-encoders is used to store a package configuration for the SvtHevcEnc library.
``` bash
export PKG_CONFIG_PATH=/c/svt-encoders/lib/pkgconfig:$PKG_CONFIG_PATH
```
This instruction will verify that the SvtHevcEnc library can be found when building ffmpeg
``` bash
pkg-config --print-errors SvtHevcEnc
export PATH=$PATH:/c/svt-encoders/lib/
```
## For Both:
2\. Install FFmpeg with SVT-HEVC FFmpeg plugin
``` bash
git clone https://github.com/FFmpeg/FFmpeg ffmpeg
cd ffmpeg

# The patch has been validated on n4.2 and n4.3 release tag
git checkout n4.2
or 
git checkout n4.3

git am ../SVT-HEVC/ffmpeg_plugin/0001*.patch
./configure --enable-libsvthevc
make -j $(nproc)
```
3\. Verify that ffmpeg can encode media files using the SVT-HEVC plug-in
``` bash
./ffmpeg -i input.mp4 -c:v libsvt_hevc -rc 1 -b:v 10M -tune 0 -preset 9 -y test.265
./ffmpeg -i input.mp4 -c:v libsvt_hevc -vframes 1000 -y test.mp4
```
