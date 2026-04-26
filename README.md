# Whisper
Whisper ipa build but modified to work on linux

[Wisp protocol](https://github.com/MercuryWorkshop/wisp-protocol) client that exposes the Wisp connection over a TUN device.

## Building on Linux

### Prerequisites
1. Install [xtool](https://xtool.sh/documentation/xtool/installation-linux/)
2. Download [Xcode.xip](https://developer.apple.com/download/all/?q=Xcode) (requires free Apple ID)
3. Install the iOS SDK:
```bash
   xtool sdk install ~/Downloads/Xcode_*.xip
```
4. Install dependencies:
```bash
   sudo apt install clang lld
   rustup target add aarch64-apple-ios
```
5. Create a fake `xcrun`:
```bash
   sudo tee /usr/local/bin/xcrun > /dev/null << 'EOF'
   #!/bin/bash
   SDKROOT="$HOME/.swiftpm/swift-sdks/darwin.artifactbundle/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk"
   case "$*" in
     *"--show-sdk-path"*) echo "$SDKROOT" ;;
     *"--find clang"*) which clang ;;
     *) exec "$@" ;;
   esac
   EOF
   sudo chmod +x /usr/local/bin/xcrun
```

### Building libwhisper.a
```bash
export SDKROOT="$HOME/.swiftpm/swift-sdks/darwin.artifactbundle/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk"
export CC_aarch64_apple_ios="clang --target=aarch64-apple-ios --sysroot=$SDKROOT"
export CXX_aarch64_apple_ios="clang++ --target=aarch64-apple-ios --sysroot=$SDKROOT"
export IPHONEOS_DEPLOYMENT_TARGET="14.0"

cargo build --release --target aarch64-apple-ios --lib
```

The built library will be at `target/aarch64-apple-ios/release/libwhisper.a`.

### Building the iOS app
```bash
# Install Theos
bash -c "$(curl -fsSL https://raw.githubusercontent.com/theos/theos/master/bin/install-theos)"

# Clone whisper-ios and build
git clone https://github.com/MercuryWorkshop/whisper-ios
cd whisper-ios
cp ../whisper/target/aarch64-apple-ios/release/libwhisper.a .
cp ../whisper/target/aarch64-apple-ios/release/libwhisper.a wispvpn/
unset SDKROOT
make package
```

The `.deb` will be in `packages/`. To convert to `.ipa`:
```bash
mkdir -p /tmp/whisper-extract && dpkg-deb -x packages/*.deb /tmp/whisper-extract
cd /tmp/whisper-extract && mkdir Payload
cp -r Applications/Whisper.app Payload/
zip -r ~/Downloads/Whisper.ipa Payload/
```

## License

Whisper is licensed under the [GNU GPL-3.0-or-later license](https://www.gnu.org/licenses/gpl-3.0.html).

## Contributing

Contributions are welcome! Please write tests and make sure they pass before submitting a pull request.
