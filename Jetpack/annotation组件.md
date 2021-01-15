### annotation 组件

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.annotation:annotation:1.1.0"
    // To use the Java-compatible @Experimental API annotation
    implementation "androidx.annotation:annotation-experimental:1.0.0"
}
```

#### 新功能

- Jetpack annotation 库提供了 Kotlin 对 Java 的兼容实现。`-lint`工件会限制使用基于 Lint 实现的实验。
- 将 `annotation-experimental`工件用于依赖项时，`annotation-experimental`工件提供的 Lint 规则会自动强制执行。

#### 各注解分解

`AnimatorRes`：表示整数参数、字段或方法的返回值应该是动画（animator）资源引用。

`AnimRes`：表示整数参数、字段或方法的返回值应该是动画（anim）资源引用。

`AnyRes`：表示整数参数、方法或方法的返回值应该是任何类型的资源引用。如果已知类型，请改用更具体的注释之一，例如`StringRes`或`DrawableRes`。

`AnyThread`：表示可以从任何线程调用带注释的方法（它是线程安全的）。如果带注释的元素是`class`，则可以从任何线程调用该类中的所有方法。

`ArrayRes`：表示整数参数、方法或方法的返回值应该是数组资源引用。

`AttrRes`：表示整数参数、方法或方法的返回值应该是属性资源引用。

`BinderThread`：表示带注释的方法只能在`binder thread`上调用。如果带注释的元素是`class`，表示类中的所有方法只能在`binder thread`上调用。

`BoolRes`：表示整数参数、方法或方法的返回值应该是布尔资源引用。

`CallSuper`：表示任何覆盖此方法的子类也应调用此方法。

```kotlin
Persion.kt
@CallSuper
open fun initName(){

}

Student.kt
override fun initName() {
    super.initName()
}
```

`CheckResult`：检查某个方法的返回值是否被使用，如果没有被使用，则会根据`suggest`配置建议使用相同的没有返回值的另一个方法。

`ColorInt`：表示带注释的元素表示一个颜色`AARRGGBB`，如果应用于数组，表示数组中的每一个元素都代表一个颜色。

`ColorLong`：表示带注释的元素代表一个打包的颜色长。

`ColorRes`：表示整数参数、方法或方法的返回值应该是颜色资源引用。

`ContentView`：表示将单个 LayoutRes 参数作为构造方法的注释，用来表明该组件打算填充布局并将其设置为内容。

`DimenRes`：表示整数参数、方法或方法的返回值应该是`dimen`资源引用。

`Dimension`：表示期望的整数参数、方法或方法的返回值应该是`dimen`类型。`unit`默认值为`PX`，分别还有`DP`、`PX`、`SP`类型。

`DoNotInline`：表示在构建优化代码时，不应内联带注释的方法。这通常用于避免刻意地在单独类中使用内联方法。

`DrawableRes`：表示整数参数、方法或方法的返回值应该是`drawable`、`mipmap`资源引用。

`FloatRange`：表示带注释的元素应为给定范围内的`float`或`double`。`from`：最小值；`fromInclusive`：起始值是否包含在范围内；`to`：最大值；`toInclusive`：最大值是否包含在范围内。

`FontRes`：表示整数参数、方法或方法的返回值应该是`font`资源引用。

`FractionRes`：表示整数参数、方法或方法的返回值应该是`fraction`资源引用。

`GuardedBy`：表示仅当持有引用的锁时才可以访问带注释的方法或字段。

`HalfFloat`：表示带注释的元素表示半精度浮点值。这些值存储在简短的数据类型中，并且可以使用`android.util.Half`类进行操作。如果应用于`short`数组，则数组中的每个元素都代表半精度浮点数。

`IdRes`：表示整数参数、方法或方法的返回值应该是`id`资源引用。

`InspectableProperty`：表示带注释的方法是资源支持的属性的获取方法，该属性应显示在 Android Studio 的检查工具中。——不知道干哈的。

`IntDef`：表示整数类型的值应该是显式命名的常量之一。值得注意的是，Kotlin1.0.3以后已经不支持注释，可以使用Java文件编写IntDef注释。参数解释，flag，定义将常量用做标志还是枚举（默认值）；open，是否允许任何其他值；value，定义此元素允许的常量。

```kotlin
@Retention(RetentionPolicy.SOURCE)
@IntDef(value = {TYPE_1, TYPE_2, TYPE_3}, flag = true)
public @interface Type {
    public static final int TYPE_1 = 1;
    public static final int TYPE_2 = 2;
    public static final int TYPE_3 = 3;
}

