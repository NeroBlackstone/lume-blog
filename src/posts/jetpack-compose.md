---
title: Jetpack Compose学习笔记
description: ''
date: 2021-08-12
img: https://res.cloudinary.com/neroblackstone/image/upload/v1628761044/Android_logo_2019_iukgvk.png
tags:
- Android
- Jetpack Compose
- Kotlin

---
## 开始之前

[Jetpack Compose](https://developer.android.com/jetpack/compose) 是一个现代工具包，旨在简化 UI 开发。它结合了反应式编程模型和 Kotlin 编程语言的简洁性和易用性。它是完全声明式的，这意味着您可以通过调用一系列将数据转换为 UI 层次结构的函数来描述您的 UI。当底层数据发生变化时，框架会自动调用这些函数，为您更新视图层次结构。

Compose 应用程序由可组合函数组成 - 只是标有 `@Composable` 的常规函数​​，可以调用其他可组合函数。创建新的 UI 组件只需要一个函数。注释告诉 Compose 为随着时间的推移更新和维护 UI 的功能添加特殊支持。 Compose 可让您将代码组织成小块。可组合函数通常简称为“可组合”。

通过制作小型的可重用组合 - 可以轻松构建应用程序中使用的 UI 元素库。每个人负责屏幕的一部分，可以独立编辑。

> 注意：在此 Codelab 中，术语“UI 组件”、“可组合函数”和“可组合”可互换使用以指代同一概念。

## Compose 入门

### 可组合函数

**可组合函数**是用`@Composable` 注释的常规函数​​。这使您的函数能够调用其中的其他 `@Composable` 函数。您可以看到如何将 `Greeting` 函数标记为 `@Composable`。此函数将生成一段 UI 层次结构，显示给定的输入**`String`**。**`Text`**是库提供的可组合函数。

``` kotlin
@Composable
fun Greeting(name: String) {
   Text(text = "Hello $name!")
}
```

> 注意：可组合函数是用@Composable 注释标记的 Kotlin 函数，如您在上面的代码片段中所见。

### 在 Android 应用中组合

使用 Compose，`Activities`仍然是 `Android` 应用程序的入口点。在我们的项目中，`MainActivity` 在用户打开应用程序时启动（在 `AndroidManifest.xml` 文件中指定）。您使用 `setContent` 来定义您的布局，但不是像在传统视图系统中那样使用 XML 文件，而是在其中调用 Composable 函数。

``` kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BasicsCodelabTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    Greeting("Android")
                }
            }
        }
    }
}
```

`BasicsCodelabTheme` 是一种设置可组合函数样式的方法。您将在主题化您的应用部分中看到有关此内容的更多信息。要查看文本在屏幕上的显示方式，您可以在模拟器或设备中运行应用程序，或者使用 Android Studio 预览。

要使用 Android Studio 预览，您只需使用 `@Preview` 注释标记任何无参数的可组合函数或带有默认参数的函数并构建您的项目。您已经可以在 `MainActivity.kt` 文件中看到 `Preview Composable` 函数。您可以在同一个文件中有多个预览并为其命名。

``` kotlin
@Preview(showBackground = true, name = "Text preview")
@Composable
fun DefaultPreview() {
    BasicsCodelabTheme {
        Greeting(name = "Android")
    }
}
```

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628840174/88d6e7a2cfc33ed9_jwjgfq.png)

> 注意：在本项目中导入与 Jetpack Compose 相关的类时，请使用来自：
>
> * androidx.compose.* 用于编译器和运行时类
> * androidx.compose.ui.* 用于 UI 工具包和库

## 声明式用户界面

如果要为 `Greeting` 设置不同的背景颜色，则需要定义一个包含它的 `Surface`

``` kotlin
import androidx.compose.ui.graphics.Color
...

@Composable
fun Greeting(name: String) {
    Surface(color = Color.Yellow) {
        Text (text = "Hello $name!")
    }
}
```

嵌套在 `Surface` 内部的组件将绘制在该背景颜色之上（除非另一个 `Surface` 另有指定）。

