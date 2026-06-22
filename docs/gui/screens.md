# 屏幕

屏幕通常是 Minecraft 中所有图形用户界面（GUI）的基础：接受用户输入、在服务器上验证，并将结果操作同步回客户端。它们可以与[菜单][menus]结合，为类似物品栏的视图创建通信网络，也可以是独立的，模组开发者可以通过自己的[网络][network]实现来处理。

屏幕由许多部分组成，因此很难完全理解 Minecraft 中"屏幕"的实际含义。因此，本文档将在讨论屏幕本身之前，先介绍屏幕的每个组成部分及其应用方式。

## 相对坐标

每当渲染任何内容时，都需要有一个标识符来指定它将在哪里出现。通过众多的抽象层，Minecraft 的大多数渲染调用都接受坐标平面中的 x、y 和 z 值。X 值从左到右递增，Y 值从上到下递增，Z 值从远到近递增。然而，坐标并非固定在一个指定的范围内。它们可以根据屏幕的大小和在选项中指定的缩放比例而变化。因此，必须格外小心，以确保渲染坐标的值能够适当地缩放到可变的屏幕大小。

关于如何相对化坐标的信息将在[屏幕][screen]部分中介绍。

!!! important
    如果你选择使用固定坐标或不正确地缩放屏幕，渲染的对象可能会看起来很奇怪或位置错误。检查坐标相对化是否正确的一个简单方法是点击视频设置中的"GUI 缩放"按钮。在确定 GUI 应渲染的缩放比例时，该值用作显示宽度和高度的除数。

## GuiGraphics

Minecraft 渲染的任何 GUI 通常都使用 `GuiGraphics` 完成。`GuiGraphics` 是几乎所有渲染方法的第一个参数；它包含渲染常用对象的基本方法。这些方法分为五类：彩色矩形、字符串、纹理、物品和提示框。还有一个用于渲染组件片段的方法（`#enableScissor` / `#disableScissor`）。`GuiGraphics` 还公开了 `PoseStack`，用于应用必要的变换以在正确的位置渲染组件。此外，颜色采用 [ARGB][argb] 格式。

### 彩色矩形

彩色矩形通过位置颜色着色器绘制。可以绘制三种类型的彩色矩形。

首先是彩色水平线和垂直线（宽度为 1 像素），分别是 `#hLine` 和 `#vLine`。`#hLine` 接受两个 x 坐标（定义左右边界，包含）、顶部的 y 坐标和颜色。`#vLine` 接受左侧 x 坐标、两个 y 坐标（定义上下边界，包含）和颜色。

其次是 `#fill` 方法，它绘制一个矩形到屏幕上。线条方法内部调用此方法。它接受左侧 x 坐标、顶部 y 坐标、右侧 x 坐标、底部 y 坐标和颜色。

最后是 `#fillGradient` 方法，它绘制一个带有垂直渐变的矩形。它接受右侧 x 坐标、底部 y 坐标、左侧 x 坐标、顶部 y 坐标、z 坐标以及底部和顶部的颜色。

### 字符串

字符串通过其 `Font` 绘制，通常使用它们自己的着色器来支持普通、透视和偏移模式。可以渲染两种对齐方式的字符串，每种都有背景阴影：左对齐字符串（`#drawString`）和居中对齐字符串（`#drawCenteredString`）。两者都接受渲染字符串所用的字体、要绘制的字符串、分别表示字符串左侧或中心的 x 坐标、顶部 y 坐标和颜色。

!!! note
    字符串通常应以 [`Component`][component] 的形式传入，因为它们可以处理各种使用场景，包括该方法的其他两个重载。

### 纹理

纹理通过 blitting（位块传输）绘制，因此方法名为 `#blit`，它复制图像的位并直接绘制到屏幕上。这些通过位置纹理着色器绘制。虽然有许多不同的 `#blit` 重载，但我们只讨论两个静态的 `#blit`。

第一个静态 `#blit` 接受六个整数，并假设正在渲染的纹理位于 256 x 256 的 PNG 文件中。它接受屏幕上的左侧 x 和顶部 y 坐标、PNG 内的左侧 x 和顶部 y 坐标，以及要渲染的图像的宽度和高度。

!!! note
    必须指定 PNG 文件的大小，以便归一化坐标以获取相关的 UV 值。

