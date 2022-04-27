---
layout: post
title: "Android 封装SDK时常用的注解"
date: 2015-07-07
categories:
- Technical
tags:
- Android

---

工作中，我们经常需要将功能模块封装成库供合作厂商调用， 如何写好一个健壮的Android Library有很多讲究，使用注解可以对SDK暴露给开发者的接口做出一些限制，从而尽可能地避免开发者错误地使用API。 下面我们介绍几种封装SDK时常用到的注解。

<!-- more -->

### IntDef与StringDef

我们有时候会使用int常量或者String常量来代替枚举， 特别在你编写SDK的时候，你可以通过IntDef或者StringDef来限制接口可接受的参数。  

比如，有一个 `disableChannel`的接口，用来关闭指定的`channel` 。 我们可以先定义自己的注解`@RequirePayChannel`

``` java
public static final int CHANNEL_UNIONPAY = 0x11000;
public static final int CHANNEL_ALIPAY = 0x12000;
public static final int CHANNEL_WECHAT = 0x13000;

@Retention(RetentionPolicy.SOURCE)
@IntDef({CHANNEL_UNIONPAY,CHANNEL_ALIPAY,CHANNEL_WECHAT})
public @interface RequirePayChannel {}
```

这样，你便可以通过`@RequirePayChannel`来指定disableChannel()的可接受参数

``` java
public void enableChannel(@RequirePayChannel int channel) {
	// do something
}
```

这样，一些IDE还会自动提供给你建议参数。如果填入指点范围之外的参数，将会出现错误提示并无法编译通过。
![错误提示2][2]
值得一说的是， 在这里，我们使用到了`@Retention(RetentionPolicy.SOURCE)`。 它指定了编译器在处理Animation时候的处理方法。 默认编译器会将常量替换成对应的数值，如果没指定该注解，你编译完成后将得到这样的class文件:

![反编译RequirePayChannel][1]

这样会导致IDE不能提示到有意义的信息。并且一定要指定为特定的int数值，否则也无法编译通过。
![错误提示3][3]
所以，应该指定`Retention`让编译器不对该注解做额外的优化处理。

### DrawableRes, StringRes 与 DimenRes

当我们在编写指定资源文件的接口时，可以通过资源注解来指定该方法接受的资源类型。 指定错误的资源将不能编译通过。 下面代码中，我们使用`@DrawableRes`来表明`setLogo`方法只支持Drawable资源的ID。

```java
public void setLogo(@DrawableRes int resurceId) {
    // do something
}

```
当我们提供错误的资源，IDE将会报错。
![错误提示4][4]  

`@StringRes` 与 `@DimenRes` 的使用方法也类似。

### NonNull 与 Nullable

将一个空值传入一个方法中可能引发潜在的Crash。 我们应该极力避免这种情况， @NonNull 可以指定参数是否接受空值，当我们传入一个空值的时候，IDE会给出响应的警告。 我们可以这样使用它：

```java
public void setContext(@NonNull Context context) {
    // do something
}
```
当我们对其传入一个空值的时候，将会显示警告（但代码仍然能通过编译）
![错误提示5][5]  

`@Nullable` 用于修饰参数或者方法的返回值可能为空，提醒开发者主要空值检查。

```
@Nullable
public Context getContext() {return null;}
```

![错误提示6][6]  




[1]: /images/int_def_anination_class.png
[2]: /images/int_def_error_tip.png
[3]: /images/int_def_error_tip2.png
[4]: /images/drawable_res_error_tip.png
[5]: /images/non_null_waring_tip.png
[6]: /images/nullable_waring_tip.png
