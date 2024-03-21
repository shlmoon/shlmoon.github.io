---
layout: post
title: nginx openresty
date: 2019-12-03 09:10:00
tags: openresty
---

#### openresty install

[https://blog.csdn.net/fyq201749/article/details/80981350](https://blog.csdn.net/fyq201749/article/details/80981350)
<!-- more -->
#### openresty sample

redirect.lua

```
local cjson = require "cjson.safe"

local function read_from_file(file_name)
  local f = assert(io.open(file_name, "r"))
  local string = f:read("*all")
  f:close()
  return string
end

ngx.req.read_body()
local body = ngx.req.get_body_data()
ngx.log(ngx.INFO, "params.\n", body, "\n")

if (body == nil) then
  local body_file = ngx.req.get_body_file()
  if body_file then
    body = read_from_file(body_file)
  else
    ngx.status = 500
    ngx.say("error")
    return
  end
end

local body_parameters, err = cjson.decode(ngx.req.get_body_data())
if err then
 body_parameters = {}
end

if ("keywords" == body_parameters["keywords"]) then
  ngx.var.proxy_host="proxy_pass"
  ngx.var.proxy_http='http'
else
  ngx.var.proxy_host="proxy_pass"
  ngx.var.proxy_http='https'
end

```

nginx config

```
location / {
    set $proxy_host '';
    set $proxy_http '';
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    rewrite_by_lua_file /file_path/redirect.lua;
    proxy_pass $proxy_http://$proxy_host;
}
```