第一个调用的静态 `#blit` 将其扩展为九个整数，仅假设图像位于 PNG 文件中。它接受屏幕上的左侧 x 和顶部 y 坐标、z 坐标（称为 blit 偏移）、PNG 内的左侧 x 和顶部 y 坐标、要渲染的图像的宽度和高度，以及 PNG 文件的宽度和高度。

#### Blit 偏移

渲染纹理时的 z 坐标通常设置为 blit 偏移。该偏移负责在查看屏幕时正确地对渲染进行分层。z 坐标较小的渲染在背景中渲染，反之，z 坐标较大的渲染在前景中渲染。可以通过 `PoseStack` 上的 `#translate` 直接设置 z 偏移。`GuiGraphics` 的某些方法内部应用了一些基本的偏移逻辑（例如物品渲染）。

!!! important
    设置 blit 偏移后，必须在渲染对象后重置它。否则，屏幕中的其他对象可能会在不正确的层中渲染，导致图形问题。建议在平移之前推送当前 pose，然后在偏移处完成所有渲染后弹出。

## Renderable

`Renderable` 本质上是可渲染的对象。这些包括屏幕、按钮、聊天框、列表等。`Renderable` 只有一个方法：`#render`。它接受用于将内容渲染到屏幕的 `GuiGraphics`、缩放到相对屏幕大小的鼠标 x 和 y 位置，以及 tick 增量（自上一帧以来经过的 tick 数）。

一些常见的可渲染对象是屏幕和"小部件"：通常在屏幕上渲染的可交互元素，例如 `Button`、其子类型 `ImageButton`，以及用于在屏幕上输入文本的 `EditBox`。

## GuiEventListener

Minecraft 中渲染的任何屏幕都实现了 `GuiEventListener`。`GuiEventListener` 负责处理用户与屏幕的交互。这些包括来自鼠标（移动、点击、释放、拖动、滚动、悬停）和键盘（按下、释放、输入）的输入。每个方法返回关联操作是否成功影响了屏幕。按钮、聊天框、列表等小部件也实现了此接口。

### ContainerEventHandler

与 `GuiEventListener` 几乎同义的是其子类型：`ContainerEventHandler`。它们负责处理包含小部件的屏幕上的用户交互，管理当前聚焦的元素以及关联的交互如何应用。`ContainerEventHandler` 添加了三个额外功能：可交互子元素、拖拽和聚焦。

事件处理器持有子元素，用于确定元素的交互顺序。在鼠标事件处理器（不包括拖拽）期间，列表中鼠标悬停的第一个子元素会执行其逻辑。

使用鼠标拖拽元素，通过 `#mouseClicked` 和 `#mouseReleased` 实现，提供更精确的执行逻辑。

聚焦允许在事件执行期间（例如在键盘事件或鼠标拖拽期间）首先检查并处理特定的子元素。焦点通常通过 `#setFocused` 设置。此外，可以使用 `#nextFocusPath` 循环切换可交互子元素，根据传入的 `FocusNavigationEvent` 选择子元素。

!!! note
    屏幕通过 `AbstractContainerEventHandler` 实现 `ContainerEventHandler`，它添加了拖拽和聚焦子元素的 setter 和 getter 逻辑。

## NarratableEntry

`NarratableEntry` 是可以通过 Minecraft 的无障碍朗读功能进行语音描述的元素。每个元素可以根据悬停或选择的内容提供不同的叙述，通常按焦点、悬停，然后所有其他情况的优先级排序。

`NarratableEntry` 有三个方法：一个确定元素的优先级（`#narrationPriority`），一个确定是否朗读叙述（`#isActive`），最后一个提供叙述给关联的输出（语音或文字）（`#updateNarration`）。

!!! note
    Minecraft 中的所有小部件都是 `NarratableEntry`，因此在使用可用的子类型时通常不需要手动实现。

## 屏幕子类型

有了以上所有知识，就可以构建一个基本的屏幕了。为了更容易理解，屏幕的组件将按通常遇到的顺序说明。

首先，所有屏幕都接受一个表示屏幕标题的 `Component`。此组件通常由其子类型之一绘制到屏幕上。在基本屏幕中，它仅用于叙述消息。

```java
// 在某些 Screen 子类中
public MyScreen(Component title) {
    super(title);
}
```

### 初始化

屏幕初始化后，会调用 `#init` 方法。`#init` 方法设置屏幕内的初始设置，从 `ItemRenderer` 和 `Minecraft` 实例到经过游戏缩放的相对宽度和高度。添加小部件或预计算相对坐标等任何设置都应在此方法中完成。如果游戏窗口调整大小，将通过调用 `#init` 方法重新初始化屏幕。