### 修饰符

大多数 Compose UI 元素（如 `Surface` 和 `Text`）都接受可选的修饰符参数。修饰符参数告诉 UI 元素如何在其父布局中进行布局、显示或行为。修饰符是常规的 Kotlin 对象。

您可以将它们分配给变量并重用它们。由于工厂扩展函数或合并为单个参数的函数，您还可以一个接一个地链接这些修饰符中的几个。

`padding`修饰符将在它装饰的元素周围应用一定量的空间。为了给屏幕上的文本添加内边距，您可以将修饰符添加到`Text`中：

``` kotlin
import androidx.compose.foundation.layout.padding
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
...

@Composable
fun Greeting(name: String) {
    Surface(color = Color.Yellow) {
        Text(text = "Hello $name!", modifier = Modifier.padding(24.dp))
    }
}
```

单击“构建和刷新”以查看新更改。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628840808/fad946d5bf776ee3_zqjcob.png)

### 组合可重用性

您添加到 UI 的组件越多，您创建的嵌套级别就越多，就像代码库中的其他功能一样。如果函数变得非常大，这会影响可读性。通过制作小型可重用组件，可以轻松构建应用程序中使用的 UI 元素库。每个人负责屏幕的一小部分，可以独立编辑。

> 注意：@Composable 注释仅对于发出 UI 或调用其他可组合函数的函数是必需的。它们可以调用常规函数和其他可组合函数。如果函数不满足这些要求，则不应使用 @Composable 对其进行注释。

注意 `MainActivity.kt` 中的可组合函数如何在 `MainActivity` 类之外，声明为顶级函数。您在 Activity 之外拥有的代码越多，您可以共享和重用的代码就越多。

首先，重构您的代码以使其更具可重用性，并创建一个新的 `@Composable MyApp` 函数，其中包含特定于此活动的 Compose UI 逻辑。

其次，将应用程序的背景颜色放在可重用的 `Greeting` Composable 中是没有意义的。该配置应该应用于此屏幕上的每个 UI 部分，因此将 Surface 从 Greeting 移动到新的 `MyApp` 函数：

``` kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}

@Composable
fun MyApp() {
    BasicsCodelabTheme {
        Surface(color = Color.Yellow) {
            Greeting(name = "Android")
        }
    }
}

@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!", modifier = Modifier.padding(24.dp))
}

@Preview
@Composable
fun DefaultPreview() {
    MyApp()
}
```

您希望在不同的 Activity 中重用 `MyApp` Composable 函数，因为它定义了可以在多个地方使用的顶级配置。但是，它的当前状态不允许它，因为它嵌入了 `Greeting`。继续阅读以了解如何创建一个包含常见应用配置的容器。

### 制作容器功能

如果您想创建一个包含应用程序所有常见配置的容器，该怎么办？

要创建通用容器，请创建一个 Composable 函数，该函数将返回 `Unit` 的 `Composable` 函数（此处称为内容）作为参数。您返回 `Unit` 是因为，您可能已经注意到，可组合函数不返回 UI 组件，而是发出它们。这就是为什么他们必须返回 `Unit` ：

``` kotlin
@Composable
fun MyApp(content: @Composable () -> Unit) {
    BasicsCodelabTheme {
        Surface(color = Color.Yellow) {
            content()
        }
    }
}
```

> 注意：使用 Composable 函数作为参数时，请注意 @Composable() 中的额外括号。由于注释应用于函数，因此需要它们！
>
> fun MyApp(content: @Composable () -> Unit) { ... }

在您的函数中，您定义了您希望容器提供的所有共享配置，然后调用传递的子项 Composable。在这种情况下，您要应用 `MaterialTheme` 和黄色表面，然后调用 `content()`。

