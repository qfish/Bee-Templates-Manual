Bee-Templates-Manual
====================

[toc]

# Preface
## Why BeeFramework
1. View
> Bee可以用xml进行相对布局，意味你可以像写HTML一样相对布局，但比HTML更简单
>
> Bee可以像Javascript一样获取XML一个按钮的点击事件，而你只需一行代码
>
> Bee同样可以兼容你自定义的控件

2. Model
3. Controller

## Why Bee Templates

话说搞过iOS的都知道frame是怎么回事，算坐标这事总能让你疼到极致~   

即使给`UIView`扩展那么几个`top`，`left`，`width`,`height`，... 属性,你依然摆脱不了`view.frame = CGrectMake(x,x,x,x);`的折磨。

你在`superview`上`add`了一些`subview`，你不告诉这些小朋友们该站在哪里，他们就全部站在教室的一个角落(origin(0,0))，并且是一个小朋友踩在另外一个小朋友上面..。

于是乎，问题就来了，为什么view和他的小伙伴们不能一个挨着一个，自觉的排列呢~

Bee Template 其中的一个特性解决了这问题，为什么说是其中的一个特性呢? 因为Bee Template 还解决了通过`XML布局(解决View层)`随后之而来的问题，比如如何处理控件的事件，如何给view填充数据等等。

## 谈谈CSS和HTML

这里简单说下，CSS和HTML

