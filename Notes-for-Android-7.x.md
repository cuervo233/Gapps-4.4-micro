## WebView Providers
Since Android Nougat 7.0 the flexibility and amount of WebViewProviders has been greatly extended. Next to AOSP WebView and Google WebView, Google Chrome (and its Beta and Dev variants) can be used as WebView provider too. Users have the possibility to select their WebView provider from the Settings Screen.

For ROM developers it is a necessity to include a list of valid WebView Providers. I.e. on Google's Nexus devices this list consists of Google Chrome variants and Google WebView.
In the upstream AOSP code only AOSP WebView is defined by default. **ROM developers should apply [this patch](https://github.com/AOSPA/android_frameworks_base/commit/d36582165d4694da101cc65755af0841d443c80e) to their source tree** to add Google WebView and Google Chrome (plus its variants) as valid WebView providers and achieve functional equivalence with stock ROMs.

Additionally there was a security fix for above patch deployed. **It is recommended to apply [this patch](https://github.com/AOSPA/android_frameworks_base/commit/b70f5994464cf6b3b29cedcc4efdd73807a27b0f) as well.**

Later a mistake in the generation of the signatures was discovered, which would break Chrome Stable and Google Webview to be recognized as Webview Providers on "user" builds. **As a fix [this patch](https://github.com/AOSPA/android_frameworks_base/commit/0b18d8bac487a367f6a1d985fd72dc4582f8ddaa) should be applied on top of the other two patches.**

***

### APK Signature Scheme v2
With Android Nougat, Google introduced the new [APK Signature Scheme v2](https://source.android.com/security/apksigning/v2.html).
This new signature scheme could pose problems for GApps packages in future versions of Android. But luckily for [Nougat there is still an exception for system apps](https://android.googlesource.com/platform/frameworks/base/+/nougat-release/core/java/android/content/pm/PackageParser.java#1204).

If a Google app receives an update from the Play Store, the libraries in its APK are usually compressed. But the libraries need to be stored in a decompressed manner within the APK to make them install-able on the `/system` partition. This results in Open GApps having to re-package the APK which changes its *outside*. This did not intervene with the original signature scheme (that only verified the *inside* of the APK), but does not pass the v2 scheme's check anymore.

Failing to pass the v2 scheme is only fatal if `X-Android-APK-Signed: 2` is included in the `META-INF/*.SF` files. But unfortunately this header is included in Google's most recent apps.
This implies that the ROM needs a patch to allow APKs on `/system` to fail the v2 test even if the `X-Android-APK-Signed: 2` header is present and only check the v1 signature.