您可以像这样使用它（利用 Kotlin 的[尾随 lambda ](https://kotlinlang.org/docs/reference/lambdas.html#passing-a-lambda-to-the-last-parameter)语法）：

``` kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp {
                Greeting("Android")
            }
        }
    }
}

@Composable
fun MyApp(content: @Composable () -> Unit) {
    BasicsCodelabTheme {
        Surface(color = Color.Yellow) {
            content()
        }
    }
}

@Preview("Text preview")
@Composable
fun DefaultPreview() {
    MyApp {
        Greeting("Android")
    }
}
```

此代码与您在上一节中的代码等效，但现在更加灵活。使容器可组合函数是一种很好的做法，可以提高可读性并鼓励重用代码。

### 使用布局多次调用可组合函数

您可以将 UI 组件提取到可组合函数中，以便您可以在不重复代码的情况下重用它们。在下面的示例中，您可以显示两个问候语，使用不同的参数重用相同的 Composable 函数。

要按垂直顺序放置项目，请使用 Column Composable 函数（类似于垂直 LinearLayout）。

``` kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.material.Divider
...

@Composable
fun MyScreenContent() {
    Column {
        Greeting("Android")
        Divider(color = Color.Black)
        Greeting("there")
    }
}
```

> 注意：分隔线是一个提供的可组合函数，用于创建水平分隔线。

由于您希望 `MyScreenContent` 成为您的用户在打开应用程序时看到的内容，因此您必须相应地修改 `MainActivity` 代码。此外，您可以修改预览代码，以便您可以在 Android Studio 中更快地迭代，而无需将应用程序部署到设备或模拟器。

如果刷新预览，您将看到垂直放置的项目：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628843131/148453907504be1f_mobenm.png)

> 注意：当 Composable 函数被调用时，它会将元素添加到 Compose UI 层次结构中。您可以从代码的多个部分调用相同的函数（可能具有不同的参数）以添加新元素。您可以将此视为通过调用可组合函数来发出 UI 元素。

### Compose 和 Kotlin

可以像 Kotlin 中的任何其他函数一样调用 Compose 函数。这使得构建 UI 非常强大，因为您可以添加语句来影响 UI 的显示方式。

例如，您可以使用 `for` 循环将元素添加到 `MyScreenContent` `Column`：

``` kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
    }
}
```

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628843154/9ec1ce770c25da83_x0sbmd.png)

## Compose状态

对状态变化做出反应是 Compose 的核心。 Compose 应用程序通过调用 Composable 函数将数据转换为 UI。如果您的数据发生变化，您可以使用新数据调用这些函数，从而创建更新的 UI。Compose 提供了用于观察应用数据变化的工具，这些工具会自动调用您的函数——这称为**重构/recomposing**。Compose 还会查看单个可组合组件需要哪些数据，以便它只需要重新组合数据已更改的组件，并且可以跳过组合未受影响的组件。

在幕后，Compose 使用自定义 Kotlin 编译器插件，因此当底层数据发生变化时，可以重新调用可组合函数以更新 UI 层次结构。

例如，当您在 `MyScreenContent` Composable 函数中调用 `Greeting("Android")` 时，您正在硬编码输入 ("Android")，所以 `Greeting` 将被添加到 UI 树中一次并且永远不会改变，即使 `MyScreenContent` 的主体被重新组合。

要将内部状态添加到可组合，请使用 `mutableStateOf` 函数，该函数提供可组合的可变内存。为了不让每次重组都具有不同的状态，请使用 `remember` 记住可变状态。而且，如果在屏幕上的不同位置有多个可组合实例，每个副本将获得自己的状态版本。您可以将内部状态视为类中的私有变量。而且，如果在屏幕上的不同位置有多个可组合实例，每个副本将获得自己的状态版本。您可以将内部状态视为类中的私有变量。

可组合函数将自动订阅它。如果状态发生变化，读取这些字段的可组合项将被重新组合。

制作一个计数器来记录用户点击`按钮`的次数。将 `Counter` 定义为一个 Composable 函数，并带有一个 `Button` 来显示它被点击了多少次：

