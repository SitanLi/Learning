### Cardview 组件

用圆角和阴影实现 Material Design 卡片图案。

应用通常需要在样式相似的容器中显示数据。这些容器通常在列表中用于展示每项的信息。借助系统提供的 CardView API ，可以轻松地在卡片内显示信息。这些卡片在整个平台都具有一致的外观，并且以默认位于所属试图上方绘制，系统会在其下方绘制阴影。

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.cardview:cardview:1.0.0"
}
```

#### 示例

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="16dp"
    android:orientation="vertical"
    card_view:cardElevation="0dp"
    card_view:cardCornerRadius="4dp">

    <TextView
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:gravity="center" />

</androidx.cardview.widget.CardView>
```

使用以下属性自定义 CardView 微件的外观：

- `card_view:cardCornerRadius`：设置圆角半径

- `card_view:cardBackgroundColor`：设置卡片背景色
- `card_view:cardElevation`：设置阴影深度，数值越大，阴影绘制越明显；数值越小，阴影越淡。