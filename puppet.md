### 条件语句
常用比较符:
* 等于==(不区分大小写)
* 不等于!=
* 正则表达式 =~操作符, !~操作符

1.if语句<br>
if语句可以没有返回值, 但一定要有notify资源. puppet会自动将if后的表达式转换为布尔类型:
* 字符串: 除了空字符串都是true
* 数字: 所有数字都是true, 包括0和负数
* undef: false
* 数组和hash: 都是true, 包括空数组和空哈希
* 资源引用: 都是true
```puppet
if $operatingsystem == "ubuntu" {
  notify {"blablabla": }
}
elsif $operatingsystem in [ "centos", "redhat" ] {
  notify {"blablabla": }
}
else {
  notify {"blablabla": }
}
```
2.selector选择器<br>
只返回一个值, 不执行代码
```puppet
$file_mode = $:: operatingsystem ? {
  debian => '0007',
  redhat => '0776',
  fedora => '0007',
  default => '0001',
}
file { '/tmp/readme.txt':
  content => "Hello World\n",
  mode => $file_mode,
}
```
3.case语句<br>
没有返回值也没有notify资源的调用
```puppet
case $:: operatingsystem {
  centos: {
    $version = '1.2.3'
  }
  solaris: {
    $version = '3.2.1'
  }
  default: {
    fail("Module ${module_name} is not supported on ${:: operatingsystem}")
  }
}
```

### 类
1.文件名
    定义一个apache类, 有两个子类 其配置方法如下:
```puppet
# /etc/puppetlabs/puppet/modules/apache/manifests
# init.pp
class apache { }
# ssl.pp
class apache:: ssl { }
# virtual_host.pp
define apache:: virtual_host () { }
```

2.资源关系声明必须从左到右
```puppet
Package['httpd'] -> Service['httpd']
```

3.类的继承
```puppet
class ssh {}
class ssh:: client inherits ssh {}
class ssh:: server inherits ssh {}
class ssh:: server:: solaris inherits ssh:: server {}
```

4.变量名称定义时最好使用双冒号标注
```puppet
$:: operatingsystem
```


###代码调试
通过为exec
```puppet
$ vim test_output.pp
exec { 'test_logoutput'
  command => "/bin/ls linuxtone.org",
  logoutput => on_failure,
}
```
也可以通过notify
```puppet
notify { "I am running on node $fqdn": }
notify { "I an running on operatingsystem is $operatingsystem": }
```


###puppet依赖
####关键词指向的资源首字母应该大写
不具备触发功能, 只表示依赖关系<br>
* require: 引用的对象执行之后该资源才被应用
* before: 与require相反

带触发功能
* subscribe: 类似require, 当引用对象资源发生改变时, 执行相应的动作
* notify: 类似before, 该资源发生改变时, 通知某资源进行更新

指定依赖常用的方法<br>
->表示require主要依赖关系, ~>表示notify主要触发动作
```puppet
Package['sshd'] -> File['/etc/sshd/sshd_config'] ~> Service['sshd']
```
指定某个资源依赖与某个类
```puppet
require => Class['repo163']
```
指定某个资源依赖于软件包及某个类
```puppet
require => [Package['sshd'], Class['ssh:: config']],
```
package应用之前所有的yumrepo必须要先被应用, <||>运算符筛选一批资源, 中间的运算符代表匹配规则
```puppet
yumrepo <| |> -> package <| provider == yum|>
```

###虚拟资源
定义虚拟资源, 前面加上@
```puppet
@user { luke: ensure => present }
```
实例化
1.采用<||>语法
```puppet
User <| title == luke |>
User <| (group ==dba or group == sysadmin or title == luke )|>
```
2.realize函数
```puppet
realize User[luke]
realize(User[johnny], User[billy])
```

###作用域与变量
####作用域
全局作用域只能访问全局变量, 节点作用域可以访问节点变量和全局变量, example::parent/example::other/example::four可以访问自己的变量以及节点变量和全局变量, example::child可以访问所有变量.
* 全局作用域在site.pp
* 节点作用域定义在node内
* 局部作用域定义在类内
* 同名变量以最局部的定义为准

####变量
同一作用域下只能赋值一次, 不同作用域可再赋值, 以最局部的赋值为准<br>
* 客户端内置变量
  * $environment: 节点环境
  * $clientcert: 节点名称certname
  * $clientversion: 客户端puppet版本
* 服务端内置变量
  * $servername: 服务端完整、合格的域名, 主机名即facter中的fqdn
  * $serverip: 服务端ip
  * $settings::<name of setting>: 服务端设置项的值
  * $module_name: 模块名称
  
###哈希类型
```puppet
{ key1 => 'val1', key2 => 'val2' }
$myhash = { key => "some value", other+key => "some other value" }
notice( $myhash[key] )
```

###标签
在puppet定义的manifests中, 资源名、类名、自定义类型将会被自动打上标签
```puppet
apache::vhost {'docs.puppetlabs.com':
  port => 80,
  tag => ['us_mirror1', 'us_mirror2'],
}
```
```shell
$ sudo puppet agent --test --tags apache, us_mirror1
```
