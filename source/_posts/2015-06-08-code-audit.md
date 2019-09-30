---
layout: post
title: php代码审计学
tags: ['代码审计']
---


## php代码审计 ##

### 概述 ###

代码审计学，通过对程序源码进行系统规范的检测，通过对源码的分析找到程序在设计开发阶段可以存在的漏洞和逻辑错误，避免程序被非法入侵，造成企业的风险。

## 输入输出 ##

输入输出是用户是系统与用户接触最频繁的交互，也是最有可能形成漏洞的区域，如果我们对用户的输入过滤验证不严格，用户 就可以构造恶意语句形成漏洞，在处理输入输出时一定要保持 用户所用能控制的值都是不安全的 都必须要通过我们程序过滤判断处理。

- 对数据进行精确匹配
- 接受白名单数据
- 拒绝黑名单数据
- 编码数据

在php中可由用户输入的变量列表：

- $_SERVER
- $_GET
- $_POST
- $_COOKIE
- $_FILES
- $_ENV
- $_HTTP_COOKIE_VARS
- $_HTTP_ENV_VARS
- $_HTTP_GET_VARS
- $_HTTP_POST_FILES
- $_HTTP_POST_VARS
- $_HTTP_SERVER_VARS

### 命令注入 ###

php中主要实现命令的函数有：

- system
- exec
- passthru
- \`\`
- shell_exec
- popen
- proc_open
- pcntl_exec 

审计中 我们通过批量搜索这些函数 对所带参数进行跟踪，判断是否过滤严格。

### 跨站脚本 ###

反射型跨站常常出现在用户提交的变量接受以后经过处理，直接输出显示给客户端；存储型跨站常常出现在用户提交的变量接受过经过处理后，存储在数据库里，然后又从数据库中读取到此信息输出到客户端。

输出函数经常使用：

- echo
- print
- printf
- vprintf
- <%=$test%> 

对于反射型跨站，因为是立即输出显示给客户端，所以应该在当前的php页面检查变量被客户提交之后有无立即显示，在这个过程中变量是否有经过安全检查

### 文件包含 ###

PHP可能出现文件包含的函数：

- include
- include_once
- require
- require_once
- show_source
- highlight_file
- readfile
- file_get_contents
- fopen
- nt>file

### 代码执行 ###

PHP可能出现代码注入的函数：

- eval
- preg_replace+/e
- assert
- call_user_func
- call_user_func_array
- create_function

### sql注入 ###

SQL注入因为要操作数据库，所以一般会查找SQL语句关键字：

- insert
- delete
- update
- select

### XPath注入 ###

Xpath用于操作xml，我们通过搜索xpath来分析，提交给<fo nt face="Arial, sans-serif">xpath</fo 函数的参数是否有经过安全处理

### HTTP响应拆分 ###

PHP中可导致HTTP响应拆分的情况为：使用header函数和使用$_SERVER变量。注意PHP的高版本会禁止HTTP表头中出现换行字符，这类可以直接跳过本测试。

### 文件管理 ###

PHP的用于文件管理的函数，如果输入变量可由用户提交，程序中也没有做数据验证，可能成为高危漏洞。我们应该在程序中搜索如下函数：

- copy
- rmdir
- unlink
- delete
- fwrite
- chmod
- fgetc
- fgetcsv
- fgets
- fgetss
- file
- file_get_contents
- fread
- readfile
- ftruncate
- file_put_contents
- fputcsv
- fputs

### 文件上传 ###

PHP文件上传通常会使用move_uploaded_file

### 变量覆盖 ###

1. 遍历初始化变量 例： ```foreach($_GET as $key => $value) $$key = $value; ```
2. 函数覆盖变量：
	- parse_str
	- mb_parse_str
	- import_request_variables 
3. Register_globals=ON时，GET方式提交变量会直接覆盖

### 动态函数 ###

当使用动态函数时，如果用户对变量可控，则可导致攻击者执行任意函数

## 会话安全 ##

- HTTPOnly设置<br>
`session.cookie_httponly = ON 时，客户端脚本(JavaScript 等)无法访问该cookie，打开该
指令可以有效预防通过XSS 攻击劫持会话ID`

- domain 设置<br>
`检查session.cookie_domain 是否只包含本域，如果是父域，则其他子域能够获取本域的
cookies`

- path设置<br>
`检查session.cookie_path，如果网站本身应用在/app，则path 必须设置为/app/，才能保
证安全`

- cookies持续时间<br>
`检查session.cookie_lifetime，如果时间设置过程过长，即使用户关闭浏览器，攻击者也会
危害到帐户安全`

- secure设置<br>
`如果使用HTTPS，那么应该设置session.cookie_secure=ON，确保使用HTTPS 来传输
cookies`

- session固定<br>
`如果当权限级别改变时（例如核实用户名和密码后，普通用户提升到管理员），我们就应该
修改即将重新生成的会话ID ，否则程序会面临会话固定攻击的风险。`

- CSRF<br>
`跨站请求伪造攻击，是攻击者伪造一个恶意请求链接，通过各种方式让正常用户访问后，
会以用户的身份执行这些恶意的请求。我们应该对比较重要的程序模块，比如修改用户密码，添
加用户的功能进行审查，检查有无使用一次性令牌防御csrf 攻击。`



## 加密解密 ##

- 明文存储密码<br>
`采用明文的形式存储密码会严重威胁到用户、应用程序、系统安全。`

- 密码弱加密<br>
`使用容易破解的加密算法，MD5加密已经部分可以利用md5破解网站来破解`

- 密码存储在攻击者能访问到的文件<br>
`例如：保存密码在txt 、ini、conf、inc、xml 等文件中，或者直接写在HTML 注释中`


## 认证和授权 ##

- 用户认证<br>
`检查代码进行用户认证的位置，是否能够绕过认证，例如：登录代码可能存在表单注入。检查登录代码有无使用验证码等，防止暴力破解的手段`

- 函数或文件的未认证调用<br>
`一些管理页面是禁止普通用户访问的，有时开发者会忘记对这些文件进行权限验证，导致漏洞发生某些页面使用参数调用功能，没有经过权限验证，比如index.php?action=upload`

- 密码硬编码<br>
`有的程序会把数据库链接账号和密码，直接写到数据库链接函数中。`


## 随机函数 ##

- rand()<br>
`rand()最大随机数是32767，当使用rand 处理session 时，攻击者很容易破解出session，建议使用mt_rand()`

- mt_srand()和mt_rand()<br>
`PHP4和PHP5<5.2.6，这两个函数处理数据是不安全的。在web 应用中很多使用mt_rand
来处理随机的session，比如密码找回功能等，这样的后果就是被攻击者恶意利用直接修改密码。`

<!--
## 编码 ## 

## php危险函数 ##
-->

## 信息泄漏 ##
phpinfo()

## 环境配置 ##

- open_basedir 设置
- allow_url_fopen 设置
- allow_url_include设置
- safe_mode_exec_dir 设置
- magic_quote_gpc设置
- register_globals设置
- safe_mode 设置
- session_use_trans_sid 设置
- display_errors 设置
- expose_php设置