有三种方式向屏幕添加小部件，每种服务于不同的目的：

方法                 | 描述
:---:                  | :---
`#addWidget`           | 添加可交互和可叙述但不可渲染的小部件。
`#addRenderableOnly`   | 添加仅渲染的小部件；不可交互也不可叙述。
`#addRenderableWidget` | 添加可交互、可叙述且可渲染的小部件。

通常，`#addRenderableWidget` 会是最常用的。

```java
// 在某些 Screen 子类中
@Override
protected void init() {
    super.init();
    // 添加小部件和预计算的值
    this.addRenderableWidget(new EditBox(/* ... */));
}
```

### 屏幕 Tick

屏幕也使用 `#tick` 方法来执行一定程度的客户端逻辑以用于渲染。最常见的例子是 `EditBox` 的光标闪烁。

```java
// 在某些 Screen 子类中
@Override
public void tick() {
    super.tick();
    this.editBox.tick();
}
```

### 输入处理

由于屏幕是 `GuiEventListener` 的子类型，也可以重写输入处理器，例如处理特定[按键][keymapping]的逻辑。

### 渲染屏幕

最后，屏幕通过作为 `Renderable` 子类型提供的 `#render` 方法进行渲染。如前所述，`#render` 方法每帧绘制屏幕需要渲染的所有内容，例如背景、小部件、提示框等。默认情况下，`#render` 方法仅将小部件渲染到屏幕。

屏幕内通常不由子类型处理的两种最常见渲染是背景和提示框。

背景可以使用 `#renderBackground` 渲染，其中一个方法在屏幕渲染时（当其后的世界无法显示时）接受一个 v 偏移用于选项背景。

提示框通过 `GuiGraphics#renderTooltip` 或 `GuiGraphics#renderComponentTooltip` 渲染，可以接受要渲染的文本组件、可选的自定义提示框组件以及提示框在屏幕上应渲染的 x/y 相对坐标。

```java
// 在某些 Screen 子类中

// mouseX 和 mouseY 表示光标在屏幕上的缩放坐标
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    // 通常首先渲染背景
    this.renderBackground(graphics);
    // 在此处渲染小部件之前的内容（背景纹理）
    // 然后渲染小部件（如果这是 Screen 的直接子类）
    super.render(graphics, mouseX, mouseY, partialTick);
    // 在小部件之后渲染内容（提示框）
}
```

### 关闭屏幕

关闭屏幕时，有两个方法处理清理工作：`#onClose` 和 `#removed`。

`#onClose` 在用户输入关闭当前屏幕时调用。此方法通常用作回调，用于销毁和保存屏幕本身的任何内部进程。这包括向服务器发送数据包。

`#removed` 在屏幕即将更改并释放到垃圾回收器时调用。它处理任何尚未重置为屏幕打开前初始状态的内容。

```java
// 在某些 Screen 子类中

@Override
public void onClose() {
    // 在此处停止任何处理器
    super.onClose();
}

@Override
public void removed() {
    // 在此处重置初始状态
    super.removed();
}
```

## `AbstractContainerScreen`

如果屏幕直接附加到[菜单][menus]，则应继承 `AbstractContainerScreen`。`AbstractContainerScreen` 充当菜单的渲染器和输入处理器，并包含用于与槽位同步和交互的逻辑。因此，通常只需要重写或实现两个方法即可拥有一个可工作的容器屏幕。同样，为了更容易理解，容器屏幕的组件将按通常遇到的顺序说明。

`AbstractContainerScreen` 通常需要三个参数：正在打开的容器菜单（由泛型 `T` 表示）、玩家物品栏（仅用于显示名称）和屏幕本身的标题。在此处，可以设置一些定位字段：

字段               | 描述
:---:             | :---
`imageWidth`      | 用于背景的纹理宽度。通常位于 256 x 256 的 PNG 内部，默认为 176。
`imageHeight`     | 用于背景的纹理高度。通常位于 256 x 256 的 PNG 内部，默认为 166。
`titleLabelX`     | 屏幕标题渲染的相对 x 坐标。
`titleLabelY`     | 屏幕标题渲染的相对 y 坐标。
`inventoryLabelX` | 玩家物品栏名称渲染的相对 x 坐标。
`inventoryLabelY` | 玩家物品栏名称渲染的相对 y 坐标。

