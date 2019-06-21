---
title: logstash插件开发
date: 2016-10-24 21:23:14
tags: research
categories:
- elasticsearch
---
## 背景
logstash强大魅力在于它的插件体系，虽然官方插件很多，但不可能满足所有的要求，因此就需要定制化个性化插件，本次结合Logstash Monitor Redis需求开发专用插件，以实现动态化获取master 实例中info 信息。
## logstash插件介绍
### 体系结构
```
$ tree logstash-input-example
├── Gemfile
├── LICENSE
├── README.md
├── Rakefile
├── lib
│   └── logstash
│       └── inputs
│           └── example.rb
├── logstash-input-example.gemspec
└── spec
    └── inputs
        └── example_spec.rb
```
其实只需要这logstash-input-example.gemspec,example.rb两个文件即可。
mypluginname_spec.rb 是测试类。

先看看logstash-input-example.gemspec都做了什么吧！
```
Gem::Specification.new do |s|
  s.name = 'logstash-input-example'
  s.version         = '2.0.4'
  s.licenses = ['Apache License (2.0)']
  s.summary = "This example input streams a string at a definable interval."
  s.description     = "This gem is a Logstash plugin required to be installed on top of the Logstash core pipeline using $LS_HOME/bin/logstash-plugin install gemname. This gem is not a stand-alone program"
  s.authors = ["Elastic"]
  s.email = 'info@elastic.co'
  s.homepage = "http://www.elastic.co/guide/en/logstash/current/index.html"
  s.require_paths = ["lib"]

  # Files
  s.files = Dir['lib/**/*','spec/**/*','vendor/**/*','*.gemspec','*.md','CONTRIBUTORS','Gemfile','LICENSE','NOTICE.TXT']
   # Tests
  s.test_files = s.files.grep(%r{^(test|spec|features)/})

  # Special flag to let us know this is actually a logstash plugin
  s.metadata = { "logstash_plugin" => "true", "logstash_group" => "input" }

  # Gem dependencies
  s.add_runtime_dependency "logstash-core", ">= 2.0.0", "< 3.0.0"
  s.add_runtime_dependency 'logstash-codec-plain'
  s.add_runtime_dependency 'stud', '>= 0.0.22'
  s.add_development_dependency 'logstash-devutils', '>= 0.0.16'
end
```
上面的信息，只要改改版本和名字，其他的信息基本不需要动。

关键的信息还有：

- s.require_paths定义了插件核心文件的位置
- s.add_runtime_dependency 定义了插件运行的环境
然后再看看example.rb
这个文件就需要详细说说了，基本的框架如下，
```
# encoding: utf-8
require "logstash/inputs/base"
require "logstash/namespace"
require "stud/interval"
require "socket" # for Socket.gethostname

# Generate a repeating message.
#
# This plugin is intented only as an example.

class LogStash::Inputs::Example < LogStash::Inputs::Base
  config_name "example"

  # If undefined, Logstash will complain, even if codec is unused.
  default :codec, "plain"

  # The message string to use in the event.
  config :message, :validate => :string, :default => "Hello World!"

  # Set how frequently messages should be sent.
  #
  # The default, `1`, means send a message every second.
  config :interval, :validate => :number, :default => 1

  public
  def register
    @host = Socket.gethostname
  end # def register

  def run(queue)
    # we can abort the loop if stop? becomes true
    while !stop?
      event = LogStash::Event.new("message" => @message, "host" => @host)
      decorate(event)
      queue << event
      # because the sleep interval can be big, when shutdown happens
      # we want to be able to abort the sleep
      # Stud.stoppable_sleep will frequently evaluate the given block
      # and abort the sleep(@interval) if the return value is true
      Stud.stoppable_sleep(@interval) { stop? }
    end # loop
  end # def run

  def stop
    # nothing to do in this case so it is not necessary to define stop
    # examples of common "stop" tasks:
    #  * close sockets (unblocking blocking reads/accepts)
    #  * cleanup temporary files
    #  * terminate spawned threads
  end
end # class LogStash::Inputs::Example
```
挨行看看！

首先第一行的# encoding: utf-8,不要以为是注释就没什么作用。它定义了插件的编码方式。

下面两行：

require "logstash/inputs/base"
require "logstash/namespace"
引入了插件必备的包。
```
class LogStash::Inputs::Example < LogStash::Inputs::Base
  config_name "example"
```
插件继承自Base基类，并配置插件的使用名称。

下面的一行对参数做了配置，参数有很多的配置属性，完整的如下：
```
 config :variable_name,:validate =>:variable_type,:default =>"Default value",:required => boolean,:deprecated => boolean
```
其中

