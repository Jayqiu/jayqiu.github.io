#  安卓中的@Nullable和NotNull 等 注释

@Nullable和NotNull

这些注解是用来标注方法是否能传入null值，如果可以传入NUll值，则标记为nullbale，如果不可以则标注为Nonnull. 在我们做了一些不安全严谨的编码操作的时候,这些注释会给我们一些警告。


* 1.@Nullable:指明一个参数，字段或者方法的返回值 告诉编译器  参数可为空

* 2.@NotNull:指明一个参数，字段或者方法的返回值 告诉编译器，参数非空


* 3.@IdRes  声明参数是个id

* 4.@StringRes  声明这个 int 参数是个字符串资源

* 5.@StyleRes  声明参数是个style 类型

* 6.@LayoutRes  声明参数是个layout类型 


其它的类似：@DimenRes @DrawableRes @RawRes @ColorRes @XmlRes @BoolRes @In
