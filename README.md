# fpicker-aflpp-android

![aflplusplus-fpicker](https://user-images.githubusercontent.com/20355405/204588017-a065943c-03de-4340-87a1-620358bdef12.png)

In the past used it to refind Stagefright bug CVE 2020-0411 within hours (several months after being 0 day)

Tested 29/11/2022 on Android 12 x86_64 (think x86) in emulator (docker) on x86_64 host

You can run this on Device or Emulator (x86, x86_64 or arm, aarch64)

With docker there are some space issues, on Android Emulator from Android Studio you can eaisly resize 

# WARNING: You can adjust all the paths other than /data/local/tmp/re.frida.server and /mnt/scratch/libandroid-shmem.so (otherwise you need to modify components)

# INFO Initially used it with Frida 14, but it also works with latest Frida 16. Please use latest Frida version



## Install NDK

make toolchain ie under $HOME/x86_64
```
 ./make_standalone_toolchain.py --arch x86_64 --install-dir ~/x86_64
 
export CC=$HOME/x86_64/bin/x86_64-linux-android31-clang
```

## SHMEM
```
make 

make libandroid-shmem.so
```

## Fpicker (modified)

copy devkit to  libfrida-core-linux.a
```
make fpicker-linux
```
## Socket

Already precompiled (adjust Makefile when need to compile)

## AFL++ for Android AOSP (modified for fpicker)

After AOSP compilation we should copy afl++ to the working directory (root of AOSP)

Now that we are at the right version, we need to extract frida-gum (https://github.com/frida/frida/releases/download/15.1.4/frida-gum-devkit-15.1.4-android-x86_64.tar.xz or arm) to the utils/afl_frida/android/ directory.

```
.
├── arm
│   ├── frida-gum-example.c
│   ├── frida-gum.h
│   └── libfrida-gum.a
├── frida-gum-example.c
├── frida-gum.h
├── libfrida-gum.a
├── README.md
└── x86
    ├── frida-gum-example.c
    ├── frida-gum.h
    └── libfrida-gum.a

2 directories, 10 files
```

libfrida-gum.a and frida-gum.h is the same as in x86 (compiling here for x86_64)

All that’s left for us to do is build AFL++ from the working directory with the following command:
```
mmm afl++
```
## Docker Android 12 Device


## On Pc
```
docker run   --privileged -d -p 6080:6080 -p 5554:5554 -p 5555:5555 -e DEVICE="Samsung Galaxy S6"  budtmo/docker-android-x86-12.0
```
```
adb root
adb disable verity
adb remount
adb reboot
```
Copy

```
adb push libandroid-shmem.so /mnt/scratch

adb push support-sock-x86_64 /data/local/tmp
```
## On device
```
cd /data/local/tmp
chmod a+x support-sock-x86_64
./support-sock-x86_64
```
## On Pc
```
adb push $HOME/fuzzer-fpicker/afl++x86_64/system/bin/afl-fuzz /mnt/scratch
adb push $HOME/fuzzer-fpicker/afl++x86_64/system/lib64 /mnt/scratch


adb push fpicker-x86_64/examples/test /mnt/scratch
adb push fpicker-x86_64/harness/ /mnt/scratch

adb push frida-server-14.2.18-android-x86_64 /mnt/scratch
```

## Start frida
```
chmod a+x frida-server-14.2.18-android-x86_64
./frida-server-14.2.18-android-x86_64
```
cp libandroid-shmem.so /data/local/tmp/re.frida.server/  (Important, keep that paths as they are)

## Start test program
```
test/test
```
## Copy fpicker


## Start fuzzing
```
mkdir in 
echo "AAA" > in/1
mkdir out

chmod a+x afl-fuzz
chmod a+x fpicker

cd /sys/devices/system/cpu
echo performance | tee cpu*/cpufreq/scaling_governor

LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/mnt/scratch/lib64 AFL_SKIP_BIN_CHECK=1 LD_PRELOAD=/mnt/scratch/libandroid-shmem.so AFL_NO_AFFINITY=1  ./afl-fuzz -m none -i in -o out -- /mnt/scratch/fpicker -v --fuzzer-mode afl -e attach -p test -f /mnt/scratch/examples/test/fuzzer-agent.js  --communication=send
```



