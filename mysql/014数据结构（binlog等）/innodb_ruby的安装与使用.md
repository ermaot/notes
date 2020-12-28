## 安装innodb_ruby

innodb_ruby主要可查看innodb数据库数据表的各种存储，解析innodb的文件，用于学习数据库底层的一些存储。

#### 安装ruby
```
yum install ruby  
```

会安装ruby语言、gem工具以及相关的包

```
Installed:
  ruby-2.5.5-106.module_el8.3.0+571+bab7c6bc.x86_64                                                                          
  rubygem-bigdecimal-1.3.4-106.module_el8.3.0+571+bab7c6bc.x86_64                                                            
  rubygem-did_you_mean-1.2.0-106.module_el8.3.0+571+bab7c6bc.noarch                                                          
  rubygem-io-console-0.4.6-106.module_el8.3.0+571+bab7c6bc.x86_64                                                            
  rubygem-rdoc-6.0.1-106.module_el8.3.0+571+bab7c6bc.noarch                                                                  
  rubygems-2.7.6.2-106.module_el8.3.0+571+bab7c6bc.noarch                                                                    
  ruby-irb-2.5.5-106.module_el8.3.0+571+bab7c6bc.noarch                                                                      
  ruby-libs-2.5.5-106.module_el8.3.0+571+bab7c6bc.x86_64                                                                     
  rubygem-json-2.1.0-106.module_el8.3.0+571+bab7c6bc.x86_64                                                                  
  rubygem-openssl-2.1.2-106.module_el8.3.0+571+bab7c6bc.x86_64                                                               
  rubygem-psych-3.0.2-106.module_el8.3.0+571+bab7c6bc.x86_64                                                                 

Complete!

```

然后再使用gem安装

```
# gem install innodb_ruby
Building native extensions. This could take a while...
Successfully installed digest-crc-0.6.3
Fetching: innodb_ruby-0.9.16.gem (100%)
Successfully installed innodb_ruby-0.9.16
2 gems installed
```

如果报错

```
Building native extensions. This could take a while...
ERROR:  Error installing innodb_ruby:
	ERROR: Failed to build gem native extension.
```

这是因为缺少ruby-devel，安装即可

```
# yum install ruby-devel
```

## innodb_ruby的使用

