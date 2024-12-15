
![](https://img2024.cnblogs.com/blog/468667/202412/468667-20241214045754962-196194616.gif)
 


【引言】


本文将介绍一个名为“颜文字搜索器”的开发案例，该应用是基于鸿蒙NEXT平台构建的，旨在帮助用户快速查找和使用各种风格的表情符号。通过本案例的学习，读者可以了解如何在鸿蒙平台上进行数据处理、UI设计以及交互逻辑的实现。


【环境准备】


• 操作系统：Windows 10


• 开发工具：DevEco Studio NEXT Beta1 Build Version: 5\.0\.3\.806


• 目标设备：华为Mate60 Pro


• 开发语言：ArkTS


• 框架：ArkUI


• API版本：API 12


【开发思路】


1\. 数据模型设计


为了表示单个表情符号的信息，我们定义了一个 EmoticonBean 类，它包含了表情符号的风格（style）、类型（type）、表情符号本身（emoticon）及其含义（meaning）。此外，还添加了一个布尔属性 isShown 来追踪表情符号是否应该显示给用户，这有助于在搜索时动态更新列表。


2\. UI 组件与布局


应用的主界面由一个 Index 组件构成，它负责整体布局的设计。界面上部是一个搜索框，允许用户输入关键词来过滤表情符号列表。下方则以表格形式展示了所有符合条件的表情符号，每一行包括四个部分：风格、类型、表情符号和含义。为了提升用户体验，当用户点击某个表情符号或其含义中的高亮文本时，会触发相应的点击事件，并输出日志信息。


3\. 数据加载与处理


表情符号的数据来源于一个本地 JSON 文件 (emoticons.json)，该文件在组件初次渲染之前被读取并解析为 EmoticonBean 对象数组。每次用户修改搜索框中的内容时，都会调用 splitAndHighlight 方法对每个表情符号的含义进行分割，并检查是否存在匹配的关键字。如果存在，则设置 isShown 属性为 true，否则为 false，以此控制表情符号是否在界面上显示。


4\. 搜索与高亮


splitAndHighlight 函数用于将表情符号的含义按关键字分割成多个片段，并返回这些片段组成的数组。对于包含关键字的片段，会在界面上以不同的颜色高亮显示，从而直观地指出匹配的部分。此外，此函数还会根据是否有匹配项来决定表情符号是否可见，确保只有相关的表情符号才会展示给用户。


5\. 用户交互


为了让用户有更好的操作体验，我们在界面上实现了触摸事件监听，当用户点击非输入区域时，自动关闭键盘。这样既保证了界面整洁，又简化了用户的操作流程。


【完整代码】


数据源：src/main/resources/rawfile/emoticons.json


[https://download.csdn.net/download/zhongcongxu01/90126325](https://download.csdn.net/download/zhongcongxu01/90126325 "https://download.csdn.net/download/zhongcongxu01/90126325")


代码



[?](https://github.com):[MeoMiao 萌喵加速](https://biqumo.org)

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232 | `// 引入必要的工具库 util 用于文本解码等操作``import` `{ util } from` `'@kit.ArkTS'``// 引入 BusinessError 类用于处理业务逻辑错误``import` `{ BusinessError } from` `'@kit.BasicServicesKit'``// 引入 inputMethod 模块用于管理输入法行为``import` `{ inputMethod } from` `'@kit.IMEKit'` `// 定义一个可以被观察的数据模型 EmoticonBean 表示单个表情符号的信息``@ObservedV2``class` `EmoticonBean {``// 定义风格属性，并初始化为空字符串``style: string =` `""``// 定义类型属性，并初始化为空字符串``type: string =` `""``// 定义表情符号本身，并初始化为空字符串``emoticon: string =` `""``// 定义含义属性，并初始化为空字符串``meaning: string =` `""` `// 构造函数，允许在创建对象时设置上述属性``constructor(style: string, type: string, emoticon: string, meaning: string) {``this``.style = style``this``.type = type``this``.emoticon = emoticon``this``.meaning = meaning``}` `// 定义是否显示的表情符号状态标记，默认为 true，使用 @Trace 装饰器使其可追踪变化``@Trace isShown: boolean =` `true``}` `// 使用 @Entry 和 @Component 装饰器定义 Index 组件作为应用入口``@Entry``@Component``struct Index {``// 定义一个状态变量 textInput 用于存储搜索框中的文本内容，默认为空字符串``@State` `private` `textInput: string =` `''``// 定义一个状态变量 emoticonList 用于存储表情符号列表，默认为空数组``@State` `private` `emoticonList: EmoticonBean[] = []` `// 定义线条颜色属性``private` `lineColor: string =` `"#e6e6e6"``// 定义标题背景色属性``private` `titleBackground: string =` `"#f8f8f8"``// 定义文本颜色属性``private` `textColor: string =` `"#333333"``// 定义基础填充大小``private` `basePadding: number = 4``// 定义线条宽度``private` `lineWidth: number = 2``// 定义单元格高度``private` `cellHeight: number = 50``// 定义列权重比例``private` `weightRatio: number[] = [1, 1, 5, 4]``// 定义基础字体大小``private` `baseFontSize: number = 14` `// 定义一个方法 splitAndHighlight 用于分割并高亮表情符号含义中的关键词``private` `splitAndHighlight(item: EmoticonBean, keyword: string): string[] {``let` `text = item.meaning` `// 获取表情符号的含义文本``if` `(!keyword) {` `// 如果没有关键词，则直接返回整个文本，并显示该表情符号``item.isShown =` `true``return` `[text]``}``let` `segments: string[] = [];` `// 用于存储分割后的文本片段``let` `lastMatchEnd: number = 0;` `// 记录上一次匹配结束的位置``while` `(``true``) {` `// 循环查找关键词在文本中的位置``const matchIndex = text.indexOf(keyword, lastMatchEnd);` `// 查找关键词出现的位置``if` `(matchIndex === -1) {` `// 如果找不到关键词，将剩余文本加入到segments中并退出循环``segments.push(text.slice(lastMatchEnd));``break``;``}` `else` `{` `// 如果找到关键词，将非关键词部分和关键词部分分别加入到segments中``segments.push(text.slice(lastMatchEnd, matchIndex));` `// 非关键词部分``segments.push(text.slice(matchIndex, matchIndex + keyword.length));` `// 关键词部分``lastMatchEnd = matchIndex + keyword.length;``}``}``// 如果有关键词出现，则设置表情符号为显示状态``item.isShown = (segments.indexOf(keyword) != -1)``return` `segments;``}` `// 当组件即将出现在屏幕上时调用此方法，用于加载表情符号数据``aboutToAppear() {``// 从资源管理器中读取本地文件 emoticons.json 的内容``getContext().resourceManager.getRawFileContent(``"emoticons.json"``, (err: BusinessError, data) => {``if` `(err) {` `// 如果读取失败，打印错误信息``console.error(``'getRawFileContent error: '` `+ JSON.stringify(err))``return``}``// 创建一个文本解码器来将二进制数据转换为字符串``let` `textDecoder = util.TextDecoder.create(``'utf-8'``, { ignoreBOM:` `true` `})``let` `jsonString = textDecoder.decodeToString(data, { stream:` `false` `})``let` `jsonObjectArray: object[] = JSON.parse(jsonString)` `// 将 JSON 字符串解析为对象数组``for` `(``let` `i = 0; i < jsonObjectArray.length; i++) {` `// 遍历对象数组，填充 emoticonList``let` `item = jsonObjectArray[i]``this``.emoticonList.push(``new` `EmoticonBean(item[``'s'``], item[``'t'``], item[``'e'``], item[``'m'``]))``}``try` `{``// 打印 emoticonList 到控制台以供调试``console.info(```this``.emoticonList:${JSON.stringify(``this``.emoticonList,` `null``,` `'\u00a0\u00a0'``)}`)``}` `catch` `(err) {``console.error(``'parse error: '` `+ JSON.stringify(err))``}``})``}` `// 定义 build 方法构建组件的UI结构``build() {``Column({ space: 0 }) {` `// 创建一个列容器，内部元素之间没有间距``// 搜索框组件，绑定到 textInput 状态变量``Search({ value: $$``this``.textInput })``.margin(``this``.basePadding)` `// 设置外边距``.fontFeature(``"\"ss01\" on"``)` `// 设置字体特征``// 创建一个列容器用于表头``Column() {``Row() {` `// 创建一行用于放置表头``// 表头文字：风格``Text(``'风格'``)``.height(``'100%'``)` `// 设置高度为父容器的100%``.layoutWeight(``this``.weightRatio[0])` `// 根据权重分配宽度``.textAlign(TextAlign.Center)` `// 文本居中对齐``.fontSize(``this``.baseFontSize)` `// 设置字体大小``.fontWeight(600)` `// 设置字体粗细``.fontColor(``this``.textColor)` `// 设置文本颜色``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 表头文字：类型``Text(``'类型'``)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[1])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontWeight(600)``.fontColor(``this``.textColor)``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 表头文字：表情``Text(``'表情'``)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[2])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontWeight(600)``.fontColor(``this``.textColor)``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 表头文字：含义``Text(``'含义'``)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[3])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontWeight(600)``.fontColor(``this``.textColor)``}.height(``this``.cellHeight).borderWidth(``this``.lineWidth).borderColor(``this``.lineColor)``.backgroundColor(``this``.titleBackground)` `// 设置背景颜色``}.width(`100%`).padding({ left:` `this``.basePadding, right:` `this``.basePadding })` `// 创建一个滚动容器 Scroll 包含表情符号列表``Scroll() {``Column() {``// ForEach 循环遍历 emoticonList 数组，创建每一行代表一个表情符号条目``ForEach(``this``.emoticonList, (item: EmoticonBean) => {``Row() {``// 显示表情符号的风格``Text(item.style)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[0])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontColor(``this``.textColor)``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 显示表情符号的类型``Text(item.type)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[1])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontColor(``this``.textColor)``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 显示表情符号``Text(item.emoticon)``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[2])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontColor(``this``.textColor)``.copyOption(CopyOptions.LocalDevice)` `// 允许复制到剪贴板``// 分割线``Line().height(``'100%'``).width(``this``.lineWidth).backgroundColor(``this``.lineColor)``// 显示表情符号的含义，支持关键字高亮``Text() {``ForEach(``this``.splitAndHighlight(item,` `this``.textInput), (segment: string, index: number) => {``ContainerSpan() {``Span(segment)``.fontColor(segment ===` `this``.textInput ? Color.White : Color.Black)` `// 根据是否是关键词设置字体颜色``.onClick(() => {` `// 设置点击事件监听器``console.info(`Highlighted text clicked: ${segment}`);` `// 打印点击的文本信息``console.info(`Click index: ${index}`);` `// 打印点击的索引信息``});``}.textBackgroundStyle({``color: segment ===` `this``.textInput ? Color.Red : Color.Transparent` `// 根据是否是关键词设置背景颜色``});``});``}``.height(``'100%'``)``.layoutWeight(``this``.weightRatio[3])``.textAlign(TextAlign.Center)``.fontSize(``this``.baseFontSize)``.fontColor(``this``.textColor)``.padding({ left:` `this``.basePadding, right:` `this``.basePadding })``}``.height(``this``.cellHeight)``.borderWidth({ left:` `this``.lineWidth, right:` `this``.lineWidth, bottom:` `this``.lineWidth })``.borderColor(``this``.lineColor)``// 根据表情符号的状态（是否显示）来决定其可见性``.visibility(item.isShown ? Visibility.Visible : Visibility.None)``})``}.width(`100%`).padding({ left:` `this``.basePadding, right:` `this``.basePadding })``}.width(``'100%'``).layoutWeight(1).align(Alignment.Top)``// 触摸事件处理，当用户点击空白区域时，关闭键盘输入``.onTouch((event) => {``if` `(event.type == TouchType.Down) {` `// 如果是按下事件``inputMethod.getController().stopInputSession()` `// 停止当前的输入会话``}``})``}.width(``'100%'``).height(``'100%'``).backgroundColor(Color.White);` `// 设置容器的宽高和背景颜色``}``}` |
| --- | --- |



　　


