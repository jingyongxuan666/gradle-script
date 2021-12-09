```

  // androidx
    implementation config.deps.android.appcompat
    implementation config.deps.android.fragmentKtx

    // lifecycle
    // implementation config.deps.android.lifecycle
    implementation config.deps.android.lifecycleProcess
    //  implementation config.deps.android.viewmodel
    implementation config.deps.android.viewmodelSavestate
    implementation config.deps.android.livedata

    // navigation
    implementation config.deps.android.navigation
    implementation config.deps.android.navigationUI

    implementation config.deps.android.collection

    // view
    implementation config.deps.android.material
    implementation config.deps.android.constraintlayout
    implementation config.deps.android.recyclerview
    implementation config.deps.android.swiperefreshlayout
    implementation config.deps.android.flexbox

    // kotlin
    implementation config.deps.kotlin.coroutines.android
    implementation config.deps.kotlin.coroutines.jdk8

    // arouter
    kapt config.deps.arouter.compiler
    implementation config.deps.arouter.api

    // rx
    implementation deps.rx.rx2
    implementation deps.rx.rx2Android
    implementation deps.rx.rx3
    implementation deps.rx.rx3Android

    // test
    testImplementation config.deps.test.javaJunit
    testImplementation config.deps.kotlin.coroutines.test
    androidTestImplementation config.deps.test.junit
    androidTestImplementation config.deps.test.espresso





```
