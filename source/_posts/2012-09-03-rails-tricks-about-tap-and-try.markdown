---
layout: post
title: "Rails 技巧之 tap & try"
date: 2012-09-03 14:38
comments: true
categories: [rails]
keywords: rails trick tap try
description: rails tricks about tap try
---
tap 和 try 是 Rails 开发过程中两个很常用的方法，在调试和写出简洁代码上有着不错的发挥

从 tap 的 [Api](http://www.ruby-doc.org/core-1.9.3/Object.html#method-i-tap) 可以看出，tap 是 Object 的 instance_method，传递 self 给一个 block，最后返回 self.
<!--more-->

```ruby
1.9.3p194 :015 > Object.instance_methods.grep /tap/
 => [:tap]

def tap		#tap 源码实现
  yield self
  self
end
```

**用途一：调试**。当你使用链式方法发生错误时，如果需要测试这个过程中哪出了问题，一般的做法是拆断这个方法，设置中间变量，接着测试中间变量是否正确，如果正确，则把变量换个地方继续测试，方法很长时就没法测了，当然 Ruby 为你想好了，有着优雅又简便的实现

```
%w(x y z).push('a').shift.upcase.next # => "B"

%w(x y z).push('a').shift.tap {|x| p x }.upcase.next
```

输出每步的结果看看

```
(1..10)                .tap {|x| puts "original: #{x.inspect}"}
  .to_a                .tap {|x| puts "array: #{x.inspect}"}
  .select {|x| x%2==0} .tap {|x| puts "evens: #{x.inspect}"}
  .map { |x| x*x }     .tap {|x| puts "squares: #{x.inspect}"}
```

**用途二：简化代码**。我们构建一个方法想返回一个 String / Array / Hash 之类结果，一般的做法是先定义一个变量，结果把运算结果赋值给这个变量，接着返回变量，用 tap 一步搞定，其实就是源码意思的实现

```
[].tap {|i| i << "abc"}
''.tap {|i| i << do_some_thing }
```

try 可以让你调用对象方法时不用担心对象是nil并抛出异常，依旧先看源码(rails 3.2.8)

```
def try(*a, &b)
  if a.empty? && block_given?
    yield self
  else
    public_send(*a, &b) if respond_to?(a.first)
  end
end

def try(*args)
  nil
end
```

上面一个try是对 Object 的扩展，下面的是对 NilClass 的扩展，这就是为什么会nil不会抛出异常的原因，[源码](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/object/try.rb) .    
try 如果只接受 block 则传递 self 给 block，返回 block 执行后的结果，否则就执行
[public_send](http://www.ruby-doc.org/core-1.9.3/Object.html#method-i-public_send)，public_send 与 send 的不同之处，public_send 只会 call public_method，看例子

```
@person && @persion.name
@person ? @person.name : nil
```

用 try 改写

```
@person.try(:name)
@person.try { |p| "#{p.first_name} #{p.last_name}" }
```

try 同样能接受参数、block，

```
Person.try(:find, 1)
@people.try(:collect) {|p| p.name}
```

通过源码可以看出 tap 和 try 两个小工具意思都非常明白，用起来也很简单方便，实乃居家简化好帮手

