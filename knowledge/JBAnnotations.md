来自于 http://ice1000.tech/2017/01/06/JBAnnotations.html  http://ice1000.tech/2017/01/07/JBAnnotations2.html

-----

maven坐标
```xml
<dependency>
    <groupId>org.jetbrains</groupId>
    <artifactId>annotations</artifactId>
    <version>15.0</version>
</dependency>
```

这个包在任何一个 JetBrains IDE 的安装目录里面都有，在 kotlin-runtime.jar 里面也有，在 maven 仓库也有。

#### 为什么需要它

首先我们知道 Kotlin 解决了一个万年大问题—— null 。 几乎每个代码量超过两千的项目，先做出原型后，前两次运行或单元测试，都死在 NullPointerException 上。 Kotlin 从根本上解决了这个问题——它要求你在编译期处理所有可能为 null 的情况，这是语言级别的支持，比C#那种只提供一个语法糖的半吊子好得多。

但是反观 Java ，就没这玩意。 因此，伟大的 JetBrains 就搞了个注解库，然后通过在 IDE 里面提示你处理那些可能为 null 的值（毕竟你没法直接让编译器自己检查并提示）来解决这个问题 （其实人家自己早就在用了，只是 Kotlin 也用到了这个注解库而已）。

本次讲五个注解。我写的全限定名，因为把包名也写出来可以方便读者区分它和另外一套同名注解（sun的垃圾玩意）。还有一个很重要的注解 @Contract ，下一篇博客再讲。

第一个注解， `@TestOnly`

```java
@org.jetbrains.annotations.TestOnly
```

这个说都不用说了，就是专门写给单元测试服务的相关代码的注解。比如我的算法库，在单元测试端我写了对应的暴力算法来验证我的算法的正确性（这种方法在 OI 中称为对拍），那么这几个暴力算法的实现就适合使用 @TestOnly 注解修饰。

```java
/**
 * brute force implementation of binary indexed tree.
 */
private inner class BruteForce
@TestOnly
internal constructor(length: Int) {
	@TestOnly
	private val data = LongArray(length)

	@TestOnly
	fun update(from: Int, to: Int, value: Long) {
		(from..to).forEach { data[it] += value }
	}

	@TestOnly
	fun query(from: Int, to: Int): Long {
		var ret = 0L
		(from..to).forEach { ret += data[it] }
		return ret
	}

	@TestOnly
	operator fun get(left: Int, right: Int) = query(left, right)
}
```

第二、三个注解， `@NotNull` 和 `@Nullable`

```java
@org.jetbrains.annotations.NotNull
@org.jetbrains.annotations.Nullable
```
这个很简单，顾名思义。它们一般被用来注解带有返回值的方法、方法参数、类的成员变量。

当 `@NotNull` 注解一个方法参数的时候， IDE 会在调用处提示你处理 **null** 的情况（当然，如果 IDE 语义上认为你传进去的参数不可能是 **Null** ，那么当然没有提示了）； 当它注解一个有返回值的方法的时候，它会检查返回值是否可能是 null 。如果可能，那也会有提示。

当 `@Nullable` 注解一个方法参数的时候， IDE 会在方法内部提示你处理该参数为 **null** 的情况； 当它注解一个有返回值的方法的时候，会在调用处提示你处理方法返回值为 **null** 的情况。

相应的，任何以 `@NotNull` 修饰的变量/属性/方法，它在 Kotlin 中对应的类型都写作它本身，即非 **null** 类型，任何以 `@Nullable` 修饰的变量/属性/方法，它在 Kotlin 中对应的类型都写作它本身后面跟一个问号，即可能为 null 的类型。 这和 Java 的区别在于， Kotlin 编译器强制你处理 null ， Java 只有 IDE 警告。 另外，这个注解本身的命名也是一种警告，不过是对于开发者而言的。

第四、五个注解， `@Nls` 和 `@NonNls`

```java
@org.jetbrains.annotations.Nls
@org.jetbrains.annotations.NonNls
```
这是两个十分有意思的注解，用于修饰字符串，而且是写给人看的，这个就不是给 IDE 看的辣。 @Nls 用于修饰自然语言字符串，比如下面这个例子。 假设 textArea 是一个 JTextArea ，是一个游戏画面用于显示一些提示信息的框框。 这些提示信息当然是自然语言了。

