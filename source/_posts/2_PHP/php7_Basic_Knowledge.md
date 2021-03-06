---
title: php7_Basic_Knowledge
date: '2020-11-01 11:36:11'
updated: '2020-11-07 10:37:05'
categories:
  - 2 PHP
---
# 一些超全局变量

```php
echo $_SERVER['HTTP_HOST'];//当前请求的 Host: 头部的内容 即域名信信息
echo $_SERVER['PHP_SELF'];//当前正在执行脚本的文件相对网站根目录地址，就算该文件被其他文件引用也可以正确得到地址
echo $_SERVER['SCRIPT_NAME'];//当前正在执行脚本的文件相对网站根目录地址，但当该文件被其他文件引用时，只显示引用文件的相对地址，不显示该被引用脚本的相对地址。
echo $_SERVER['DOCUMENT_ROOT'];//网站相对服务器地址即网站的绝对路径名 #当前运行脚本所在的文档根目录。在服务器配置文件中定义
echo $_SERVER['SCRIPT_FILENAME'];//当前执行脚本的绝对路径名。
```

# 类型约束

　　php7中，函数入口出口可以进行类型审查：

```php
function func(int $i,string $s,bool $b,float $f,类名 $c,接口名 $if,array $a,callable $cb):int{
}
```

　　php7增加了一个declare指令：“declare(strict_types=1);”，即使用严格模式。使用返回值类型声明时，如果没有声明为严格模式，如果返回值不是预期的类型，php还是会对其进行强制类型转换。但是如果是严格模式，则会出发一个TypeError的Fatal error（但不包括返回值的类型是预期类型的子类的情况）。

　　经过测试，php7还不能对变量和类当中成员变量进行类型约束，现在只能用以下方法：

```php
class A{
	private $haha;

	public function __set($name,$value){
		switch($name){
		case "haha":
			if(is_string($value)) $this->haha=$value;
			break;
		}
	}

	public function __get($name){
		switch($name){
		case "haha":
			return $this->haha;
			break;
		}
	}
}
```

# file_get_contents

　　如果文件不存在，会报错并返回布尔值false。可以在前面增加@抑制错误。

```php
$s=@file_get_contents(文件名);
```

# file_put_contents

　　如果文件所在的目录不存在，会报错。可以：

```php
@mkdir(dirname(文件名),0750,true);//true表示允许创建多级目录，@用来抑制如果目录存在报的错误
file_put_contents(文件名,写入内容);
```

# 数组和循环

　　php里的数组不管键是数字还是字符串，都是由内在顺序的，即使像这样：

```php
$a=[1=>"miaomiao",0=>"hehe"];
foreach($a as $k=>$v){
	echo $k;
}
```

输出是“10”。因此可以用array_push($arr,$value)、array_pop($arr,$value)、array_shift($arr,$value)（开头弹出）、array_unshift($arr,$value)（开头插入）进行操作。如下循环里取出键值就是按这样的顺序取出来的，并且取出的是__副本__：

```php
while(list($key,$value)=each($arr)){}
foreach($arr as $key=>$value){}
```

# 错误机制

## 解释

　　目前知道使用trigger_error(errormsg,errortype)可以产生用户错误，E_USER_ERROR会停止脚本运行，E_USER_WARNING、E_USER_NOTICE不会停止脚本运行。

　　PHP7改变了大多数错误的报告方式。不同于传统（PHP 5）的错误报告机制，现在大多数错误被作为 Error异常抛出。这种Error异常可以像Exception异常一样被第一个匹配的try/catch块所捕获。Error类和Exception类都是Throwable的子类，或者说Error类和Exception类都实现了Throwable接口。可以像下面这样拦截Error和Exception：

```php
catch (Throwable $e)  
{  
    echo "Error code: " . $e->getCode() . '<br>';  
    echo "Error message: " . $e->getMessage() . '<br>';  
    echo "Error file: " . $e->getFile() . '<br>';  
    echo "Error fileline: " . $e->getLine() . '<br>';  
}  
```  

　　另外，可以使用set_exception_handler可以设置没有被解决而被抛到最外面的异常的处理函数。

　　使用set_error_handler

```php
mixed set_error_handler ( callable $error_handler [, int $error_types = E_ALL | E_STRICT ] ) 
```

可以设置没有被解决而被抛到最外面的Eroror异常的处理函数：

```php
//自定义的错误处理方法  
function _error_handler($errno, $errstr ,$errfile, $errline)  
{  
    echo "错误编号errno: $errno<br>";  
    echo "错误信息errstr: $errstr<br>";  
    echo "出错文件errfile: $errfile<br>";  
    echo "出错行号errline: $errline<br>";  
}  
set_error_handler('_error_handler', E_ALL | E_STRICT);
```

## 所有错误种类

- __E_ERROR(0x1)__

Fatal run-time errors. Errors that can not be recovered from. Execution of the script is halted

- __E_WARNING(0x2)__

Non-fatal run-time errors. Execution of the script is not halted

- __E_PARSE(0x4)__

Compile-time parse errors. Parse errors should only be generated by the parser

- __E_NOTICE(0x8)__

Run-time notices. The script found something that might be an error, but could also happen when running a script normally

- E_CORE_ERROR(0x10)

Fatal errors at PHP startup. This is like an E_ERROR in the PHP core

- E_CORE_WARNING(0x20)

Non-fatal errors at PHP startup. This is like an E_WARNING in the PHP core

- E_COMPILE_ERROR(0x40)

Fatal compile-time errors. This is like an E_ERROR generated by the Zend Scripting Engine

- E_COMPILE_WARNING(0x80)

Non-fatal compile-time errors. This is like an E_WARNING generated by the Zend Scripting Engine

- __E_USER_ERROR(0x100)__

Fatal user-generated error. This is like an E_ERROR set by the programmer using the PHP function trigger_error()

- __E_USER_WARNING(0x200)__

Non-fatal user-generated warning. This is like an E_WARNING set by the programmer using the PHP function trigger_error()

- __E_USER_NOTICE(0x400)__

User-generated notice. This is like an E_NOTICE set by the programmer using the PHP function trigger_error()

- E_STRICT(0x800)

Run-time notices. PHP suggest changes to your code to help interoperability and compatibility of the code

- E_RECOVERABLE_ERROR(0x1000)

Catchable fatal error. This is like an E_ERROR but can be caught by a user defined handle (see also set_error_handler())

- __E_ALL(0x2000)__

All errors and warnings, except level E_STRICT (E_STRICT will be part of E_ALL as of PHP 6.0)

# SQLite3

　　使用SQLite3类的exec函数时，如果有多条语句，其中一条语句如果出错，就会报WARNING错误，并且所有操作会被回滚。

　　存取中文没有问题。

参考：
- <https://segmentfault.com/q/1010000002557534>
- <http://blog.csdn.net/zhang197093/article/details/75094816>

# 原生post和JSON

　　JSON优势就是数据结构更复杂，原生post不用担心“&”问题，会转义,但是无法分辨数字和字符串，传过来的都是字符串。

# 作用域

　　函数内无法直接读取函数外的变量，需要在函数内用

```php
global $v;
```

才能访问。另外匿名函数里使用

```php
function()use(&$v){}
```
应该也能访问外部变量。
# 添加新插件

　　Ubuntu16.04下开启sqlite3的扩展要修改/etc/php/7.0/apache2/php.ini里的对应扩展项，注意结尾改成.so。然后

```sh
apt-get install php7.0-sqlite
```

参考：<https://www.cnblogs.com/boundless-sky/p/6616401.html>