> 注意：Compose 根据 [Material Design Buttons](https://material.io/develop/android/components/buttons/) 规范提供了不同类型的 Button——Button、OutlinedButton 和 TextButton。在您的情况下，您将使用一个具有文本的 Button 作为显示它被点击的次数的 Button 内容。

由于 `Button` 读取的是 `count.value`，因此每当 `Button` 发生变化时都会重新组合并显示新的 `count` 值。

您现在可以在屏幕上添加一个计数器：

``` kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter()
    }
}
```

如果您在模拟器中运行该应用程序或单击交互式预览按钮 ，您可以看到 Counter 如何保持状态并在每次单击时增加。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628923952/8766b5185e6476a5_jtcfba.gif)

### 真相的来源

在 Composable 函数中，应该公开对调用函数有用的状态，因为这是它可以被消费或控制的唯一方式——这个过程称为状态提升/**state hoisting**。

状态提升是使内部状态可由调用它的函数控制的方法。您可以通过受控制的可组合函数的参数公开状态并从控制可组合函数的外部实例化它来实现此目的。使状态可提升可避免重复状态和引入错误，有助于重用可组合物，并使可组合物更易于测试。可组合调用者不感兴趣的状态应该是内部状态。

在某些情况下，消费者可能不关心某个状态（例如，在滚动条中，`scrollerPosition` 状态是公开的，而 `maxPosition` 不是）。真相的来源属于创造和控制那个状态的人。

在示例中，由于 `Counter` 的使用者可能对状态感兴趣，因此您可以通过引入 (count, updateCount) 对作为 Counter 的参数将其完全推迟到调用者。这样，Counter 就是在提升它的状态：

``` kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    val counterState = remember { mutableStateOf(0) }

    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter(
            count = counterState.value,
            updateCount = { newCount ->
                counterState.value = newCount
            }
        )
    }
}

@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(onClick = { updateCount(count+1) }) {
        Text("I've been clicked $count times")
    }
}
```

## Flex布局

您之前简要介绍过 `Column`，它用于按垂直顺序放置项目。同样，您可以使用 `Row` 水平放置项目。

`Row`和`Column`将它们的项目一个接一个地放置。如果你想让一些项目变得灵活，让它们以一定的重量占据屏幕，你可以使用`weight`修饰符。

假设您想将 `Button` 放在屏幕底部，而其他内容保留在顶部。您可以通过以下步骤执行此操作：

1. 使用`weight`修饰符将柔性项目包裹在另一个 `Column` 中。由于这个 `Column` 是灵活的，其余的内容是不灵活的，它会尽可能多地占用空间。在调整外部 `Column` 的大小后，它将能够使用所有剩余的高度。
2. 将 `Counter` 留在默认情况下不灵活的外部 `Column` 中。

``` kotlin
import androidx.compose.foundation.layout.fillMaxHeight
...

@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    val counterState = remember { mutableStateOf(0) }

    Column(modifier = Modifier.fillMaxHeight()) {
        Column(modifier = Modifier.weight(1f)) {
            for (name in names) {
                Greeting(name = name)
                Divider(color = Color.Black)
            }
        }
        Counter(
            count = counterState.value,
            updateCount = { newCount ->
                counterState.value = newCount
            }
        )
    }
}
```

您可以在外部 Column 上使用 `fillMaxHeight()` 修饰符以使其占据尽可能多的屏幕（`fillMaxSize()` 和 `fillMaxWidth()` 修饰符也可用）。

1. 如果刷新预览，您可以看到新的更改：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628928206/eb089602c1c5d55e_foufyp.png)

作为在 Compose 中利用 Kotlin 的另一个示例，您可以根据用户使用 if...else 语句点击按钮的次数来更改按钮的背景颜色：

``` kotlin
import androidx.compose.material.ButtonDefaults
...

@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(
        onClick = { updateCount(count+1) },
        colors = ButtonDefaults.buttonColors(
            backgroundColor = if (count > 5) Color.Green else Color.White
        )
    ) {
        Text("I've been clicked $count times")
    }
}
```

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628928647/ce6ac33a852a4ee9_syzjwg.gif)

现在让我们使名单更加真实。到目前为止，您已经在一个 `Column` 中显示了两个项目，但它可以处理数千个项目吗？在更改列表项之前，让我们将名称列提取到专用的 `Composable`

``` kotlin
@Composable
fun NameList(names: List<String>, modifier: Modifier = Modifier) {
   Column(modifier = modifier) {
       for (name in names) {
           Greeting(name = name)
           Divider(color = Color.Black)
       }
   }
}
```

更改 `MyScreenContent` 参数中的默认列表值以使用另一个列表构造函数，该构造函数允许设置列表大小并使用其 lambda 中包含的值填充它（此处 `$it` 代表列表索引）：

``` kotlin
names: List<String> = List(1000) { "Hello Android #$it" }
```

无论您是在交互模式下呈现它还是在设备/模拟器上部署它，您都无法滚动这些数千行，因为 `Column` 默认情况下不可滚动。

要显示可滚动列，我们使用 `LazyColumn`。 `LazyColumn` 仅呈现屏幕上的可见项目，从而在呈现大列表时提高性能。相当于Android Views中的`RecyclerView`。

由于列表包含数千个项目，这会在呈现时影响应用程序的流动性，因此请使用 `LazyColumn` 仅呈现屏幕上的可见元素，而不是所有元素。

在其基本用法中，`LazyColumn` API 在其范围内提供了一个 `items` 元素，其中编写了单个 item 呈现逻辑：

``` kotlin
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
...

@Composable
fun NameList(names: List<String>, modifier: Modifier = Modifier) {
   LazyColumn(modifier = modifier) {
       items(items = names) { name ->
           Greeting(name = name)
           Divider(color = Color.Black)
       }
   }
}
```

> 注意： LazyColumn 不会像 RecyclerView 那样回收它的孩子。当您滚动浏览它时，它会发出新的可组合物，并且仍然具有性能，因为与实例化 Android 视图相比，发出可组合物的成本相对较低。

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628929988/7021b34729244ba8_mv1e5z.gif)

## 动画列表

到目前为止，用不到 100 行代码，您就能够显示一个长而高效的滚动列表，其中一个按钮在单击 5 次后会更改其背景颜色，太棒了！现在让我们使列表更具交互性。

假设您想在单击后更改列表项的背景颜色。您之前已经使用按钮完成了它，但是这一次，从一种背景颜色到另一种背景颜色的过渡将是动画的，而不是即时的，如下所示：

![](https://res.cloudinary.com/neroblackstone/image/upload/v1628930132/1a7b3467851eec7b_jzyxot.gif)

为此，您将使用 `animateColorAsState` API，但首先您需要更新 `Greeting` Composable 以添加 `isSelected` 状态（使用`remember`为 false 进行初始化）和单击处理程序，以切换该状态：

``` kotlin
import androidx.compose.animation.animateColorAsState
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
...

@Composable
fun Greeting(name: String) {
   var isSelected by remember { mutableStateOf(false) }
   val backgroundColor by animateColorAsState(if (isSelected) Color.Red else Color.Transparent)

   Text(
       text = "Hello $name!",
       modifier = Modifier
           .padding(24.dp)
           .background(color = backgroundColor)
           .clickable(onClick = { isSelected = !isSelected })
   )
}
```

> Note: Make sure these imports are included in your file otherwise delegate properties (the by keyword) won't work:
>
>import androidx.compose.runtime.getValue
>
>import androidx.compose.runtime.setValue

在这个特定用例中，`animateColorAsState` 将颜色作为参数，将其保存并自动生成显示从先前设置的颜色到新颜色的动画过渡所需的中间颜色。

``` kotlin
val backgroundColor by animateColorAsState(if (isSelected) Color.Red else Color.Transparent)
```

您现在可以通过在可组合文本上设置`background`修饰符来添加背景的动画变化：

> 注意：由于 isSelected 状态在 Greeting 可组合中被提升，NameList 将不会跟踪其项目是否被选中。一旦项目滚动出屏幕，它们的状态将被设置为 false。该行为的目的是因为本练习的目标是保留一个简单的列表。为了跟踪列表中选定的项目，它们的 isSelected 状态应该在 NameList 级别提升。