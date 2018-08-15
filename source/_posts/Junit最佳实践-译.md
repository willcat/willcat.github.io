---
title: Junit最佳实践(译)
date: 2018-08-13 18:38:01
tags: [编程语言,测试,Junit,单元测试]
categories: [测试]
---
### 原文
[JUnit Best Practices](http://www.kyleblaney.com/junit-best-practices/)
### 本文针对单元测试的最佳实践目标:
1. 非常快- 你会写很多的单元测试，而且他们会被频繁的运行，所以他们需要运行的很快。我说的是秒级而非毫秒级别的快。
2. 非常可靠- 你希望通过测试找到`production`代码的问题。当且仅当`production`代码被破坏时，测试才失败。有些人认为测试间歇性失败比不测试更问题更严重。

### 1. 保证单元测试只在内存中进行
比如，不要在单元测试中发起HTTP请求、访问数据库、或者读取文件系统。这些行为太慢而且不可靠，所以最好把他们留给其他类型的测试，比如`功能测试`

读取文件系统的测试对单元测试来说太复杂了：
1. 它们需要配置当前的工作目录的位置，要想在开发机器和构建机器上都做好是比较复杂的。
2. 它们通常需要存储在源码管理中的文件，并且保持源码管理中这些文件实时更新是很复杂的。

### 2. 不要跳过单元测试
有许多方式跳过单元测试，但是你不应该这么做：
- 不要使用Junit的`@Ignore`注解
- 不要使用Maven的`maven.skip.test`属性
- 不要使用Maven Surefire插件的`skipTests`属性
- 不要使用Maven Surefire插件的`excludes`属性
被跳过的单元测试不能提供任何好处，但是仍然会被从源码管理中检出和编译。对于不需要的测试，我们应该从源码管理中移除，而不是跳过测试。

### 3. 针对每个单元测试方法只执行一个断言
- 当测试失败时，更容易判断是哪里出了问题。假如一个单元测试有三个断言，我们需要进一步尝试去判断到底是哪个断言失败了。
- 当一个单元测试执行多个断言时，不能保证每个断言都发生。比如一个`unchecked exception`发生，在这个异常之后的所有断言就都不会再出现了，Junit会把这个方法标记为有一个错误然后执行下一个测试方法。

### 4. 尽可能使用最强的断言
没有最强断言，单元测试除了覆盖其他什么都做不了。覆盖率是单元测试的有效结果，但我们可以做得更好！特别是，我们希望我们的单元测试确保我们的生产代码正常工作。如果没有强有力的断言，我们的单元测试只能确保我们的生产代码不会在我们面前爆炸。

按照强度递减的顺序，断言分属于以下级别:
- strongest assertions - 断言方法的返回值
- strong assertions - 验证与重要依赖的模拟对象交互的正确性
- weak assertions - 验证与非重要依赖的模拟对象交互的正确性

### 5. 不要验证与一个模拟logger的交互，除非日志对被测方法至关重要

### 6. 使用最合适的断言方法
- 使用 `assertTrue(classUnderTest.methodUnderTest())` 而非 `assertEquals(true, classUnderTest.methodUnderTest())`
- 使用 `assertEquals(expectedReturnValue, classUnderTest.methodUnderTest())`而非`assertTrue(classUnderTest.methodUnderTest().equals(expectedReturnValue))`
- 使用`assertEquals(expectedCollection, classUnderTest.getCollection())`,而非断言集合的大小和每个元素。

### 7. 断言参数顺序要合适
比如Junit的断言参数有:
1. `expected`
2. `actual`
则应使用`assertEquals(expected, actual)`而非`assertEquals(actual, expected)`。断言参数顺序的准确性可以确保`JUnit`消息的准确性。

### 8. 使用模拟框架(mock Framework)时应使用准备匹配

### 9. 命名一个单元测试时，使用包含被测方法和条件的惯例
我们项目中的惯例是:`methodUnderTest_condition`
比如：
- `encode_nullBytes`
- `encode_emptyBytes`
- `encode_tooFewBytes`
- `encode_tooManyBytes`
- `encode_rightNumBytes` or `encode_validBytes`

### 10. 确保测试类与被测试的生产代码类在同一个包下
当与被测试的生产类位于同一个包中时，测试类可以使用包私有类并调用包私有方法。根据我的经验，高质量的代码库广泛使用包私有类和方法来隐藏实现细节。

### 11. 确保测试代码与生产代码分开
Maven项目中的默认文件夹结构这样做:
生产代码存在于src / main / java文件夹中
测试代码存在于src / test / java文件夹中。
即使您不使用Maven，也请将测试代码和生产代码放在不同的文件夹中

### 12. 不要在单元测试中打印任何东西

### 13. 不要在单元测试类构造函数中初始化,请改用`@Before`方法
如果在测试类构造函数期间发生故障，则会发生`AssertionFailedError`，并且堆栈跟踪的信息量不大;特别是，堆栈跟踪不包括原始错误的位置。另一方面，如果在`@Before`方法期间发生故障，则可以获得有关故障位置的所有详细信息。


有些团队允许在构造函数中初始化测试类成员，当这些成员是一个简单类型（如String）时，但我个人不会初始化测试类成员。相反，我总是做以下其中一项：
- 在使用`@Before`（首选）注释的方法中为测试类成员分配值
- 将简单值分配给测试类中的`final static `成员变量

### 14.不要在测试类中使用静态成员
静态成员使单元测试方法产生依赖。不要使用它们！相反，努力编写完全独立的测试方法

### 15. 不要自己编写为了让测试失败时才存在的`catch`块
没必要编写自己的`catch`块只是为了应对测试失败，因为JUnit框架会为您处理这种情况。例如，假设您正在为以下方法编写单元测试:
``` java
final class Foo
{
  int foo(int i) throws IOException;
}
```
这里我们有一个方法接受一个整数并返回一个整数，并在遇到错误时抛出IOException。这是编写单元测试的错误方法，该测试确认方法在传递7时返回3
``` java
// Don't do this - it's not necessary to write the try/catch!
@Test
public void foo_seven()
{
  try
  {
    assertEquals(3, new Foo().foo(7));
  }
  catch (final IOException e)
  {
    fail();
  }
}
```
正在测试的方法指定它可以抛出IOException，这是一个经过检查的异常。因此，除非您捕获异常或声明测试方法可以传播异常，否则单元测试将无法编译。第二种替代方案是优选的，因为它导致更短和更集中的测试
``` java
// Do this instead
@Test
public void foo_seven() throws Exception
{
  assertEquals(3, new Foo().foo(7));
}
```

### 16. 不要编写自己的catch块，只是为了通过测试
不要这么写：
``` java
// Don't do this - it's not necessary to write the try/catch!
@Test
public void foo_nine()
{
  boolean wasExceptionThrown = false;
  try
  {
    new Foo().foo(9);
  }
  catch (final IOException e)
  {
    wasExceptionThrown = true;
  }
  assertTrue(wasExceptionThrown);
}
```
应该:
``` java
// Do this instead
@Test(expected = IOException.class)
public void foo_nine() throws Exception
{
  new Foo().foo(9);
}
```

### 17. 不要编写自己的catch块，仅用于打印堆栈跟踪
``` java
// Don't do this - it's not necessary to write the try/catch!
@Test
public void foo_seven()
{
  try
  {
    assertEquals(3, new Foo().foo(7));
  }
  catch (final IOException e)
  {
    e.printStackTrace();
  }
}
```
应该：
``` java
// Do this instead
@Test
public void foo_seven() throws Exception
{
  assertEquals(3, new Foo().foo(7));
}```

### 18. 在测试类中，不要声明方法抛出任何特定类型的异常
``` java
// Don't do this - the throws clause is too specific!
@Test
public void foo_seven() throws IOException
{
  assertEquals(3, new Foo().foo(7));
}
```
这种测试方法很脆弱。想象一下，测试中的方法发生了变化，导致它抛出`IOException`或`GeneralSecurityException`。在这种情况下，我们必须更改它的测试方法进行编译
所以我们应该
```java
// Do this instead
@Test
public void foo_seven() throws Exception
{
  assertEquals(3, new Foo().foo(7));
}
```

### 19. 不要在单元测试中使用Thread.sleep
当单元测试使用Thread.sleep时，它不能可靠地指示生产代码中的问题。例如，这样的测试可能会失败，因为它在比平常慢的机器上运行。目标是当且仅当生产代码被破坏时才会失败的单元测试.

不应该在单元测试中使用Thread.sleep，而是重构生产代码以允许注入模拟对象，该模拟对象可以模拟通常必须等待的可能长时间运行的操作的成功或失败

### 20. 不要尝试测试直接或间接调用Thread.sleep的生产方法的时间
同上

### 21. 
