# Django模板语言

## for循环
	{% for item in List %}
	    {% if item%}|{% empty %}
	    	{{ item }},
	    {% endif %}  
	{% endfor %}
	
	
| 变量 | 描述 |
| :-- | :-- |
| forloop.counter | 索引从 1 开始算 |
| forloop.counter0 | 索引从 0 开始算 |
| forloop.revcounter | 索引从最大长度到 1 |
| forloop.revcounter0 | 索引从最大长度到 0 |
| forloop.first | 当遍历的元素为第一项时为真 |
| forloop.last | 当遍历的元素为最后一项时为真 |
| forloop.parentloop | 用在嵌套的 for 循环中，获取上一层 for 循环的 forloop |


## 判断
 ==, !=, >=, <=, >, <
 
 and, or, not, in, not in
 
 {% if 'item' in List %}
 
## 过滤器
 |truncatewords:30 
 
### 验证用户
 {% if user.is_authenticated %}
 
 {% if perms.polls %}
 <p>You have permission to do something in the polls app.</p>
 {% if perms.polls.can_vote %}
  <p>You can vote!</p>
 {% endif %}
{% else %}
 <p>You don't have permission to do anything in the polls app.</p>
{% endif %}