所以就可以这样修饰：
```java
public void boyNextDoor(@Nls String msg) {
	textArea.append(msg);
}
```
或者用于程序的 Crash 信息：
```java
public DividedByZeroException(@NotNull @Nls String msg) {
	super(msg);
}
```
然后阅读代码的人就知道，哦，这个 msg 里面是一些自然语言，比如什么”My name is Van, I’m an artist.”之类的。

反之， `@NonNls` 就是用于修饰非自然语言。 比如我的算法库有一个大整数类，其中有一个构造方法接收一个字符串，然后这个大整数就从字符串构造。 这个字符串一般长这样： “23333333333333333333333333” 或者： “-6666666666666666666666666666666”。

这显然不是自然语言！于是：
```java
public BigInt(@NotNull @NonNls String origin) {
	boolean sigTemp;
	if (origin.startsWith("-")) {
		sigTemp = false;
		origin = origin.substring(1);
	} else sigTemp = true;
	byte[] ori = origin.getBytes();
	if (ori.length == 1 && ori[0] == '0') sigTemp = true;
	data = ori;
	sig = sigTemp;
}
```


最后的注解 `@Contract`
```java
@org.jetbrains.annotations.Contract
```

我已经在上一篇博客提到了这个神奇的 `@Contract` 注解。事实上，这个注解能描述的内容更详细，还能在一定程度上代替 `@NotNull` 和 `@Nullable` 。

这是一个有值的、用于修饰那些参数数量大于零并且返回值不为 void 的普通方法和构造方法的注解，比起 @NotNull 和 @Nullable ，它相对来说更复杂。因此，首先我们得看看它的源码。
```java
package org.jetbrains.annotations;

import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
public @interface Contract {
	String value() default "";

	boolean pure() default false;
}
```
可以看到，这个注解类有两个属性。首先有一个 value 的字符串属性，还有一个 pure 的布尔属性。

首先从容易理解的 pure 说起吧。

pure 属性

这个属性听名字都能猜出什么意思——表示被注解的函数（就是方法，包含普通方法、静态方法和构造方法，下同）是否为纯函数。

纯函数（好像）是一个 fp 里面的概念，如果一个函数，对于特定的输入，都将产生对应的唯一的输出，并且不会影响别的东西（即没有副作用），那么这个函数就是纯函数。 数学上的函数就是最标准的纯函数，比如我有一个 f(x) ，那么对应每一个 x ，这个函数的输出都有一个对应的 f(x) ，并且我多次输入同一个 x ，输出的 f(x) 也是同一个。 这就是一个纯函数。

举个反例，下面是一段完整的（即可以编译运行的） C 语言代码，这里的函数 not_Pure 就不是一个纯函数。

```c
#include <stdio.h>

int not_Pure(int assWeCan) {
	static int boyNextDoor = 233;
	return ++boyNextDoor * assWeCan;
}

int main(const int argc, const char *argv[]) {
	printf("%d\n", not_Pure(5));
	printf("%d\n", not_Pure(5));
	printf("%d\n", not_Pure(5));
	printf("%d\n", not_Pure(5));
	printf("%d\n", not_Pure(5));
	return 0;
}
```
我在 main 函数里面对 not_Pure 进行了五次调用，每次传进去的参数都是 5 ，但是它却产生了这样的输出：
```
1170
1175
1180
1185
1190
```
好。话题回到 @Contract 上。那么这个 pure 值怎么使用呢？

当你的函数是一个纯函数时（比如三角函数运算，这是再简单不过的纯函数了），你就可以这样修饰。
```java
	@Contract(pure = true)
	public static native double sin(double a);

	@Contract(pure = true)
	public static native double cos(double a);

	@Contract(pure = true)
	public static native double tan(double a);

	@Contract(pure = true)
	public static native double cot(double a);

	@Contract(pure = true)
	public static native double csc(double a);

	@Contract(pure = true)
	public static native double sec(double a);
```
再比如我算法库大整数的 JNI 端和 Java 端，加减乘除和比大小都不会影响原来的两个算子，会新分配一块内存来放置运算结果，那么这些函数也统统可以使用 @Contract(pure = true)注解。

value 属性

