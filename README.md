# android
Are you looking for "publish android to google play with travis-ci"?
Here you go.

## Helpful Links
https://docs.travis-ci.com/user/languages/android/
https://developer.android.com/studio/publish/app-signing.html#signing-manually
https://docs.travis-ci.com/user/encrypting-files/
https://github.com/fastlane/fastlane/tree/master/supply

## Process Overview
1. Initialize a Github repo.
2. Create empty Android project using Android Studio in that repo.
3. play store listing
4. Setup Travis-CI to build that repo.
5. Get keystore ready
6. Get TRAVIS-CI ready
7. Setup .travis.yml
8. Setup supply

I'm assuming you can do steps 1-4.  If not, create an issue.

## 5. Keystore
Create a keystore and key.
Only do this once.
The .jks file will hold all the keys you create for all Android apps.
The keystore alias will be the name of the key for an individual app (or you can use one key for everything).
Remember the passwords.
Keep a copy of this file somewhere permanent.
You can use Android Studio or the commandline to do this.

```shell
keytool -genkey -v -keystore /PATH_TO_/MY.jks -alias MY_KEYSTORE_ALIAS -keyalg RSA -keysize 2048 -validity 10000
```

Follow the [instructions](https://docs.travis-ci.com/user/encrypting-files/) to encrypt the keystore and add it to your repo.

```shell
travis encrypt-file /PATH_TO_/MY.jks --add
```

### 6. Get Travis-CI Ready
Add the passwords for your keystore to Travis-CI as encrypted environment variables.
I named them to match the jarsigner command, i.e., storepass and keypass.

### 7. Setup .travis.yml
Your .travis.yml file will need to build, sign, zipalign, and deploy.
$storepass and $keypass are the encrypted environment variables that were setup with step #6.
Substitude for the following:

* MY.jks  (the name of your keystore from step #5)
* PATH_TO_/MY.apk  (as of now, the build command will create the apk here -> app/build/outputs/apk/app-release-unsigned.apk)
* MY_KEYSTORE_ALIAS  (the name of your key alias from step #5)
* PATH_TO_/MY_RELEASE.apk  (the location/name of your release apk)

```yaml
script:
- "./gradlew assembleRelease"
- jarsigner -verbose -sigalg SHA1withRSA -storepass $storepass -keypass $keypass -digestalg SHA1 -keystore MY.jks PATH_TO_/MY.apk MY_KEYSTORE_ALIAS
- zipalign -v 4 PATH_TO_/MY.apk PATH_TO_/MY_RELEASE.apk
```

### 8. Setup supply
[Install](https://github.com/fastlane/fastlane/tree/master/supply#installation) supply.
Follow the [setup](https://github.com/fastlane/fastlane/tree/master/supply#setup) to get your Google Developers Service Account API file.
The file must be encrypted like the keystore was in step #5. TODO these two files have to be tar'd together.
[Init](https://github.com/fastlane/fastlane/tree/master/supply#quick-start) your project.

* PATH_TO_/FOO.json
* MY.PACKAGE.NAME
* PATH_TO_/MY_RELEASE.apk


```shell
supply init -j PATH_TO_/FOO.json -p MY.PACKAGE.NAME
```

After you have initialized your project for supply, you can create publish.

```yaml
supply run -j PATH_TO_/FOO.json -p MY.PACKAGE.NAME -b PATH_TO_/MY_RELEASE.apk
```