HTML是一种标签语言，是XML的一个子集，是一种简单数据表示方式。与XML不同的是，它的标签能被浏览器识别，然后浏览器针对不同的标签，赋予其不同的内置的样式，当然，不同的标签的固有属性和作用也不一样。有了这些基本的标签还不够，他们只能把数据展示出来，如果没有样式，你往往看到的就是黑色的字和白色的背景，这也是为什么有的网页偶尔会出现这样，因为CSS没有加载出来. 那CSS的作用就很明显了，规定一个元素的样式. 这些样式分很多种: 布局，背景，文字，边框等等，更多请参考 [CSS Reference](http://www.w3.org/TR/CSS/). 

顺便说一下HTML和CSS的规范,其中有一个叫`语义化`，所谓的`语义化`，就是指每个标签的作用更贴近标签本身的所代表的含义。语义化之后会提到的就是`标签与样式分离`。这样做的目的很明显，`数据与展示分离`。所以不推荐，在一个标签内，显式的指定太多的样式，这样做的结果只有一个，不便于维护，而推荐各司其职，样式的部分还是留给CSS来做。

## Bee Template 原理
Bee Template里默认的标签对应的是cocoa里的控件(当然你也可以自定义)，所以当你写了一个标签的时候，程序运行到某个Board的时候，Bee会做下面几件事情：
1. 在 `BeeUIBoard.CREATE_VIEWS` 时加载XML ( `view.FROM_RESOURCE()` ),这一步主要是`alloc,init.addSubview`；
2. 在 `BeeUIBoard.LAYOUT_VIEWS` 时触发自动布局( `view.RELAYOUT()` )，这一步会根据你在`CSS`或者`标签`内指定的`大小，位置`等布局属性，经过计算之后自动`setFrame`；

当然，这些也都是可以手动调用的，只需要把自动布局关闭就行了(下面有介绍)。

# 搭建基于Bee Template的工程
## 1. 添加 BeeFramework
1. [Download SDK][download-sdk] (推荐)
    
    该包为BeeFramework的SDK安装包，其中包含了Xcode的Templates，请下载后解压并安装。

2. [From Git or Cocopods][download-bee]

## 2. 基于SDK创建工程
1. 新建一个空的工程
    
    ![new-preoject][new-preoject]

2. 新建一个Board (这里名字为Template)，Board(ViewController)属于View，点击next，勾选`With XML`选项。
    
    ![new-file-board][new-file-board]    

3. 将`TemplateBoard`显示出来
    
    在`AppBoard`的`ON_SIGNAL2( BeeUIBoard, signal )`里添加如下代码：
    
    ```
    if ( [signal is:BeeUIBoard.CREATE_VIEWS] )
    {
        TemplateBoard_iPhone * board = [TemplateBoard_iPhone board];
        board.view.backgroundColor = [UIColor whiteColor];
        [self.view addSubview:board.view];
    }
    ```
    
    然后`Run`一下，如果是白色的界面，那说明这一步成功了。

## 3. 试着编辑一下XML

打开`TemplateBoard_iPhone.xml`，改成下面这样，然后`Run`一下，你会看到一个黑色的小块在背景为白色的模拟器中间。

```
<?xml version="1.0" encoding="UTF-8"?>
<ui namespace="Template_Board_iPhone">
    <linear orientation="v" class="wrapper">
        <view id="view"/>
    </linear>
    <style type="text/css">
        #wrapper {
            width: 100%;
            height: 100%;
        }
        #view {
            background-color: #333;
            width: 100px;
            height: 100px;
            float: center;
            v-float: center;
        }
    </style>
</ui>
```
    
## 4. 如何加载布局资源
### 4.1 自动加载
上面的例子就是自动加载的布局资源，在`TemplateBoard_iPhone.m`里，会发现有两行宏:

    SUPPORT_AUTOMATIC_LAYOUT( YES ) // 支持自动布局
    SUPPORT_RESOURCE_LOADING( YES ) // 支持自动加载布局资源

1. `SUPPORT_AUTOMATIC_LAYOUT`作用是来控制`BeeUIBoard`或者`BeeUICell`是否支持自动布局，默认是如果NO，则不支持；如果YES，该view会在创建时自动加载对应的XML。

2. `SUPPORT_RESOURCE_LOADING`作用是来控制是否自动加载资源，<b class="blue">加载的资源名就是类名</b>，比如类名是`TemplateBoard_iPhone`，那Bee会加载`TemplateBoard_iPhone.xml`的资源文件。

### 4.2 手动加载

```
// in board or view
self.FROM_RESOURCE( @”XXX.xml”);
self.RELAYOUT();

// board
someBoard.FROM_RESOURCE( @”YYY.xml”);
someBoard.RELAYOUT();

// view
someView.FROM_RESOURCE( @”YYY.xml”);
someView.RELAYOUT();
```

# 1. Bee Template 属性
## 1.1 常用通用属性
下列的属性都是标签内属性，通用是指所有标签均有此属性。

<!--
属性	    |        值        |   描述
--------|------------------|---------
id		| id               | 用来唯一标识一个元素，同时也是响应事件处理默认的`Signal Name`
class	| classname        | 规定元素的类名（classname）
style	| style_definition | 规定元素的行内样式（inline style）
-->
<table>
<thead>
<tr>
<th>属性      </th>
<th>        值        </th>
<th>   描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>id      </td>
<td> id               </td>
<td> 用来唯一标识一个元素，同时也是响应事件处理默认的<code>Signal Name</code></td>
</tr>
<tr>
<td>class   </td>
<td> classname        </td>
<td> 规定元素的类名（classname）</td>
</tr>
<tr>
<td>style   </td>
<td> style_definition </td>
<td> 规定元素的行内样式（inline style）</td>
</tr>
</tbody>
</table>

# 2. Bee Template 标签

Bee Template的标签目前有4种: `文档标签`, `布局标签`，`实体标签`，`样式标签`。

## 2.1 文档标签
### 2.1.1 ui
ui是`Bee Template XML`的最外层标签，&lt;ui&gt; 与 &lt;/ui&gt; 标签限定了文档的开始点和结束点，类似HTML里的 &lt;html&gt;.

## 2.2. 布局标签
### 2.2.1 linear
该标签是相对布局的核心元素，比HMTL的盒子模型简化了许多，CSS3的 [Flex-Box 草案](http://www.w3.org/TR/css3-flexbox/)与此类似。理论上来讲，通过水平和垂直两种布局方式的组合可以满足绝大部分的布局需求，不满足的部分可以通过自定义view来处理。

<b class="red">linear在解析时不会在oc里生成view，它的作用就是规定一组元素之间的相对位置关系。这也就意味着，如果只写了很多linear，没有包含view，你不会看到效果，因为在程序里并没有实际view生成，同样，给 linear 设定 <code>非布局样式(如background,border等)</code> 也是不起作用的，</b>
<!--
重要属性     |        值         |   描述    
------------|------------------|---------
orientation | [ v \| h (默认) ] |表示其内部元素是何种排列，v表示垂直，h表示水平
-->
<table>
<thead>
<tr>
<th>重要属性     </th>
<th>        值         </th>
<th>   描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>orientation </td>
<td> [ v | h (默认) ] </td>
<td>表示其内部元素是何种排列，v表示垂直，h表示水平</td>
</tr>
</tbody>
</table>

**example** : 

1. 水平布局 
    
    ![example-1][example-1]

2. 垂直布局

    把上面的例子中`wrapper`的orientation改为“v”，效果如下：

    ![example-2][example-2]

3. 水平与垂直嵌套
    
    一个水平里面嵌套两个垂直，每个垂直的各自包含两个

    ![example-3][example-3]

4. 各种嵌套
    
    可以找个界面，分析结构，然后试着写一下。

    如果不显示或者没有到达预期的效果，可以先检查一下<b class="red">是否给linear指定了大小</b>，请参考 [布局属性详解的`width`和`height`](#322-width-height)


## 2.3 实体标签
### 2.3.1 内置标签
<!--
标签     | 对应Cocoa控件      | 部分属性
--------|-------------------|----
view    | 默认`UIVIew`       | type
image   | `UIImageView`     | src 
input   | `UITextField`
label   | `UILabel`
button  | `UIButton`
scroll  | `UIScrollView`
slider  | `UISlider`
switch  | `UISwitch`
...     | ...
-->

<table>
<thead>
<tr>
<th>标签</th>
<th> 对应Cocoa控件</th>
</tr>
</thead>
<tbody>
<tr>
<td>view    </td>
<td> 默认<code>UIVIew</code></td>
</tr>
<tr>
<td>image   </td>
<td> <code>UIImageView</code></td>
</tr>
<tr>
<td>input   </td>
<td> <code>UITextField</code></td>
</tr>
<tr>
<td>label   </td>
<td> <code>UILabel</code></td>
</tr>
<tr>
<td>button  </td>
<td> <code>UIButton</code></td>
</tr>
<tr>
<td>scroll  </td>
<td> <code>UIScrollView</code></td>
</tr>
<tr>
<td>slider  </td>
<td> <code>UISlider</code></td>
</tr>
<tr>
<td>switch  </td>
<td> <code>UISwitch</code></td>
</tr>
<tr>
<td>...     </td>
<td> ...</td>
</tr>
</tbody>
</table>

以上标签的可以在 0.4版 [BeeFramework](https://github.com/gavinkwoe/BeeFramework) 

`/framework/application/mvc/view/template/parsers/Bee_UITemplateXML.m` 里找到默认的对应关系

```
+ (NSMutableDictionary *)classMapping
{
    if ( nil == __mapping )
    {
        __mapping = [[NSMutableDictionary alloc] init];
        __mapping.APPEND( @"view",      @"UIView" );
        __mapping.APPEND( @"image",     @"BeeUIImageView" );
        __mapping.APPEND( @"label",     @"BeeUILabel" );
        __mapping.APPEND( @"input",     @"BeeUITextField" );
        __mapping.APPEND( @"button",    @"BeeUIButton" );
        __mapping.APPEND( @"scroll",    @"BeeUIScrollView" );
        __mapping.APPEND( @"indicator", @"BeeUIActivityIndicatorView" );
        __mapping.APPEND( @"textArea",  @"BeeUITextView" );
        __mapping.APPEND( @"switch",    @"BeeUISwitch" );
        __mapping.APPEND( @"slider",    @"BeeUISlider" );
        __mapping.APPEND( @"check",     @"BeeUICheck" );
        __mapping.APPEND( @"zoom",      @"BeeUIZoomView" );
    }
    
    return __mapping;
}
```

### 2.3.2 <b class="red">自定义标签</b>
通过 `<view type="CustomViewClass"/>` 可以自定义View类型。  
`CustomViewClass`是自定义的View的className，在创建View时通过`NSClassFromString()`这个函数获取到oc对应的类。  
然后，发散一下思维，自定的这个view，也可以通过xml来写，那就是意味着可以xml的嵌套。

<!-- 好吧，你懂了。 -->
## 2.4 样式标签
### 2.4.1 style
在该标签内可以定义CSS。

<!--
属性         |        值                 |   描述    
------------|---------------------------|---------
type        | `text/css` or `text/json` | 目前推荐使用`text/css`。以后也许会扩展别的方式 (less？sass?)。
-->

<table>
<thead>
<tr>
<th>属性         </th>
<th>        值                 </th>
<th>   描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>type        </td>
<td> <code>text/css</code> or <code>text/json</code> </td>
<td> 目前推荐使用<code>text/css</code>。以后也许会扩展别的方式 (less？sass?)。</td>
</tr>
</tbody>
</table>

一般来讲，都是这样写

	<style type="text/css">
        #id /* #对应的元素的id */
        {
            key: value; (具体参考)
            key: value;
            ...
        }

        .class /* #对应的元素的class */
        {
            key: value;
            key: value;
            ...
        }
	</style>

# <b class="green">3. Bee Template Layout</b>

Bee Template在布局

## 3.1 Some Tips
### 3.1.1 布局的基本原则
由于布局的灵活性，可以用不同的方式来实现同一种效果。所以，我们有一个的原则：`尽量少得写代码`。
### 3.1.2 View显示优先级
与`addSubview`的顺序一致，最后添加的在最上面。
### 3.1.3 数值类型
<table>
<tbody>
<tr><th>值</th><th>描述</th></tr>
<tr><td rowspan="2">auto</td><td>如果是`Linear`，会根据包含元素大小总和计算出实际的宽/高度。</td></tr>
<tr><td>如果是`Label`，会根据内容的大小计算出实际的高度，实际应用时会至少指定宽/高。</td></tr>
<tr><td><i>length</i></td><td>单位px，于oc一致，固定值。</td></tr>
<tr><td><i>%</i></td><td>基于包含它的父级元素的百分比高度，父级的高度可以是auto，但确保它最终可以计算出固定的值。</td></tr>
</tbody>
</table>

## 3.2 布局属性详解

###  3.2.1 position

下面说提到的一组元素，是指属于同一个linear的子元素的集合。

#### 3.2.1.1 relative

默认值，相对布局。

元素位置取决于父级linear的排列方向。如果是v，那该元素位于相邻上一个元素的下方；如果是h，那该元素位于相邻上一个元素的右方。

#### 3.2.1.2 <b class="orange">absolute</b>

绝对布局。

position为该值的元素不参与同组元素布局的计算，也就意味着，它的起始点是父级元素的原点(0, 0)，与相邻元素无关，他的大小对同组元素也不产生影响。

通过`absolute`，你可以实现“[给一组元素加背景](#3222-auto)”，“元素之间重合、叠加”等等。

在该属性下，通过`top, left`来控制相对于父级原点的位置，也可以用margin来控制。

但是<b class="red">不推荐：用它来指定所有元素之间的位置关系，如果这样，就又回归到原始的方法。</b>

###  3.2.2 width 和 height

不论是`linear`还是`view`，宽高都要指定，值就是上面表上列出的类型，可以使固定值，可以是百分比，也可以是auto。

####  3.2.2.1 auto
    
   1. 当auto应用的元素是`linear`时，比如`height:auto`，它的高度是所有子元素高度的总和，这个时候就要保证子元素的高度是能确定的。
   
   * 下面这种情况是不对的
    
   ```    
    <linear h="auto">
        <linear h="50%"></linear> // 因为父级的高度根据子元素无确定，而子元素的高度又是父级的百分比。
    </linear>
   ```

   2. 当auto应用的元素是`label`时，比如`width:300px; height:auto`, 它的高度是根据文字内容的多少计算出来的，只不过有个前提的限定是文字的宽度为300px。

#### 3.2.2.2 auto 与 % 的混合使用
    
   1. 上面提到的“给一组元素加背景”，就是一个二者的混合使用的例子，代码如下：

   ```
    <?xml version="1.0" encoding="UTF-8"?>
    <ui namespace="TemplateBoard_iPhone">
        
        <linear orientation="h" class="wrapper">
            <image class="background"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
        </linear>

        <style type="text/css">
            .background {
                width: 100%;
                height: 100%;
                position: absolute; /* 表明 .background 这个元素不再参与布局计算 */
                background-color: #0ff;
            }

            .wrapper {
                width: 320px;
                height: auto; /* 这里可以是auto, 因为子元素两个 view 的高度是确定的，所以.background的高度100%最终取值和它的高度一样 */
            }
            .view {
                margin: 10px 5px;
                width: 150px;
                height: 100px; /* 如果是 label，高度也可以是 auto，因为最后的高度是可以根据文字的多少确定 */
            }
            .blue {
                background-color:#007DBC
            }
            .orange {
                background-color:#FF5E22
            }
        </style>
    </ui>
   ```

   ![example-4][example-4]

###  3.2.3 float 和 v-float

<!--
属性|可能的值|描述
-|-|-
float|left|元素自身水平居左
float|center|元素自身水平居中
float|right|元素自身水平居右
v-float|top|元素自身垂直居上
v-float|center|元素自身垂直居中
v-float|bottom|元素自身垂直居下
-->
这两个属性的作用就是`控制元素自身在某个方向上相对于父级元素的位置`，两个属性可以同时使用，也就是可以同时控制两个方向上的位置。

<table>
<thead>
<tr>
<th>属性</th>
<th>可能的值</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="3">float</td>
<td>left</td>
<td>水平居左</td>
</tr>
<tr>
<td>center</td>
<td>水平居中</td>
</tr>
<tr>
<td>right</td>
<td>水平居右</td>
</tr>
<tr>
<td rowspan="3">v-float</td>
<td>top</td>
<td>垂直居上</td>
</tr>
<tr>
<td>center</td>
<td>垂直居中</td>
</tr>
<tr>
<td>bottom</td>
<td>垂直居下</td>
</tr>
</tbody>
</table>

我们一个例子来搞定，先看效果：

![example-5][example-5]

代码如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<ui namespace="TemplateBoard_iPhone">
    <linear orientation="v" class="wrapper">
        <linear orientation="h" class="h-wrapper"> 
            <image class="bg gray"/>
            <view class="view blue fl"></view> <!-- fl, fc, fr 是定义的class，见style -->
            <view class="view orange fc"></view>
            <view class="view green fr"></view>
        </linear>
        <linear orientation="h" class="h-wrapper">
            <image class="bg gray"/>
            <view class="view blue vfl"></view>
            <view class="view orange vfc"></view>
            <view class="view green vfr"></view>
        </linear>
        <linear orientation="v" class="v-wrapper">
            <image class="bg gray"/>
            <view class="view blue fl"></view>
            <view class="view orange fc"></view>
            <view class="view green fr"></view>
        </linear>
        <linear orientation="v" class="v-wrapper">
            <image class="bg gray"/>
            <view class="view blue vfl"></view>
            <view class="view orange vfc"></view>
            <view class="view green vfr"></view>
        </linear>
    </linear>
    <style type="text/css">
        .fl { float:left; }
        .fc { float:center; }
        .fr { float:right; }
        
        .vfl { v-float:top; }
        .vfc { v-float:center; }
        .vfr { v-float:bottom; }

        .view { width: 30px; height: 30px; }
        .wrapper { width: 100%; height: 100%; }
        .h-wrapper { width: 320px; height: 120px; margin-top: 5px; }
        .v-wrapper { width: 320px; height: 120px; margin-top: 5px; }
        
        .bg { position: absolute; top:0;left:0; width: 100%; height: 100%; }

        .red { background-color:#DD1144 }
        .blue { background-color:#007DBC }
        .gray { background-color: #AEAEAE }
        .green { background-color:#008888 }
        .orange { background-color:#FF5E22 }
    </style>
</ui>
```

###  3.2.4 align 和 v-align
<!--
属性|可能的值|描述
-|-|-
align|left| 元素包含的所有子元素 水平居左
align|center| 元素包含的所有子元素 水平居中
align|right| 元素包含的所有子元素 水平居右
v-align|top| 元素包含的所有子元素 垂直居上
v-align|center| 元素包含的所有子元素 垂直居中
v-align|bottom| 元素包含的所有子元素 垂直居下
-->
该属性float不一样的是，它控制不是元素自身，而是包含其中的子元素。通常这个元素是linear，因为目前只有它具有包含别的元素的能力。

<table>
<thead>
<tr>
<th>属性</th>
<th>可能的值</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>align</td>
<td>left</td>
<td> 元素包含的所有子元素 水平居左</td>
</tr>
<tr>
<td>align</td>
<td>center</td>
<td> 元素包含的所有子元素 水平居中</td>
</tr>
<tr>
<td>align</td>
<td>right</td>
<td> 元素包含的所有子元素 水平居右</td>
</tr>
<tr>
<td>v-align</td>
<td>top</td>
<td> 元素包含的所有子元素 垂直居上</td>
</tr>
<tr>
<td>v-align</td>
<td>center</td>
<td> 元素包含的所有子元素 垂直居中</td>
</tr>
<tr>
<td>v-align</td>
<td>bottom</td>
<td> 元素包含的所有子元素 垂直居下</td>
</tr>
</tbody>
</table>

同样用一个例子来说一下，先看效果：

![example-6][example-6]

代码如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<ui namespace="TemplateBoard_iPhone">
    <linear orientation="v" class="wrapper">
        <linear orientation="h" class="h-wrapper val"> 
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
        <linear orientation="h" class="h-wrapper vac">
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
        <linear orientation="h" class="h-wrapper var">
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
        <linear orientation="v" class="v-wrapper al">
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
        <linear orientation="v" class="v-wrapper ac">
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
        <linear orientation="v" class="v-wrapper ar">
            <image class="bg gray"/>
            <view class="view blue"></view>
            <view class="view orange"></view>
            <view class="view green"></view>
        </linear>
    </linear>
    <style type="text/css">

        .al { align:left; }
        .ac { align:center; }
        .ar { align:right; }

        .val { v-align:top; }
        .vac { v-align:center; }
        .var { v-align:bottom; }

        .view { width: 20px; height: 20px; }
        .wrapper { width: 100%; height: 100%; }
        .h-wrapper { width: 320px; height: 60px; margin-top: 5px; }
        .v-wrapper { width: 320px; height: 60px; margin-top: 5px; }

        .bg { position: absolute; top:0;left:0; width: 100%; height: 100%; }

        .red { background-color:#DD1144 }
        .blue { background-color:#007DBC }
        .gray { background-color: #AEAEAE }
        .green { background-color:#008888 }
        .orange { background-color:#FF5E22 }
    </style>
</ui>
```

###  3.2.5 margin 和 padding

#### 3.2.5.1 margin

该属性是设置一个元素的外边距。外边距是指该元素与其他元素之间的距离，包括和它相邻的元素，以及与上一级边框之间的距离。本质上是改变自身的起始位置，如果是绝对定位则不影响后接相邻元素；如果是相对定位，则会影响到后接相邻元素的起始位置。

![example-7][example-7]

#### 3.2.5.1 padding

该属性是设置一个元素的内边距。内边距是指该元素边框与其包含元素之间的距离。由于UIView本是不具有padding的概念，所以本质上是改变元素自身的大小，达到相同效果。

![example-8][example-8]


# Not the End
## Demo Source
上面的例子中的XML源码可在 [Example-source][example-source] 中找到
## TODO
   * 数据的填充
   * Css样式详解

<!-- > 我愿意假设一下: 有一天,你会爱上这种界面布局的方式。

<p style="text-align:right; color:#999;">- By QFish</p> -->


<!-- links -->

[example-source]: https://github.com/qfish/Bee-Templates-Manual
[download-sdk]: http://bee-framework.com/download/BeeFramework_SDK_0.4.0.pkg.zip
[download-bee]: https://github.com/gavinkwoe/BeeFramework#lastest-version

<!-- images -->

[new-preoject]: http://imglf2.ph.126.net/ojKzvL5AYZ_L3uTT_ULIWw==/2561140813108279472.png
[new-file-board]: http://imglf2.ph.126.net/fJkjvkBEF6LOLVTSLztWSQ==/28991922619207010.png

[example-1]:http://imglf1.ph.126.net/pSJtEYfjCcwD0x6upTHGKQ==/6597166523681906911.png
[example-2]:http://imglf1.ph.126.net/ghVi3QgxMH5FfIKIoddnvA==/1981020886189847401.png
[example-3]:http://imglf2.ph.126.net/ISMAgtUHZLZJLuihFxZ7wg==/1141381030661826173.png
[example-4]:http://imglf0.ph.126.net/fQnYexMl9rP91G1RQbdCzg==/1265511495391172028.png
[example-5]:http://imglf2.ph.126.net/MEprDiYG4oBsiOJDgeHzNg==/3138727465400614892.png
[example-6]:http://imglf1.ph.126.net/5Zvt0scA2bUQWleOt3wZ2Q==/739997713872418672.png
[example-7]:http://imglf0.ph.126.net/rW1FbNNS6CeizTcCOjcSLA==/2110780850453299830.png
[example-8]:http://imglf2.ph.126.net/eFICHg71d3v7CQ8ArPSaiQ==/3381358895325351914.png

