# HOWTO install Android SDK using command line tools

Find `Command line tools only` section on [Android Studio downloads
page](https://developer.android.com/studio#downloads) and download appropriate
package.

Unzip package and go to the bin dir:
```
unzip commandlinetools-linux-*_latest.zip
cd cmdline-tools/bin
```

Export future Android SDK root variable:
```
export ANDROID_SDK=/opt/android
```

Now you can get full list of packages using:
```
./sdkmanager --sdk_root=${ANDROID_SDK} --list
```

Install required packages, at least you need:
```
./sdkmanager --sdk_root=${ANDROID_SDK} \
	'cmdline-tools;latest' \
	'platform-tools' \
	'platforms;android-30' \
	'build-tools;30.0.2'
```

To make sdk available for all users you need to add `android.sh` to the
`/etc/profile.d`:
```
# Support android SDK
export ANDROID_SDK=/opt/android
export PATH=${PATH}:${ANDROID_SDK}/platform-tools:${ANDROID_SDK}/tools:${ANDROID_SDK}/tools/bin
```

Create new emulator instance from command line:
```
avdmanager create avd -n android-23-arm -k 'system-images;android-23;default;armeabi-v7a'
```

Launch emulator from command line:
```
cd ${ANDROID_SDK}/tools
./emulator @android-23-arm
```