variable_name就是参数的名称了。
validate 定义是否进行校验，如果不是指定的类型，在logstash -f xxx --configtest的时候就会报错。它支持多种数据类型，比如:string, :password, :boolean, :number, :array, :hash, :path (a file-system path), :codec (since 1.2.0), :bytes.
default 定义参数的默认值
required 定义参数是否是必须值
deprecated 定义参数的额外信息，比如一个参数不再推荐使用了，就可以通过它给出提示！典型的就是es-output里面的Index_type，当使用这个参数时，就会给出提示
### 插件安装
1. 便捷安装方式

第一步，首先把这个插件文件夹拷贝到下面的目录中
```
logstash-2.1.0\vendor\bundle\jruby\1.9\gems
```
第二步，修改logstash根目录下的Gemfile,添加如下的内容：
```
gem "logstash-filter-example", :path => "vendor/bundle/jruby/1.9/gems/logstash-filter-example-1.0.0"
```
第三步，编写配置文件，test.conf：
```
input{
    example{} 
}
filter{
    
}
output{
    stdout{
        codec => rubydebug
    }
}
```
第四步，输入logstash -f test.conf时，输入任意字符，回车~~~大功告成！
```
{
       "message" => "Hello World!",
      "@version" => "1",
    "@timestamp" => "2016-01-27T19:17:18.932Z",
          "host" => "cadenza"
}
```
2. 官方指导方式

第一步，build
```
gem build logstash-input-example.gemspec
```
会在当前路径下生成logstash-input-example-2.0.4.gem
第二步，install

```
bin/logstash-plugin install /logstash-input-example/logstash-input-example-2.0.4.gem
```
验证
```
validating /logstash-input-example/logstash-input-example-2.0.4.gem >= 0
Valid logstash plugin. Continuing...
Successfully installed 'logstash-input-example' with version '2.0.4'
```
第三步，查看plugin：
```
bin/logstash-plugin list
```
第四步，使用

略

## 开发案例
开发插件实现根据cluster nodes信息获取redis cluster 中master节点 info信息。使用该插件只用输入一条命令，即可动态获取相关信息。
### 插件开发
此插件是基于exec基础上封装的，主要修改内容为：
```
  def execute(command, queue)
    @logger.debug? && @logger.debug("Running exec", :command => command)
    begin
	  @io = IO.popen(command)
	  fields = (@io.read).split(/\r\n|\n/)
	  puts fields
	  length = fields.length-1
      	for i in 0..length do 
		  if fields[i].include?':' then
			field = fields[i].split(':')
			newcommand = "redis-cli -c -h #{field[0]} -p #{field[1]} info"
			@io = IO.popen(newcommand)
			@codec.decode(@io.read) do |event|
			decorate(event)
			event.set("host", @hostname)
			event.set("command", newcommand)
			queue << event
		  end
		end
		
      end
    rescue StandardError => e
      @logger.error("Error while running command",
        :command => command, :e => e, :backtrace => e.backtrace)
    rescue Exception => e
      @logger.error("Exception while running command",
        :command => command, :e => e, :backtrace => e.backtrace)
    ensure
      stop
    end
  end
```
### 使用Demo
使用方式
```
redisexec {
command => "redis-cli -h 127.0.0.1 -p 6379 cluster nodes|grep master|awk '{print $2}'"
interval => 20
type => "info"
}
```
完整使用案例

将info 信息存储到 ElasticSerach中
```
input {
redisexec {
command => "redis-cli -h 127.0.0.1 -p 6379 cluster nodes|grep master|awk '{print $2}'"
interval => 20
type => "info"
}
}
filter {
        grok {
            match => {"command" => "redis-cli -c -h %{IP:node:} -p %{NUMBER:port}%{DATA:data}" }
      remove_field => [ "host" ]
        }
    ruby {
        code => "fields = event['message'].split(/\r\n|\n/)
        length = fields.length-1
        for i in 1..length do
          if fields[i].include?':' then
            field = fields[i].split(':')
            event[field[0]] = field[1].to_f
          end
        end
        "
        remove_field => [ "message" ]
    }
}
output {
#stdout {  codec => rubydebug }
elasticsearch {
hosts => ["127.0.0.1:9200"]
template_overwrite => true
index => "rediscluster-%{+YYYY.MM.dd}"
workers => 5
}
}
```
## 参考
1. http://www.cnblogs.com/xing901022/p/5259750.html
2. https://github.com/logstash-plugins?utf8=%E2%9C%93&query=example
3. https://www.elastic.co/guide/en/logstash/current/_how_to_write_a_logstash_input_plugin.html