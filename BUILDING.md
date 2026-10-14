## Installing libraries on Linux/MacOS
1. Open a terminal
2. Install pip if not already installed
```sh
python3 -m ensurepip
```
3. Clone the repo and `cd` into it
```sh
git clone https://github.com/snesrev/zelda3
cd zelda3
```
4. Install requirements using pip
```sh
python3 -m pip install -r requirements.txt
```
5. Install SDL2 (only needed for the Makefile build; CMake uses the bundled SDL2)
* Ubuntu/Debian `sudo apt install libsdl2-dev`
* Fedora Linux `sudo dnf install SDL2-devel`
* Arch Linux `sudo pacman -S sdl2`
* macOS: `brew install sdl2` (you can get homebrew [here](https://brew.sh/))

## Compiling on Linux/MacOS
1. Place your US ROM file named `zelda3.sfc` in `zelda3`
2. Compile
```sh
make
```
<details>
<summary>
Advanced make usage ...
</summary>

```sh
make -j$(nproc) # run on all core
make clean all  # clear gen+obj and rebuild
CC=clang make   # specify compiler
```
</details>

## Compiling with CMake

Install CMake 3.16 or newer and a C compiler. This build compiles and statically
links SDL2 from `third_party/SDL`; no system SDL2 installation is needed.

From the repository root:
```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release --parallel
```

Asset extraction is separate from the CMake build. If `zelda3_assets.dat` is
missing, install the Python requirements above, place your US ROM at
`zelda3.sfc`, and run:
```sh
python3 assets/restool.py --extract-from-rom
```

Run `./build/zelda3` from the repository root on Linux/macOS. With a
multi-configuration generator such as Visual Studio, run
`build/Release/zelda3.exe` instead. Keep `zelda3.ini` and `zelda3_assets.dat`
in the working directory.

### iOS

On macOS with Xcode and the iOS SDK installed, generate `zelda3_assets.dat`
as described above, then configure in a separate build directory:
```sh
cmake -S . -B build-ios -G Xcode -DZELDA3_IOS=ON \
  -DCMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM=YOUR_TEAM_ID
```

Open `build-ios/zelda3.xcodeproj`, select the `zelda3` target, choose your
signing team and a unique bundle identifier, then build/run on your device.
Replace `YOUR_TEAM_ID` with your Apple development team ID. To compile
without signing instead, omit the team argument and configure with
`-DCMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_ALLOWED=NO`; such a build cannot be
installed on a physical device until signed.
The default minimum iOS version is 15.0; override it with
`-DCMAKE_OSX_DEPLOYMENT_TARGET=<version>` when configuring.
The app bundles `zelda3.ini` and `zelda3_assets.dat`; saves go in the app's
writable preferences directory. No ROM is bundled. Use a game controller;
this option does not add touch controls.

For an unsigned simulator build:
```sh
cmake -S . -B build-ios-simulator -G Xcode -DZELDA3_IOS=ON \
  -DCMAKE_OSX_SYSROOT=iphonesimulator
cmake --build build-ios-simulator --config Release --target zelda3 -- \
  CODE_SIGNING_ALLOWED=NO
```

Use a fresh build directory when switching between desktop, device, and
simulator targets. Desktop builds remain unchanged when `ZELDA3_IOS` is off.

## Nintendo Switch

You need [DevKitPro](https://devkitpro.org/wiki/Getting_Started) and [Atmosphere](https://github.com/Atmosphere-NX/Atmosphere) installed.

```sh
(dkp-)pacman -S git switch-dev switch-sdl2 switch-tools
cd platform/switch
make # Add -j$(nproc) to build using all cores ( Optional )
# You can test the build directly onto the switch ( Optional )
nxlink -s zelda3.nro
```

## More Compilation Help

Look at the wiki at https://github.com/snesrev/zelda3/wiki for more help.

The ROM needs to be named `zelda3.sfc` and has to be from the US region with this exact SHA256 hash
`66871d66be19ad2c34c927d6b14cd8eb6fc3181965b6e517cb361f7316009cfb`

In case you're planning to move the executable to a different location, please include the file `zelda3_assets.dat`.