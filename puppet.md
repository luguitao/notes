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
