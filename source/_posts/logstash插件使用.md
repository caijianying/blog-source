---
title: Filter plugins
date: 2016-10-24 21:24:37
tags: research
categories:
- elasticsearch
---
# Filter plugins
1. grok
```
grok {
match => {"command" => "redis-cli -c -h %{IP:node:} -p %{NUMBER:port}%{DATA:data}" }
remove_field => [ "host" ]
}
```
2. ruby

功能描述：将redis info 信息格式化按字段输出
```
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
```
3. mutate

功能描述：字段类型指定
```
filter {
        mutate {
            convert => {"latestResponse" => "integer"}
            convert => {"cacheHit" => "string"}
            convert => {"cacheRate" => "float"}
        }
        
}
```
# Output plugins