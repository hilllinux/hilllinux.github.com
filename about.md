---
layout: page
title: "About"
description: "业余爱好的点滴记录，同时作为配置备份的存储地，好记性不如滥笔头。"
---

#### Who Am I
 
```ruby
class Profile
  attr_accessor :name,:company,email
  attr_accessor :skill,:description
  def initialize
      :name    = "王宋清"
      :company = "Huawei Technologies"
      :skill   = ["Ruby", "Web Development", "DevOps", "Computer Security"] 
      :email   = "songqing.wang (at) hotmail.com"
      :description = "新手白帽一枚"
  end
end
```

#### ChangeLog
- 2014年10月: 开始写blog


{% include comments.html %}
