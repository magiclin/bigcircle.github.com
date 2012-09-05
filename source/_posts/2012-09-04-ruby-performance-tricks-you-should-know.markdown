---
layout: post
title: "一些 Ruby 性能技巧"
date: 2012-09-04 19:57
comments: true
categories: [ruby]
keywords: ruby performance tricks
description: ruby performance tricks you should know
---
前两天看到这个 [ruby performance tricks](http://greyblake.com/blog/2012/09/02/ruby-perfomance-tricks/) 作者总结的不错，但是很多人不喜欢看 e 文，刚好学习顺便翻译下   
由于现在基本上都已经过渡到 1.9.3，测试数据均在 1.9.3 版本下测试
<!--more-->

######不要在控制流中滥用rescue捕获异常

```ruby
require 'benchmark'

class Obj
  def with_condition
    respond_to?(:mythical_method) ? self.mythical_method : nil
  end

  def with_rescue
    self.mythical_method
  rescue NoMethodError
    nil
  end
end

obj = Obj.new
N = 10_000_000

puts RUBY_DESCRIPTION

Benchmark.bm(15, "rescue/condition") do |x|
  rescue_report     = x.report("rescue:")    { N.times { obj.with_rescue  } }
  condition_report  = x.report("condition:") { N.times { obj.with_if      } }
  [rescue_report / condition_report]
end
```

跑分数据可以看出使用了rescue，性能损失超过43倍

```
ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-linux]
                        user     system      total        real
rescue:           111.530000   2.650000 114.180000 (115.837103)
condition:          2.620000   0.010000   2.630000 (  2.633154)
rescue/condition:  42.568702 265.000000        NaN ( 43.991767)
```

google 了下发现不光 Ruby 会有这个问题，C，C++，Java 中也都会有很大的性能问题，原因大概是因为异常捕获这个操作本身就很慢，[这](http://www.simonecarletti.com/blog/2010/01/how-slow-are-ruby-exceptions/) 有一份比较详细的测试，对比使用 exception 在速度上的劣势。当然该捕捉的时候还是要捕捉的，只是不要滥用

######使用 << 拼接字符串

这个tip可能大家都听过，估计平时也都是这么用的，为什么用 << 不要用 += 呢，看这个测试

```
require 'benchmark'

N = 1000
BASIC_LENGTH = 10

5.times do |factor|
  length = BASIC_LENGTH * (10 ** factor)
  puts "_" * 60 + "\nLENGTH: #{length}"

  Benchmark.bm(10, '+= VS <<') do |x|
    concat_report = x.report("+=")  do
      str1 = ""
      str2 = "s" * length
      N.times { str1 += str2 }
    end

    modify_report = x.report("<<")  do
      str1 = "s"
      str2 = "s" * length
      N.times { str1 << str2 }
    end

    [concat_report / modify_report]
  end
end
```

结果如下 

```
____________________________________________________________
LENGTH: 10
                 user     system      total        real
+=           0.000000   0.000000   0.000000 (  0.004671)
<<           0.000000   0.000000   0.000000 (  0.000176)
+= VS <<          NaN        NaN        NaN ( 26.508796)
____________________________________________________________
LENGTH: 100
                 user     system      total        real
+=           0.020000   0.000000   0.020000 (  0.022995)
<<           0.000000   0.000000   0.000000 (  0.000226)
+= VS <<          Inf        NaN        NaN (101.845829)
____________________________________________________________
LENGTH: 1000
                 user     system      total        real
+=           0.270000   0.120000   0.390000 (  0.390888)
<<           0.000000   0.000000   0.000000 (  0.001730)
+= VS <<          Inf        Inf        NaN (225.920077)
____________________________________________________________
LENGTH: 10000
                 user     system      total        real
+=           3.660000   1.570000   5.230000 (  5.233861)
<<           0.000000   0.010000   0.010000 (  0.015099)
+= VS <<          Inf 157.000000        NaN (346.629692)
____________________________________________________________
LENGTH: 100000
                 user     system      total        real
+=          31.270000  16.990000  48.260000 ( 48.328511)
<<           0.050000   0.050000   0.100000 (  0.105993)
+= VS <<   625.400000 339.800000        NaN (455.961373)
```

从结果可以看出，字符串比较短时就差25倍，随着字符串长度的增加，性能差距也越来越大   
为什么作用相同的操作性能方便差距这么大呢，看这个例子

```
str1 = "first"
str2 = "second"
str1.object_id       # => 16241320

str1 += str2    # str1 = str1 + str2
str1.object_id  # => 16241240, id is changed

str1 << str2
str1.object_id  # => 16241240, id is the same
```

当使用 += 时会在内存中多复制一份原始字符串，生成 str1 + str2 结果的临时object覆盖掉 str1,  因此操作完之后 str1 的 object_id 发生了变化，而 << 是直接在原 str1 上进行操作，操作完之后 object_id 没有发生变化

+= 还会导致内存中存在大量多余的string object，GC 触发也会消耗掉许多时间

######注意使用迭代

假设你需要写个函数把数组转换成一个 hash, 就像这样

```
func([1, 2, 3])  # => {1 => 1, 2 => 2, 3 => 3}
```

一个可以解决问题的做法

```
def func(array)
  array.inject({}) { |h, e| h.merge(e => e) }
end
```

这个方法的时间复杂度是 O(n2)，当数组很大时速度会非常慢，稍作调整，时间复杂度 O(n)

```
def func(array)
  array.inject({}) { |h, e| h[e] = e; h }
end
```

看下测试时间

```
require 'benchmark'

def n_func(array)
  array.inject({}) { |h, e| h[e] = e; h }
end

def n2_func(array)
  array.inject({}) { |h, e| h.merge(e => e) }
end

BASE_SIZE = 10

4.times do |factor|
  size   = BASE_SIZE * (10 ** factor)
  params = (0..size).to_a
  puts "_" * 60 + "\nSIZE: #{size}"
  Benchmark.bm(10) do |x|
    x.report("O(n)" ) { n_func(params)  }
    x.report("O(n2)") { n2_func(params) }
  end
end
```

从时间上看虽然都消耗不大，但是还是有明显性能差距的，记住不要用迭代做大规模计算

```
SIZE: 10
                user     system      total        real
O(n)        0.000000   0.000000   0.000000 (  0.000014)
O(n2)       0.000000   0.000000   0.000000 (  0.000033)
____________________________________________________________
SIZE: 100
                user     system      total        real
O(n)        0.000000   0.000000   0.000000 (  0.000043)
O(n2)       0.000000   0.000000   0.000000 (  0.001070)
____________________________________________________________
SIZE: 1000
                user     system      total        real
O(n)        0.000000   0.000000   0.000000 (  0.000347)
O(n2)       0.130000   0.000000   0.130000 (  0.127638)
____________________________________________________________
SIZE: 10000
                user     system      total        real
O(n)        0.020000   0.000000   0.020000 (  0.019067)
O(n2)      17.850000   0.080000  17.930000 ( 17.983827)
```

######使用　method!

很多时候直接用 method 和 使用 method! 都在做同样的事，不同的是用 method! 不会重新复制一份副本，使用 method! 速度会快点

```
require 'benchmark'

def merge!(array)
  array.inject({}) { |h, e| h.merge!(e => e) }
end

def merge(array)
  array.inject({}) { |h, e| h.merge(e => e) }
end

N = 10_000
array = (0..N).to_a

Benchmark.bm(10) do |x|
  x.report("merge!") { merge!(array) }
  x.report("merge")  { merge(array)  }
end
```

这个差距有点大

```
                 user     system      total        real
merge!       0.010000   0.000000   0.010000 (  0.011370)
merge       17.710000   0.000000  17.710000 ( 17.840856)
```

###### 使用实例变量

直接访问实例变量是通过 accessor 访问速度的2倍

```
require 'benchmark'

class Metric
  attr_accessor :var

  def initialize(n)
    @n   = n
    @var = 22
  end

  def run
    Benchmark.bm(10) do |x|
      x.report("@var") { @n.times { @var } }
      x.report("var" ) { @n.times { var  } }
      x.report("@var =")     { @n.times {|i| @var = i     } }
      x.report("self.var =") { @n.times {|i| self.var = i } }
    end
  end
end

metric = Metric.new(100_000_000)
metric.run
```

```
                 user     system      total        real
@var         6.980000   0.010000   6.990000 (  7.193725)
var         13.040000   0.000000  13.040000 ( 13.131711)
@var =       7.960000   0.000000   7.960000 (  8.242603)
self.var =  14.910000   0.010000  14.920000 ( 15.960125)
```

######平行的任务速度比分开执行慢

```
require 'benchmark'

N = 10_000_000

Benchmark.bm(15) do |x|
  x.report('parallel') do
    N.times do
      a, b = 10, 20
    end
  end

  x.report('consequentially') do |x|
    N.times do
      a = 10
      b = 20
    end
  end
end
```

速度差距1倍以上，我平时就喜欢为了少写几行平行赋值，看来这个习惯要改下啦

```
                      user     system      total        real
parallel          1.900000   0.000000   1.900000 (  1.928063)
consequentially   0.880000   0.000000   0.880000 (  0.879675)
```

######动态定义方法比较 define_method 和 class_eval

在动态定义方法时是用 class_eval string 快还是 defing_method 快呢

```
require 'benchmark'

class Metric
  N = 1_000_000

  def self.class_eval_with_string
    N.times do |i|
      class_eval(<<-eorb, __FILE__, __LINE__ + 1)
        def smeth_#{i}
          #{i}
        end
      eorb
    end
  end

  def self.with_define_method
    N.times do |i|
      define_method("dmeth_#{i}") do
        i
      end
    end
  end
end

Benchmark.bm(22) do |x|
  x.report("class_eval with string") { Metric.class_eval_with_string }
  x.report("define_method")          { Metric.with_define_method     }

  metric = Metric.new
  x.report("string method")  { Metric::N.times { metric.smeth_1 } }
  x.report("dynamic method") { Metric::N.times { metric.dmeth_1 } }
end
```

结果显示 define_method 比 class_eval string 快3倍以上，class_eval 虽然在方法生成上慢却应该优先使用，因为它生成的方法执行起来更快

```
                             user     system      total        real
class_eval with string 219.840000   0.720000 220.560000 (221.933074)
define_method           61.280000   0.240000  61.520000 ( 62.070911)
string method            0.110000   0.000000   0.110000 (  0.111433)
dynamic method           0.150000   0.000000   0.150000 (  0.156537)
```

俗话说，不积跬步，无以至千里。虽然这些都是一些性能方面的小技巧，但累积多了也能对性能有很大的优化