!!! important
    在前面的部分中提到，预计算的相对坐标应在 `#init` 方法中设置。这仍然成立，因为此处提到的值不是预计算的坐标，而是静态值和相对化坐标。

    image 值是静态且不变的，因为它们代表背景纹理大小。为了简化渲染，`#init` 方法预计算了两个额外值（`leftPos` 和 `topPos`），标记了背景渲染的左上角位置。标签坐标相对于这些值。

    `leftPos` 和 `topPos` 也可以作为渲染背景的便利方式，因为它们已经代表了传递给 `#blit` 方法的位置。

```java
// 在某些 AbstractContainerScreen 子类中
public MyContainerScreen(MyMenu menu, Inventory playerInventory, Component title) {
    super(menu, playerInventory, title);

    this.titleLabelX = 10;
    this.inventoryLabelX = 10;
}
```

### 菜单访问

由于菜单被传入屏幕，菜单内已同步的任何值（通过槽位、数据槽位或自定义系统）现在都可以通过 `menu` 字段访问。

### 容器 Tick

当玩家存活并正在查看屏幕时，容器屏幕在 `#tick` 方法中通过 `#containerTick` 进行 tick。这实际上取代了容器屏幕中的 `#tick`，其最常见的用途是对配方书进行 tick。

```java
// 在某些 AbstractContainerScreen 子类中
@Override
protected void containerTick() {
    super.containerTick();
    // 在此处进行 tick
}
```

### 渲染容器屏幕

容器屏幕通过三个方法进行渲染：`#renderBg`（渲染背景纹理）、`#renderLabels`（在背景之上渲染文本）和 `#render`（包含了前两个方法，并额外提供了灰色背景和提示框）。

从 `#render` 开始，最常见的重写（通常也是唯一的情况）添加背景、调用 super 渲染容器屏幕，最后在其上渲染提示框。

```java
// 在某些 AbstractContainerScreen 子类中
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    this.renderBackground(graphics);
    super.render(graphics, mouseX, mouseY, partialTick);
    this.renderTooltip(graphics, mouseX, mouseY);
}
```

在 super 内部，调用 `#renderBg` 来渲染屏幕背景。最标准的表示使用三个方法调用：两个用于设置，一个用于绘制背景纹理。

```java
// 在某些 AbstractContainerScreen 子类中

private static final ResourceLocation BACKGROUND_LOCATION = new ResourceLocation(MOD_ID, "textures/gui/container/my_container_screen.png");

@Override
protected void renderBg(GuiGraphics graphics, float partialTick, int mouseX, int mouseY) {
    graphics.blit(BACKGROUND_LOCATION, this.leftPos, this.topPos, 0, 0, this.imageWidth, this.imageHeight);
}
```

最后，调用 `#renderLabels` 在背景之上、提示框之下渲染文本。

```java
// 在某些 AbstractContainerScreen 子类中
@Override
protected void renderLabels(GuiGraphics graphics, int mouseX, int mouseY) {
    super.renderLabels(graphics, mouseX, mouseY);
    graphics.drawString(this.font, this.label, this.labelX, this.labelY, 0x404040);
}
```

!!! note
    渲染标签时，你**不需要**指定 `leftPos` 和 `topPos` 偏移。这些已经在 `PoseStack` 中进行了平移，因此此方法内的所有内容都是相对于这些坐标绘制的。

## 注册 AbstractContainerScreen

要将 `AbstractContainerScreen` 与菜单一起使用，需要注册它。这可以通过在 `FMLClientSetupEvent`（在[**模组事件总线**][modbus]上）中调用 `MenuScreens#register` 来完成。

```java
// 事件在模组事件总线上监听
private void clientSetup(FMLClientSetupEvent event) {
    event.enqueueWork(
        // 假设 RegistryObject<MenuType<MyMenu>> MY_MENU
        // 假设 MyContainerScreen<MyMenu> 接受三个参数
        () -> MenuScreens.register(MY_MENU.get(), MyContainerScreen::new)
    );
}
```

!!! warning
    `MenuScreens#register` 不是线程安全的，因此需要在并行调度事件提供的 `#enqueueWork` 内部调用。

[menus]: ./menus.md
[network]: ../networking/index.md
[screen]: #屏幕子类型
[argb]: https://en.wikipedia.org/wiki/RGBA_color_model#ARGB32
[component]: ../concepts/internationalization.md#translatablecontents
[keymapping]: ../misc/keymappings.md#在-gui-中
[modbus]: ../concepts/events.md#模组事件总线
