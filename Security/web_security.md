## CGI命令注入
### 漏洞描述：
空白黑客可以执行任意系统命令，这通常意味着服务器完全被拿下。

Web程序代码中把用户提交的数据未做过滤就直接传给shell去执行，导致攻击者可以执行任意系统命令。某些脚本是通过system这类的系统函数来运行命令。在设计存在缺陷的程序中，会将用户的输入也作为被运行命令的一部分，而没有做严格的限制，在这种情况下，攻击者可以通过构造传入的参数来运行任意命令。这类代码注入可以使用各种语法，举例如下：	
```
command1|command2（利用command1的输出作为command2的输入－攻击在于“|命令”）
 
command1&&command2（如果command1的返回码是true，便运行command2－攻击在于“&&命令”）
 
command1||command2（如果command1的返回码是false，便运行command2－攻击在于“||命令”）。
```

有时候，会用单引号（'）或双引号（"）括住第一个命令，因此，如果要运行第二个命令，必须先转义引号。攻击者可以利用这些变体，尝试在主机上执行任意代码。
		 
例如，PHP中可以使用下列5个函数来执行外部的应用程序或函数：`system`、`exec`、`assthru`、`shell_exec`、 `` ` ` ``(与`shell_exec`功能相同)。
	 
如下面的程序exp.php
	
```php
<?php
$dir=$_GET["dir"];
if(isset($dir))
{
	echo"<pre>";
	system&#40;"ls-al".$dir&#41;;
	echo"</pre>";
}
?>;
```
 
我们提交 http://www.vuln.com/exp.php?dir=|cat/etc/passwd ，提交以后，命令变成了`system(“ls-al|cat/etc/passwd”);`。将会把`/etc/passwd`中内容输出到页面上。

### 解决方案:
1. 尽量不要执行外部命令
2. 使用自定义函数或函数库来替代外部命令的功能
3. 指定可执行文件的路径，如php中在php.ini指定 `safe_mode_exec_dir=on`
4. 清理输入以排除对执行操作系统命令有意义的符号例如：|（竖线符号）, &（&符号）, ;（分号）
5. 绝不将未检查的用户输入传递给`eval()`、`open()`、`sysopen()`、`system();`之类的Perl命令。确保输入未包含恶意的字符，例如: $（美元符号）,%（百分比符号）, @（at符号）
6. 编码时使用ESAPI安全库


## SQL注入漏洞

### 漏洞描述：
若需要注入利用，请选择`sqlmap`或者最新版`pangolin`等测试下载地址：http://sqlmap.org/, sqlmap用户手册：http://drops.wooyun.org/tips/143.

SQL注入被广泛用于非法入侵网站服务器，获取网站控制权。它是应用层上的一种安全漏洞。通常在设计存在缺陷的程序中，对用户输入的数据没有做好过滤，导致恶意用户可以构造一些SQL语句让服务器去执行，从而导致数据库中的数据被窃取，篡改，删除，以及进一步导致服务器被入侵等危害。

SQL注入的攻击方式多种多样，较常见的一种方式是提前终止原SQL语句，然后追加一个新的SQL命令。为了使整个构造的字符串符合SQL语句语法，攻击者常用注释标记如“--”（注意空格）来终止后面的SQL字符串。执行时，此后的文本将被忽略。如某个网站的登录验证SQL查询代码为`strSQL="SELECT*FROMusersWHEREname=‘”+userName+“’andpw=’”+passWord+”’”`，其中`userName`和`passWord`是用户输入的参数值，用户可以输入任何的字符串。如果用户输入的`userName=admin’--`，`passWord`为空，则整个SQL语句变为`SELECT*FROMusersWHEREname=’admin’--‘andpw=’’`，等价于`SELECT*FROMusersWHEREname=’admin’`，将绕过对密码的验证，直接获得以admin的身份登录系统。
 
### SQL注入导致的危害：

1. 数据库信息泄漏，例如个人机密数据，帐户数据，密码等。
2. 删除硬盘数据，破坏整个系统的运行。
3. 数据库服务器被攻击，系统管理员帐户被窜改（例如`ALTERLOGINsaWITHPASSWORD='xxxxxx'`）。
4.取得系统较高权限后，可以篡改网页以及进行网站挂马。
5.经由数据库服务器提供的操作系统支持，让黑客得以修改或控制操作系统，植入后门程序（例如`xp_cmdshell"netstopiisadmin"`可停止服务器的IIS服务）。

### 解决方案:
1. 输入过滤，对于整数，判断变量是否符合[0-9]的值；其他限定值，也可以进行合法性校验；对于字符串，对SQL语句特殊字符进行转义(单引号转成两个单引号，双引号转成两个双引号)。MySQL也有类似的转义函数`mysql_escape_string`和`mysql_real_escape_string`。Asp的过滤参考此页面http://blogs.iis.net/nazim/archive/2008/04/28/filtering-sql-injection-from-classic-asp.aspx
2. 在设计应用程序时，完全使用参数化查询（ParameterizedQuery）来设计数据访问功能。
3. 使用其他更安全的方式连接SQL数据库。例如已修正过SQL注入问题的数据库连接组件，例如ASP.NET的SqlDataSource对象或是LINQtoSQL，安全API库如ESAPI。
4. 使用SQL防注入系统。
5. 严格限制数据库操作的权限。普通用户与系统管理员用户的权限要有严格的区分。建立专门的账户，同时加以权限限制，满足应用的需求即可。

## 路径遍历漏洞
### 漏洞描述

