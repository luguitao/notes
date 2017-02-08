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
