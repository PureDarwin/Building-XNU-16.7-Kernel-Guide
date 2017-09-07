# **README for Building XNU 16.7 (xnu-3789.70.16)**
For Building XNU 16.7 from start to finish
Requirements macOS 10.12.6 and Xcode 8.3.3

**These must be done in order or it will not build**

Open terminal.app // copy each command in as you go.   

Creating workspace
*************************************
> 
```
mkdir -p ~/Desktop/xnu_build
```
************************************
>
```
cd ~/Desktop/xnu_build
```

#### Curling needed Sources
************************************
>
```
curl -O https://opensource.apple.com/tarballs/dtrace/dtrace-209.50.12.tar.gz && 
curl -O https://opensource.apple.com/tarballs/AvailabilityVersions/AvailabilityVersions-26.50.4.tar.gz && 
curl -O https://opensource.apple.com/tarballs/xnu/xnu-3789.70.16.tar.gz && 
curl -O https://opensource.apple.com/tarballs/libdispatch/libdispatch-703.50.37.tar.gz && 
curl -O https://opensource.apple.com/tarballs/libplatform/libplatform-126.50.8.tar.gz
```

#### Extracting and Removing Tar.gz files
************************************
>
```
for file in *.tar.gz; do tar -zxf $file; done && rm -f *.tar.gz
```

#### Building Dtrace's CTF"s Bianries
************************************
>
```
cd dtrace-209.50.12
```

************************************
>
```
mkdir -p obj sym dst
```

************************************
>
```
xcodebuild install -target ctfconvert -target ctfdump -target ctfmerge ARCHS="x86_64" SRCROOT=$PWD OBJROOT=$PWD/obj SYMROOT=$PWD/sym DSTROOT=$PWD/dst
```

************************************
>
```
sudo ditto $PWD/dst/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain   
```

#### Installing Availability
************************************
>
```
cd ../AvailabilityVersions-26.50.4/ 
```

>
```
mkdir -p dst   
```
>
```
make install SRCROOT=$PWD DSTROOT=$PWD/dst
```

************************************
>
```
sudo ditto $PWD/dst/usr/local `xcrun -sdk macosx -show-sdk-path`/usr/local
```
#### Installing Headers to build libfirehose_kernel.a for XNU
************************************
>
```
cd ../xnu-3789.70.16/
```

>
```
mkdir -p BUILD.hdrs/obj BUILD.hdrs/sym BUILD.hdrs/dst
```

>
```
make installhdrs SDKROOT=macosx ARCH_CONFIGS=X86_64 SRCROOT=$PWD OBJROOT=$PWD/BUILD.hdrs/obj SYMROOT=$PWD/BUILD.hdrs/sym DSTROOT=$PWD/BUILD.hdrs/dst
```

>
```
sudo xcodebuild installhdrs -project libsyscall/Libsyscall.xcodeproj -sdk macosx ARCHS='x86_64 i386' SRCROOT=$PWD/libsyscall OBJROOT=$PWD/BUILD.hdrs/obj SYMROOT=$PWD/BUILD.hdrs/sym DSTROOT=$PWD/BUILD.hdrs/dst   
```
>
```
sudo ditto BUILD.hdrs/dst `xcrun -sdk macosx -show-sdk-path`  
```

### Copying libplatform source needed for libfirehose_kernel.a
************************************
> 
```
cd ../libplatform-126.50.8
```
************************************
> 
```
sudo ditto $PWD/include `xcrun -sdk macosx -show-sdk-path`/usr/local/include   
```

#### Building libfirehose_kernel.a
************************************
> 
```
cd ../libdispatch-703.50.37
```

> 
```
mkdir -p BUILD.hdrs/obj BUILD.hdrs/sym BUILD.hdrs/dst
```
> 
```
sudo xcodebuild install -project libdispatch.xcodeproj -target libfirehose_kernel -sdk macosx ARCHS='x86_64 i386' SRCROOT=$PWD OBJROOT=$PWD/obj SYMROOT=$PWD/sym DSTROOT=$PWD/dst
```
>
```
sudo ditto $PWD/dst/usr/local `xcrun -sdk macosx -show-sdk-path`/usr/local
```
#### Building XNU
************************************
> 
```
cd ../xnu-3789.70.16/
```
************************************
> 
```
make SDKROOT=macosx ARCH_CONFIGS=X86_64 KERNEL_CONFIGS=RELEASE
```