这是 @Contract 注解的精髓，应用场景也非常好说明。 首先考虑一个函数——大整数类的 equals 。 我当然是需要处理一些特殊情况的，比如如果你传进去了一个 null ，那么我返回 true 。于是按照 Java 标准库那个注释思路，我们应该这样写：
```java
/**
 * This is a pure function.
 * returns true if the value of given
 * parameter is equaled to the caller.
 *
 * if obj is null, it will return false.
 * if obj is not a org.algo4j.BigInt, it will return false.
 *
 * @param obj the given obj
 * @returns if obj is equaled to the caller
 */
@Override
public boolean equals(@Nullable Object obj) {
	if (obj == null || !(obj instanceof BigInt)) return false;
	if (obj == this) return true;
	return compareTo((BigInt) obj) == 0;
}
```
但是，仔细想想，其实你只是需要告诉用户，要是你给我 null ，我就还你 false 。 使用一大坨自然语言描述这个简单逻辑，不仅会让人看很久，而且如果注释丢了，用户也就无从得知这个情况了，而且这对于英语不好的人非常不友好。

于是我们可以这样写：
```java
@Override
@Contract(value = "null -> false", pure = true)
public boolean equals(@Nullable Object obj) {
	if (obj == null || !(obj instanceof BigInt)) return false;
	if (obj == this) return true;
	return compareTo((BigInt) obj) == 0;
}
```
首先我们省去了一大堆注释，并且使用了一个字符串描述这个逻辑： “null -> false” 。意思就是，给我一个 null ，还你一个 false 。

这个字符串由两部分组成，箭头前面的是参数说明，后面是返回值说明。如果有多种需要说明的情况，那么使用分号隔开。

一共有这么几个允许使用的值：
```
null // null
!null // not null
true // boolean value true
false // boolean value falses
fail // means this function will not work in this case
_ // any value
```
一共六个，不要忽略了最后一个下划线，它的含义和 Scala 中的下划线相同——表示通配符。比如这个扩展欧几里得函数：
```java
@NotNull
@Contract(value = "_, _ -> !null", pure = true)
public static ExgcdRes exgcd(long a, long b) {
	return new ExgcdRes(exgcdJni(a, b));
}
```
直接两个注解说明一切：首先返回值为非 null ，因此 @NotNull 。 然后，不论你传进来任何值，我都不会返回 null 的，因此 @Contract(“_, _ -> !null”) 。 最后，它是一个纯函数，因此 pure = true 。综上，有了这个函数的注解：
```java
@NotNull
@Contract(value = "_, _ -> !null", pure = true)
```
我那几个大整数加减乘除的方法就可以这样写了：

本地接口：
```java
@NotNull
@Contract(value = "!null, !null -> !null", pure = true)
private static native byte[] plus(@NotNull byte[] a, @NotNull byte[] b);

@NotNull
@Contract(value = "!null, !null -> !null", pure = true)
private static native byte[] times(@NotNull byte[] a, @NotNull byte[] b);

@NotNull
@Contract(value = "!null -> !null", pure = true)
private static native byte[] times10(@NotNull byte[] a);

@NotNull
@Contract(value = "!null, !null -> !null", pure = true)
private static native byte[] divide(@NotNull byte[] a, @NotNull byte[] b);

@NotNull
@Contract(value = "!null, !null -> !null", pure = true)
private static native byte[] minus(@NotNull byte[] a, @NotNull byte[] b);

@NotNull
@Contract(value = "!null, _ -> !null", pure = true)
private static native byte[] pow(@NotNull byte[] a, int pow);

@Contract(pure = true)
private static native int compareTo(@NotNull byte[] a, @NotNull byte[] b);
调用：

@NotNull
@Contract(value = "_ -> !null", pure = true)
public BigInt plus(@NotNull BigInt anotherBigInt) {
	if (sig == anotherBigInt.sig)
		return new BigInt(plus(data, anotherBigInt.data), sig);
	if (compareTo(data, anotherBigInt.data) > 0)
		return new BigInt(minus(data, anotherBigInt.data), sig);
	return new BigInt(minus(anotherBigInt.data, data), !sig);
}

@NotNull
@Contract(value = "_ -> !null", pure = true)
public BigInt minus(@NotNull BigInt anotherBigInt) {
	if (sig != anotherBigInt.sig)
		return new BigInt(plus(data, anotherBigInt.data), sig);
	if (compareTo(data, anotherBigInt.data) > 0)
		return new BigInt(minus(data, anotherBigInt.data), sig);
	return new BigInt(minus(anotherBigInt.data, data), !sig);
}

@NotNull
@Contract(value = "_ -> !null", pure = true)
public BigInt times(@NotNull BigInt anotherBigInt) {
	return new BigInt(times(data, anotherBigInt.data), sig == anotherBigInt.sig);
}
```
是不是很简单呢？其实有时 JetBrains IDE 还会给出建议，让你为你的方法加上这些注解哦。注解库也不大，就几十 kb 。