未对用户输入的字符进行正确过滤，未检查用户输入中是否包含`..`（两个点）字符串，攻击者利用该漏洞能够跳出当前目录，访问其他目录的文件。
 
### 漏洞危害：
攻击者或许会浏览受限制的文件（例如：unix下的/etc/passwd文件），读取数据库配置文件，系统配置文件，或者执行一些命令，这会导致对整个webserver的威胁。

### 解决方案:
空白确保请求的文件驻留于Web服务器的虚拟路径中。
确保仅可打开特定的扩展名
从用户输入中除去特殊字符（元字符），如`./\&%#^()@`


## DOMXSS
### 漏洞描述：

发生在客户端DOM（DocumentObjectModel文档对象模型）DOM是一个与平台、编程语言无关的接口，它允许程序或脚本动态地访问和更新文档内容、结构和样式，处理后的结果能够成为显示页面的一部分。DOM中有很多对象，其中一些是用户可以操纵的，如uRI，location，refelTer等。客户端的脚本程序可以通过DOM动态地检查和修改页面内容，它不需要提交数据到服务器端，而从客户端获得DOM中的数据在本地执行，如果DOM中的数据没有经过严格确认，就会产生DOMXSS漏洞
	
### 解决方案:
1. 对输入数据严格匹配，比如只接受数字输入的就不能输入其他字符。不仅要验证数据的类型，还要验证其格式、长度、范围和内容。
2. 输入过滤，应该在服务器端进行。PHP在设置`magic_quotes_gpc`为`On`的时候，会自动转义参数中的单双引号，但这不足以用于XSS漏洞的防御，仍然需要在代码级别防御。
3. 输出编码
	A. 用户输入的参数值会展现在HTML正文中或者属性值中例如：
	
```
<a href='http://test.com'>Un-trustedinput</a>
```

	属性值：`<inputname="searchword" value="Un-trustedinput">`;
 
此时需要将红色的不可信内容中做如下的转码(即将`<>‘“` `` ` ``转成html实体)：
 
	B. 用户输入落在`<script>`的内容中, 最好不要让用户的输入落在`<script>`用户输入`</script>`这里，如果无法避免的话，建议严格限制用户的输入，比如输入为整数时，要验证输入是否只包含数字。当输入为字符串时，将字符串用单引号或双引号包含起来，并且对用户的输入字符中包含的单双引号过滤或转换为HTML实体。
 
4.编码时使用ESAPI库或其他antixss库。

## 目录浏览 
### 漏洞描述： 
空白被检测目录有列出文件目录权限，从此目录用户可以浏览目录下所有文件列表，这可能导致敏感信息的泄露
 
### 漏洞危害：
文件信息、敏感信息的泄露，为进一步的针对性攻击提供了信息
如：www.7dscan.com/databackup/若存在目录浏览，则数据库备份文件暴露就可被任意下载

### 解决方案:
空白你必须确保此目录中不包含敏感信息。
可以在Web服务器配置中限制目录列表。建议修正所使用的Web服务器软件的目录权限设置。例如，在IIS中取消目录浏览：
在nginx中取消目录浏览：
去掉配置文件里面的目录浏览项：`autoindexon`;
在apache中取消目录浏览：
在Apache配置文件中的目录配置中的“`Indexes`”删除或者改为“`-Indexes`”

## ApacheHTTPServer413errorpagesXSS漏洞

###漏洞描述：
目标服务器apache版本在用户传入的请求头包含一些特殊构造的的数据时，会造成XSS攻击。
参考：http://procheckup.com/Vulnerability_PR07-37.php
http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2007-6203

###解决方案:
更新Apache到最新版本

## CRLF注入漏洞
### 漏洞描述：
应用程序常常会将用户的数据嵌入到http的响应头中，常常会包含用户数据的HTTP头部字段有以下三个：`Set-Cookie`、302跳转的Location、其他自定义Header。当服务器段对这些用户的数据没有做好特殊字符的过滤时，恶意用户可以通过精心构造的数据请求来达到恶意的目的。
 
### 漏洞危害：
跨站点脚本编制：a)攻击者通过输入两个回车换行符提前结束HTTP响应的头部，然后将恶意的脚本添加到响应体中实现跨站。b)直接在头部添加标签`Link:<http://www.test.com/xss.css>;REL:stylesheet`实现跨站。攻击者可以通过跨站来窃取或操纵客户会话和cookie，它们可能用于模仿合法用户，从而使黑客能够以该用户身份查看或变更用户记录以及执行事务。
跨域问题：通过插入一个P3P头，然后再在一个别的域引用此处，则会造成隐私数据的跨域问题。
Web高速缓存毒害：在这个情况下，目标是在攻击者与Web服务器之间的路径上强迫Web高速缓存，将攻击者提供的数据当作属于易受攻击的站点的资源来高速缓存。
 
### 解决方案:
请确保输入未包含恶意的字符，过滤用户输入，例如： 
[1]CR（回车符，ASCII0x0d）
[2]LF（换行，ASCII0x0a）

## 内网地址信息泄露
服务器开启`OPTIONS`方法

### 漏洞描述： 
OPTIONS方法是用于请求获得由Request-URI标识的资源在请求/响应的通信过程中可以使用的功能选项。通过这个方法，客户端可以在采取具体资源请求之前，决定对该资源采取何种必要措施，或者了解服务器的性能。开启该方法有可能泄漏一些敏感信息，为攻击者发起进一步攻击提供信息。

### 解决方案:
建议关闭该功能
