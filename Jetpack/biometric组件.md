### Biometric 组件

通过生物识别特征或设备凭据进行身份验证，以及执行加密操作。

如果需要集成生物识别功能，可参阅API：https://developer.android.com/training/sign-in/biometric-auth

#### 依赖声明

```groovy
dependencies {
    // Java language implementation
    implementation "androidx.biometric:biometric:1.0.1"

    // Kotlin
    implementation "androidx.biometric:biometric:1.2.0-alpha01"
    //ktx
    implementation "androidx.biometric:biometric-ktx:1.2.0-alpha01"
  }
```

#### 使用方式

如需定义应用支持的身份验证类型，使用`BiometricManager.Authenticators`接口，系统允许使用声明一下的身份验证：

- `BIOMETRIC_STRONG`：使用第 3 类生物识别进行身份验证
- `BIOMETRIC_STRONG`：使用第 2 类生物识别进行身份验证
- `DEVICE_CREDENTIAL`：使用屏幕锁定凭据（即用户的PIN码、解锁图案或密码）进行身份验证。

为了注册身份验证器，用户需要创建 PIN 码、解锁图案或密码。如果用户尚无 PIN 码、解锁图案或密码，生物识别注册流程会提示他们创建一个。

以下代码展示了如何使用 3 类生物识别或屏幕锁定凭据支持身份验证：

```kotlin
executor = ContextCompat.getMainExecutor(this)
biometricPrompt = BiometricPrompt(this, executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationError(errorCode: Int,
                                           errString: CharSequence) {
            super.onAuthenticationError(errorCode, errString)
            Toast.makeText(applicationContext,
                "Authentication error: $errString", Toast.LENGTH_SHORT)
                .show()
        }

        override fun onAuthenticationSucceeded(
            result: BiometricPrompt.AuthenticationResult) {
            super.onAuthenticationSucceeded(result)
            Toast.makeText(applicationContext,
                "Authentication succeeded!", Toast.LENGTH_SHORT)
                .show()
        }

        override fun onAuthenticationFailed() {
            super.onAuthenticationFailed()
            Toast.makeText(applicationContext, "Authentication failed",
                Toast.LENGTH_SHORT)
                .show()
        }
    })

promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Biometric login for my app")
    .setSubtitle("Log in using your biometric credential")
    .setNegativeButtonText("Use account password")
    .build()

// Prompt appears when user clicks "Log in".
// Consider integrating with the keystore to unlock cryptographic operations,
// if needed by your app.
val biometricLoginButton =
    findViewById<Button>(R.id.biometric_login)
biometricLoginButton.setOnClickListener {
    biometricPrompt.authenticate(promptInfo)
}
```

#### 新功能

##### 版本1.2.0

- 引入了`androidx.biometricLbiometric-ktx`模块，此模块可基于`androidx.biometric:biometric`添加 kotlin 专用 API 和拓展。

##### 版本1.1.0

###### 版本 1.1.0-beta01

- 在 Android 8.0 及更低版本上，使用静态资源替换对话框对话，大大降低了库的 APK 占用空间。
- 现在，如果生物识别身份验证工作流被锁定，`BiometricPrompt`会在所有受支持的 Android 版本中自动回退到设备凭据身份验证工作流。

###### 版本 1.1.0-alpha02

- `BiometricManager#canAuthenticate()`现已可以返回`BIOMETRIC_STATUS_UNKNOW`以指示用户也许仍然可以进行身份验证或返回`BIOMETRIC_ERROR_UNSUPPORTED`以指示相应设备不支持某身份验证器组合。
- `BiometricPrompt#authenticate()`目前仅在 Android 11 （Api 30）及更高版本上可与相关联的`CryptoObject`用于设备凭据身份验证。

###### 版本 1.1.0-alpha01

- 重构了内部库实现，以解决潜在的内存泄漏和其他错误；
- 内部 Fragment 现在使用与客户端应用的Activity生命周期相关联的`ViewModel`来共享和保留数据。
- Android 10（API 29）之前的设备凭据身份验证不会再在客户端应用中气动透明Activity。

##### 版本1.0.0

- 已在 Android 10 中实现`BiometricPrompt`和`BiometricManager` API 兼容性版本，可向前兼容 Android 6.0 的全部功能（API 23）。
- 在`Fragment`或`FragmentActivity`中为`BiometricPrompt`内置生命周期管理功能。
- 对已知在基于加密的身份验证期间呈现错误的弱生物识别特性的设备进行特殊处理。

