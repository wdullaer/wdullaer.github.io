---
layout: post
title: "Release an Android Library on Maven Central"
quote: "A lot more hassle than it should be: now explained"
image: false
video: false
comments: true
---
# How to release an Android Library on Maven Central

## Introduction
I was going to put "the easy way", but well the reality is that there is no easy way.
Hopefully this post can make the entire experience a bit less painful for you.

You will need the following ingredients

1. An Android library
2. [An account on sonatype](#create-a-sonatype-account)
3. [A gpg key](#create-a-gpg-key)
4. [A gradle build script](#get-the-gradle-build-script)
5. [Lot's of gradle configuration](#configure-gradle)
6. [Perform the release steps on sonatype](#promote-the-staged-release)


I'm going to assume that you have a library in a releasable state and go straight to step 2


## Create a sonatype account
<https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134>

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
This is in fact the easiest step. Someone went through the trouble of creating a proper Maven
release build script at <https://raw.githubusercontent.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle>

You can include this one directly into the root of your `build.gradle` of your library.

```javascript
apply from: 'https://raw.github.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle'
```

Or if you're the paranoid sort: you can add a hardcopy to your project and reference that.

## Configure gradle
You will need to create a total of 3 `gradle.properties` files:

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


## Promote the staged release
<http://central.sonatype.org/pages/releasing-the-deployment.html>
