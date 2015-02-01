---
layout: post
title: "Release an Android Library on Maven Central"
quote: "A lot more hassle than it should be: now explained"
image: /media/25-01-2015-Release-An-Android-Library-On-Maven-Central/cover.jpg
video: false
comments: true
categories:
- Android
- Tutorial
---
I was going to put "the easy way", but well the reality is that there is no easy way.
Hopefully this post can make the entire experience a bit less painful for you.

You will need the following ingredients

1. An Android library
2. [An account on sonatype](#create-a-sonatype-account)
3. [A gpg key](#create-a-gpg-key)
4. [A gradle build script](#get-the-gradle-build-script)
5. [Lot's of gradle configuration](#configure-gradle)
6. [Perform the release steps on sonatype](#release-the-staged-artifact)


I'm going to assume that you have a library in a releasable state and go straight to step 2


## Create a sonatype account
The first thing you need to do is create an account on <https://issues.sonatype.org>. This involves the standard fare of picking a username and a password.
Once that is done, you can [create an issue](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134) to provision your account and obtain a groupId.

On any other website you would be done now, but sonatype verifies your account manually and this takes some time. Don't expect to release your library the day you create your account.

Make sure you read and follow the guidlines for the groupId, they are strictly enforced. If it is a reversed fully qualified domain name of a domain you own (eg: com.wdullaer for me) or where they
can reasonably expect your project to live under your control (eg: com.github.wdullaer), you should be fine.

## Create a gpg key
On Linux the gpg or gpg2 binaries should be installed by default. If not use your systems package manager to get them.
I used gpg instead gpg2, because the latter version kept complaining about gnome-keyring, but they should functionally be the same.

1. Run `gpg --gen-key`  
The default values for all the parameters should be fine.
Fill out your email address, name and optional comment (nickname).

2. If gpg complains about insufficient memory, run the `find / > /dev/null` in a separate shell **while** gpg is running

3. Once the key is created, you'll see something like.

    ```
    gpg: key 1332E476 marked as ultimately trusted
    public and secret key created and signed.

    gpg: checking the trustdb
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   4  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 4u
    pub   2048R/1332E476 2014-12-16
    Key fingerprint = 4709 0122 F4F9 466A 9E60  CCAC B0E4 98EA 1332 E476
    uid                  Wouter Dullaert <wouter.dullaert@gmail.com>
    sub   2048R/DFEF78B0 2014-12-16
    ```
  Here you should get your key-ID, which in this example is: 1332E476

4. Upload this key to a keyserver (any will do, they sync among themselves)

    ```bash
    gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 1332E476
    ```

5. Dump both the public and private keys to file and back them up somewhere safe

    ```bash
    gpg -ao gpg_public.key --export 1332E476
    gpg -ao gpg_private.key --export-secret-keys 1332E476
    ```

More information can be found here: <http://central.sonatype.org/pages/working-with-pgp-signatures.html>


## Get the gradle build script
This is in fact the easiest step. Maven Central puts a lot of quality requirements on the artifacts you can upload: pom.xml files, javadocs, etc.

However, someone already went through the trouble of creating a proper Maven
release build script at <https://raw.githubusercontent.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle>
This script takes care of preparing your artifact in a Maven Central compatible fashion and uploads it to your staging repository.

You can include this one directly into the root of your `build.gradle` of your library.

```javascript
apply from: 'https://raw.github.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle'
```

Or if you're the paranoid sort: you can add a hardcopy to your project and reference that.

## Configure gradle
You will need to create a total of 3 `gradle.properties` files to configure the build script you added in the previous step:

* 1 at the module level (`/MyAwesomeLibrary/library/gradle.properties`)

    ```ini
    POM_NAME=SwipeActionAdapter
    POM_ARTIFACT_ID=swipeactionadapter
    POM_PACKAGING=aar
    ```

* 1 at the project level (`/MyAwesomeLibrary/gradle.properties`)  

    ```ini
    VERSION_NAME=1.3.0
    VERSION_CODE=5
    GROUP=com.wdullaer

    ANDROID_BUILD_MIN_SDK_VERSION=14
    ANDROID_BUILD_TARGET_SDK_VERSION=21
    ANDROID_BUILD_SDK_VERSION=21
    ANDROID_BUILD_TOOLS_VERSION=21.1.1

    POM_DESCRIPTION=Android Swipe Action Adapter
    POM_URL=https://github.com/wdullaer/SwipeActionAdapter
    POM_SCM_URL=https://github.com/wdullaer/SwipeActionAdapter
    POM_SCM_CONNECTION=scm:git@github.com:wdullaer/SwipeActionAdapter.git
    POM_SCM_DEV_CONNECTION=scm:git@github.com:wdullaer/SwipeActionAdapter.git
    POM_LICENCE_NAME=Apache v2
    POM_LICENCE_URL=https://github.com/wdullaer/SwipeActionAdapter/blob/master/LICENSE
    POM_LICENCE_DIST=repo
    POM_DEVELOPER_ID=wdullaer
    POM_DEVELOPER_NAME=Wouter Dullaert
    ```

* 1 in your gradle home folder. (`~/.gradle/gradle.properties`)  

    ```ini
    signing.keyId=1332E476
    signing.password=<your_gpg_key_password>
    signing.secretKeyRingFile=/home/wdullaer/.gnupg/secring.gpg

    NEXUS_USERNAME=<your_sonatype_username>
    NEXUS_PASSWORD=<your_sonatype_password>
    ```

Strictly speaking you could add all the variables to the lowest level, but if we're going through all this hassle
we might as well do it properly.


## Release the staged artifact
Once you have uploaded your artifact to Maven Central using the build script you need to release it.

1. Go to <http://oss.sonatype.org> and login with the account you created in step 1.

2. Initially your artifact is located in a so called "staging" repository. It's name is your groupId with a dash and some numbers.  
Like all proper enterprise software there is a whole bunch of other options and data shown here, that are totally irrelevant, which you should under no condition touch.

3. Once you've located your staging repository you can press the close button. This won't actually close the repository but triggers a process that checks whether the contents of the repository meets sonatype's guidelines.  
After a few seconds this process should be finished and you can hit the refresh button.

4. If your repository passes the tests. It will be marked closed and you should be able to click the release button. This will terminate the process and make your library publicly available.

Your users can now access your library by putting the following line in their `build.gradle` dependencies:

```javascript
compile 'com.wdullaer:materialdatetimepicker:1.0.0'
```

More information on this last step can be found here: <http://central.sonatype.org/pages/releasing-the-deployment.html>
