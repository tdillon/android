# android
Try to get continuous deployment working. 

./gradlew assembleRelease

/usr/local/java/jdk1.8.0_92/bin/keytool -genkey -v -keystore /home/travis/keystores/demo.keystore -alias demo -keyalg RSA -keysize 2048 -validity 10000

/usr/local/java/jdk1.8.0_92/bin/jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore /home/travis/keystores/demo.keystore app/build/outputs/apk/app-release-unsigned.apk demo

/usr/local/java/jdk1.8.0_92/bin/jarsigner -verify -verbose -certs app/build/outputs/apk/app-release-unsigned.apk

/home/travis/Android/Sdk/build-tools/23.0.3/zipalign -v 4 app/build/outputs/apk/app-release-unsigned.apk app/build/outputs/apk/app-release.apk