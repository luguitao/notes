### 条件语句
1.保持资源的声明 
```puppet
$file_mode = $:: operatingsystem ? {
  debian => '0007',
  redhat => '0776',
  fedora => '0007',
}
file { '/tmp/readme.txt':
  content => "Hello World\n",
  mode => $file_mode,
}
```
2.case语句配置的默认值
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
