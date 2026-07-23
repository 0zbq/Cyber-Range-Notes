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

![1](xss_labsxss_labs/photo/1.png)

这说明，网页直接将用户输入内容直接写入html中

# level2

第二关是一个有搜索框的GET类型的注入，注入点：keyword

![2](xss_labs/photo/2.png)

位置1是html标签或文本，位置2是input标签的value值

尝试过<script>alert()</script>发现位置1正常输出，说明位置1进行了特殊字符编码，不存在漏洞。

位置2是input标签的value值，可以尝试闭合。

然后，尝试

```
'"><script>alert()</script> 
```

发现成功。

# level3

![3](xss_labs/photo/3.png)

尝试

<script>alert()</script>

![3.1](xss_labs/photo/3.1.png)

![3.2](xss_labs/photo/3.2.png)

两个位置，均进行了特殊字符编码。并且，有页面源代码可以发现，位置2或许可以用单引号闭合，并使用事件来实现注入。

payload：

```
' onclick='alert()'
```

点击搜索框，触发事件。

![3.3](xss_labs/photo/3.3.png)

# level4

![4](xss_labs/photo/4.png)

位置1应该不存在漏洞。

位置2可以尝试用双引号闭合，并使用事件。

![4.1](xss_labs/photo/4.1.png)

# level5

![5](xss_labs/photo/5.png)

位置1应该不存在漏洞。

位置2可以尝试用双引号闭合，并使用事件。

payload

```
" onclick="alert()"
```

![5.1](xss_labs/photo/5.1.png)

可见，onclick被替换成o_nclick。很有可能对on进行了替换。

payload

```
"><script>alert()</script>"<
```

![5.2](xss_labs/photo/5.2.png)

可见，<script>被替换成<scr_ipt>

payload

```
script
```

![5.3](xss_labs/photo/5.3.png)

发现script并没有被替换。

payload

```
"><a href='javascript:alert()'>
```

![5.4](xss_labs/photo/5.4.png)

![5.5](xss_labs/photo/5.5.png)

闭合input标签，并使用a标签的href属性可以执行javascript脚本的这一特性。

# level6

payload

```
'"> onclick script
```

![6](xss_labs/photo/6.png)

可见onclick被替换了，但script没有被替换。

payload

```
><a href="javascript:alert()">
```

![6.1](xss_labs/photo/6.1.png)

发现href被替换了

我测试了一下<script>,被替换成了<scr_ipt>。

尝试大写

payload

```
<SCRIPT> ONCLICK HREF
```

![6.2](xss_labs/photo/6.2.png)

发现都未被替换

随便尝试几种方案：

```
" ONCLICK="alert()"
```

![6.3](xss_labs/photo/6.3.png)

```
"><a HREF="javascript:alert()">
```

![6.4](xss_labs/photo/6.4.png)