private var type:Int = TYPE_1
fun setType(@Type type:Int)
```

`IntegerRes`：表示整数参数、方法或方法的返回值应该是`integer`资源引用。

`InterpolatorRes`：表示整数参数、方法或方法的返回值应该是`interpolator`资源引用。

`IntRange`：表示带注释的元素应为给定范围内的`int`或`long`。

`Keep`：表示在构建时不应删除带该注释的元素。它通常用于通过反射的方法和类。因为编辑器可能认为代码未被使用。

`layoutRes`：表示整数参数、方法或方法的返回值应该是`layout`资源引用。

`LongDef`：参考`IntDef`。

`MainThread`：表示带注释的方法只能在主线程调用。

`MenuRes`：表示整数参数、方法或方法的返回值应该是`menu`资源引用。

`NavigationRes`：表示整数参数、方法或方法的返回值应该是`navigation`资源引用。

`NonNull`：表示参数、字段或方法的返回值永远不能为null。

`Nullable`：表示参数、字段或方法的返回值可以为null。

`PluralsRes`：表示整数参数、方法或方法的返回值应该是`plurals`资源引用

`Px`：表示期望整数参数、字段或方法的返回值表示像素尺寸。

`RawRes`：表示整数参数、方法或方法的返回值应该是`raw`资源引用。

`RequiresApi`：表示仅在给定的API级别或更高的API上调用带注释的元素或方法。

`RequiresFeature`：表示带注释的元素需要一个或多个`features`。这用于自动生成文档，更重要的是，为了确保应用程序代码的正确使用。name，所需功能的名称；enforcement，使用与javadoc相同的签名格式，定义用于检查功能是否可用的方法名称。

`RequiresPermission`：表示带注释的元素需要一个或多个权限。value，如果只需要一个权限时赋值；allOf，制定所有必须的权限名称列表；anyOf：指定一列权限名称，其中至少需要一个；conditional，如果为true，则不需要所有的权限都允许。

`RequiresPermission.Read`：指定读取操作需要给定的权限。当在参数上指定时，表示该方法仅需要一个权限。

`RequiresPermission.Write`：指定写操作需要给定的权限。

`RestrictTo`：表示应仅从特定范围内访问带注释的元素。LIBRARY，限制在同一库中使用；LIBRARY_GROUP，限制在同一组库中使用；LIBRARY_GROUP_PREFIX，限制在统一前缀库中使用；TESTS，限制在测试代码中使用；SUBCLASSES：限制在子类中使用。

`Size`：表示带注释的元素应具有给定的大小或长度。-1表示未设置。

`StringDef`：参考`IntDef`。

`StringRes`：表示整数参数、方法或方法的返回值应该是`string`资源引用。

`StyleRes`：表示整数参数、方法或方法的返回值应该是`style`资源引用。

`TransitionRes`：表示整数参数、方法或方法的返回值应该是`transitionRes`资源引用。

`UiThread`：表示带注释的方法或构造函数仅在UI线程上调用。通常，应用的UI线程也是主线程，但是在特殊情况下，UI线程可能不是主线程。

`VisibleForTesting`：表示该类、方法或字段在测试期间的可见性得到了放宽，您可以选择指定可见性。

`WorkThread`：表示带注释的方法只能在工作线程上调用。如果带注释的元素是类，则应在工作线程上调用该类中的所有方法。

`XmlRes`：表示整数参数、方法或方法的返回值应该是`xml`资源引用。