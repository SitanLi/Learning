### Browser组件

使用 browser 组件，将 Web 移形换影成原生 APP，TWA 是集成 Web 应用的新方法，可以通过基于 Custom Tabs 的协议将 PWA 应用和 Android app 进行集成，通过使用 Digital AssetLinks 的方式对网站和 native 进行授权，保证双方的开发者是相同的，这样便可以隐藏浏览器中的地址栏，使得网站更加 native 化。

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.browser:browser:1.3.0"
}
```

#### 新功能

##### 版本1.3.0

- 通过调用`TrustedWebActivityServiceConnection#sendExtraCommand`，可从浏览器传递自定义数据到 Trusted Web Activity 客户端。该客户端可以在`TrustedWebActivityService#onExtraCommand`中处理这些数据。
- 添加了`TrustedWebActivityCallback`接口，可供 Trusted Web Activity 客户端用来将数据返回到浏览器。
- 添加了`CustomTabsIntent#setShareState`，可让开发者指定是否显示分享选项。
- 可以使用`TrustedWebActivityIntentBuilder`中的`setScreenOrientation`方法设置默认屏幕方向。
- 向`CustomTabColorSchemeParams`中添加了`setNavigationBarDividerColor`方法，以支持更改导航栏分割线的颜色。
- 添加了`CustonTabsIntent.Builder#setDefaultColorSchemeParams`，用于替换现已启用的~~`#setNavigationBarColor`~~、~~`#setNavigationBarDividerColor`~~、~~`#setToolbarColor`~~和~~`#setSecondaryYoolbarColor`~~方法。
- 添加了`CustomTabsClient#bindCustomTabsServicePreservePriority`方法，使开发者无需使用`Content.BIND_WAIVE_PRIORITY`标志即可连接到自定义标签页服务。

##### 版本 1.2.0

- Trusted Web Activity
  - 增强 Trusted  Web Activity 的稳定性。
  - `TrustedWebActivityIntentBuilder`可用于自定义和创建`TrustedWebActivityIntent`，以启动 Trusted Web Activity。
  - 可以添加或拓展`TrustedWebActivityService`，以允许客户端浏览器向用户发送推送通知。
  - 浏览器可使用`TrustedWebActivityServiceConnectionPool`连接到客户端中的`TrustedWebActivityService`。`TrustedWebActivityServiceConnection`表示此类连接。
  - 可以启动 Trusted Web Activity，并向 Web Share Target 提供信息。
- 深色主题
  - 开发者可以（通过`CustomTabColorSchemeParams`）提供设备处于光亮模式或深色模式时使用的不同主题背景颜色。
  - 开发者可以设置浏览器的光亮或深色模式。
- 会话恢复
  - 可以创建带有ID的`CustomTabsSession`，这样能够合并从统一客户端和 ID 启动的后续标签页。
- 可以为“自定义标签页”指定导航栏颜色
- 浏览器操作相关类由于功能使用率极低而被标记为“已弃用”，并预计从未来版本删除。