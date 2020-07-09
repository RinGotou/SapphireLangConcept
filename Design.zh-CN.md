# Sapphire Language 基础设计

## 关于本语言
本语言为笔者在课余时间基于学习目的而设想并开发的实验性小型脚本语言, 脱胎于原本的旧语言项目.
该语言依然处于高度实验状态, 在各方面可能存在无法预料的变动, 且笔者对于编译原理的相关内容依然处于学习阶段(先开发的此项目, 后开始学习编译原理相关内容), 因此如有错误望加以指正.

本文档应与以下版本的解释器一起使用:
- Sapphire-Adhoc, 版本2.5 Alpha, 'Oratorio'

部分设计尚未实现, 在文档中有标注.

Kagami的很多设计主要来源于Lua, Ruby以及Visual Basic.

## (理想)设计目标
- 简明的语法, 并且提供一定的自我运行时修改能力
- 避免存在二义性的内容
- 直觉化, 在编写/阅读代码时能够更快地确定逻辑

## 基本约定
- 严格区分大小写
- 执行顺序为主启动脚本的从上至下顺序, 不存在主入口函数
- 默认使用词法作用域
- 不支持单个完整语句的换行书写. 请避免书写过长的单个表达式.

## 注释
本语言支持两种风格的注释:
### '\#'符号
这种注释仅可书写一行, 符号后所有字符均视为注释.
```
i = 1 #注释1
#注释2
```

### '=begin'与'=end'构成的多行注释
这种注释将'=begin'与'=end'标记之间的所有内容视为注释.
```
=begin
注释内容
=end
```

## 标识符

标识符用于与一个存在的Object进行关联使其保持一定的生命周期或便于访问.一个标识符应当遵循以下规则:

- 仅能存在大小写字母/数字/下划线
- 不能以数字开头

## 保留关键字
### 列表
(待补全)
### 不可作为Object对待的函数
(待补全)

## 平凡类型/字面量, 以及类型转换
### 平凡类型
有一部分与语言功能联系紧密不可分割的Object类型, 这些类型被称为"平凡类型"(Plain types).这些类型具有以下特点:
 - 使用浅拷贝策略
 - 可通过字面量/特定表达式生成
 - 可以跨二进制被解释器扩展程序访问(但是, 可被扩展有限制地访问的数组并不是平凡类型)
 - 在参加运算时具有固定的转换规则

### 强类型系统?弱类型系统?
笔者将本语言定义为强类型语言.
- 本语言不允许除平凡类型参加运算以外的任何情况发生自动类型转换
- 运算时的类型转换具有固定的转换规则/方向, 而且没有左右运算符不同所导致的转换结果不同

### 强制规定的运算时平凡类型间转换规则
### 空类型(null)
空类型为该语言中的一个特殊类型, 其内部实现指向空指针(nullptr/NULL, 与KPC使用的编译工具链实现有关).

以下行为会产生这种Object:
- 尝试 从没有进行显式return声明(结构体构造器除外) 的 函数 获取返回值 
- 通过内置类型构造函数null()生成
- 函数可选参数没有被赋值
空类型用于代表标识符没有与任何其他所有类型的Object产生关联或表示值为空的情况. 这是该语言唯一表示空值的方式.

## 语句分隔
### 同行书写多个语句
若需要将多个语句写在一行以保证美观或避免多个过短语句分行导致的信息密度过低, 可以使用半角分号在同一行内分割多个语句.
但需要注意到是, 每行代码的行尾不需要添加分号. 如果存在这类分号, KPC将会提示存在多余的分号.
```
i = 1; b = 2; c = 3 
```

### 括号表达式
括号表达式由两端的圆括号和使用逗号分隔的语句组成.
```
(i = 1, b = 2, c = 3)
```
此类表达式从左至右执行每一个语句, 值为最后一个语句所得到的值, *在此过程中产生的Object释放到该表达式所在作用域.*
*注意, 在条件判断/循环语句块的条件使用括号表达式时, 产生的Object会释放到语句块外部作用域而不是语句块的作用域, 务必注意.*

### 数组生成表达式
(该表达式可能考虑废弃或者使用其他设计代替.当前实现可能存在未知问题.)

数组生成表达式由两端的花括号和使用逗号分隔的语句组成.
```
arr =  { 1, 2, 3, plus(1, 3) }
```
该表达式将其中每一个语句的返回值按照从左至右的顺序组装成数组.

## Object与关联
Object可以视为每一个类型所对应的实体, 关联表达式则通过对Object关联一个标识符来使其可以被进行直接访问, 并能够保持一段的生命周期, 且在其所有浅拷贝副本均消亡之后进行内存的回收; 同时, 也可以对一个已经存在的标识符使用关联表达式使其关联一个新的Object.
```
i = 1 
println(i) # 此时输出的内容为1

i = 'hello again, world'
println(i) # 此时输出的内容为hello again, world
```

### 'local'关键字
'local'关键字可以用于标识关联表达式, 使其作用范围强制限定在本作用域内而不影响到外部作用域.
```
#样例1
i = 1

fn Output()
  local i = 2
  println(i)
end

Output()   # 此时输出为2
println(i) # 此时输出为1
```

### 'global'关键字
(未实现, 暂不能使用)'global'关键字可以强制关联表达式访问全局作用域. 当全局作用域不存在对应标识符时, 则会在*当前作用域*创建标识符并将Object与其进行关联.
```
i = 1

fn Output()
  local i = 2
  global i = 3
  println(i)
end

Output()   # 此时输出为2
println(i) # 此时输出为3
```

### 'ref'关键字
(未实现, 暂不能使用)'ref'关键字可以用于创建一个已知Object的引用(于下面的直接关联新关键字不同, 引用不会导致Object的共享副本增加).

### 'shadow'关键字
(未实现, 暂不能使用)'__rcc'关键字可以直接将指定的Object直接添加新的标识符关联. 当所有的关键字都离开作用域时, 该Object才会消亡并进行内存回收. 

*不推荐在常规代码的编写中使用这个修饰关键字, 但是在特定逻辑实现以及对KPC扩展的包装的时候可以使用该关键字来实现一些特性*.

## Object的移交
可以使用左向箭头'<-'将一个Object与已经关联的标识符进行解绑, 然后与新的标识符进行关联, 原标识符则会与一个类型为null的Object进行关联. 移交表达式可以使用关联表达式的修饰关键字, 
行为基本相同.
```
i = 0
j <- i

println(null_obj(i)) # 输出true
println(j) # 输出0
```

## 生命周期, 浅拷贝(共享副本)/深拷贝(完全副本)
### 生命周期, 消亡
一个Object从被生成直到被语言执行引擎回收为止的存在时间称为一个生命周期.

当前的主要实现(KPC)使用C++ STL的引用计数设施来实现Object的内存管理和回收, 当一个受引擎管理的Object所关联的引用计数指针归零时, 这个Object会被回收, 称为消亡.

### 有关共享副本和完全副本
多个共享副本会共享一个Object的内部实现实例, 对其中一个共享副本进行操作所有共享副本的值都会受到影响.

完全副本是通过关联表达式使用已存在的关键字对其他关键字进行赋值, 或者一些内置函数, 且该类型的复制策略为深拷贝时, 生成的与原Object值相同的副本, 是一个新的副本, 对完全副本做出的修改不会对原Object产生影响.

### 复制策略
采用浅拷贝策略的内置类型在使用关联表达式时, 产生的新Object为共享副本.

采用深拷贝策略的内置类型在使用关联表达式时, 产生的新Object为完全副本.

通过结构体定义的类型视其具体实现而定.

所有的平凡类型(Plain types)均为深拷贝策略. 其余类型参考内置类型文档. (待补全)

### 引用
引用的含义与大部分其他语言的定义相同. 一个引用会将所有的消息转发给其所指向的Object, 行为上与其指向的Object相同.
在某些时候可能存在引用所指向的Object已经消亡但是引用仍然存在的情况, 这种引用称为"悬空引用". 在语言层尝试操作这些引用时, KPC会拒绝执行并且停止整个脚本的运行.

(KPC的当前实现可能在某些位置仍未能够拦截对悬空引用的访问, 有待后续更多的测试用例来发现这些未实现位置.)

## 函数
函数由函数声明/函数体/结尾的'end'关键字组成. 

所有函数均具有返回值, 在未进行显式return的函数默认返回类型为null的Object, 结构体构造器不能书写return语句. 不支持函数重载. 

函数的返回值行为类似于移交, Object不会触发深拷贝策略而是直接与标识符进行关联, 也就是说无需对返回值使用移交表达式. 

函数的参数为引用形式.

本语言不支持函数重载, 如果同一级作用域内存在重复名称的函数定义, 只有最靠前的一个会生效.

(已知问题: KPC在运行直接执行类型为函数的返回值的代码时, 会导致该Object提前消亡, 从而造成解释器段错误/非法内存访问)
```
fn greet(name)
  println('hello, ' + name)
end

fn plus(a, b)
  return a + b
end

greet('rin')
result = plus(1, 2)
println(result)
```
函数也是Object, 具有大部分Object的特征和行为. 一部分函数不能作为Object进行操作, 具体参照"保留关键字"小节.

### 嵌套函数
函数内依然可以声明函数, 且内部函数会对外部一层函数的作用域进行Object捕捉.

(已知问题:目前会对作用域内所有Object进行捕捉而不进行调用的辨别, 需修改实现.)
```
fn Generate(a)
  fn Func(b)
    return a + b
  end
end

func = Generate(3)
println(func(4))    # 输出为7
```

### 可选参数
可选参数在调用时可以不进行赋值, 其在不被赋值时则被置为null类型Object. 不可以在函数声明内设置可选参数的默认值.
```
fn Test(a, optional b)
  if null_obj(b)
    return a
  end

  return a + b
end

println(Test(1))     # 输出1
println(Test(1, 2))  # 输出3
```
可选参数只能出现在常规参数之后, 数量不受限制, 不可以与变长参数同时存在.

### 变长参数
变长参数可以将常规参数之后的参数组装到声明为变长参数的标识符所关联的数组内. 当没有可以组装到数组内的参数时, 数组为空. 变长参数同一个声明只能出现一个.
```
fn Test(a, variable arr)
  result = a
  for number in arr
    result = result + number
  end

  return result
end

println(Test(1, 2))
println(Test(1, 2, 3, 4))
```
变长参数不可以与可选参数同时存在.

### 参数推断顺序
与C/C++不同, 该语言的参数推断顺序为*从左至右*. 这是故意为之的.
```
fn print(a, b, c)
  println(a + b + c)
end

fn test(value)
  println(value)
  return value
end

print(test(1), test(2), test(3))

=begin
以上语句会输出如下结果:
1
2
3
6
=end
```

### 返回值约束
当需要把返回值固定在特定的类型时, 可以通过在函数头部添加约束类型的名称进行限制. (暂不考虑同时约束到多个类型)
```
fn SampleFunc(value) -> int 
  attribute result
  if value == 1; result = 'hello'
  elif value == 2; result = 2
  end
  return result
end

fn SampleWithoutConstraint()
  println('Sample')
end

SampleWithoutConstraint()
SampleFunc(2)
SampleFunc(1)
```

当上述代码执行到最后一个调用时, 由于返回类型不匹配, 解释器将直接产生错误.

## 条件判断
### if - elif - else - end
使用if语句块可以在判断某条件是否为真之后执行一定的操作. if/elif后隔空格书写判断语句, 并且判断语句的返回值必须能够转化为布尔值形式, 具体可参考"字面量"小节.
注意, if之后有无括号将会存在区别, 具体请参照"括号表达式"小节.
```
str = input('enter a string:')
if str == 'hello'
  println('world')
end
```
可以使用elif在语句块内添加多个判断条件, 并且可以使用else处理条件不满足时的情况.
```
if str == 'hello'
  println('world')
elif str == 'world'
  println('hello')
else
  println('hello again, world')
end
```
### case - when - else - end
case语句块可以实现对同一个Object进行多个值的比对并执行相应的行为, 该语句块也可以使用else处理所有比对条件均不满足时的情况. 目前只能用于平凡类型(Plain types).

具体内容请参照内置类型文档.(待补全)

```
case value
when 1
  # actions
when 2
  # actions
when 'nothing'
  # actions
else
  # actions
end
```

## 循环
### while表达式
while语句块会不断对提供的条件进行检测, 当条件为false时退出循环.
```
str = null()

while str != 'hello'
 str = input('>')
end
```

### for ... in ... 表达式
该语句块会对'in'之后的表达式所提供的Object进行尝试遍历操作, 且Object对应的类型必须满足如下条件:
- 该类型包含head()/tail()/empty()方法
- 上述方法返回的Object类型必须包含obj()/step_forward()/__compare()方法

遍历时可通过'in'之前的标识符访问元素. 标识符所关联的元素是否为完全副本/共享副本使其实现而定, 具体参考内置类型文档(待补全).

```
# arr = .........

for unit in arr
  println(unit)
end
```

## 运算
### 数学运算
本语言支持以下的数学运算符号:

### 逻辑运算
本语言支持以下的数学运算符号:

### 字符串在运算时的行为
字符串不支持全部的数学/逻辑运算符, 而且行为和常规的运算存在一些不同. 这些行为可以参考平凡类型在参与运算时的转换规则.



## 结构体/模块
本语言中的结构体能够根据编码者的设计产生用户定义的类型的实例, 类似于其他语言中类的行为.

模块为带有限制的结构体, 其不能用于编写用户定义类型, 也不能作为类生产用户定义类型对应的实例, 即使在其中定义了initializer函数.

模块和结构体也是Object, 具备Object的大部分行为.

结构体/模块目前不支持嵌套.

### 结构体(struct)
通过struct语句块可以定义一个新的结构体.
```
struct character
  # contents
end
```
#### 结构体成员
可以通过在结构体语句块内书写关联表达式和函数来定义该结构体的成员.
结构体本级语句块内只能存在函数语句块和关联表达式. (是的, 不能使用移交表达式)
```
struct character
  name = null()

  fn greet()
    println(name)
  end
end
```

在KPC执行到结构体语句块时, 会触发其结构对应的行为. 也就是说会出现如下现象:
```
fn InitName()
  println('Init character name')
  return '(none)'
end

struct character
  name = InitName()

  fn greet()
    println(name)
  end
end

# 假设引擎当前执行到这个位置

=begin
输出:
Init character name
=end
```

可以通过半角圆点'.'访问结构体内成员.
```
struct character
  name = '(noname)'

  fn greet()
    println()
  end
end

character.greet()

=begin
输出:
(noname)
=end
```
#### 构造器/构造器函数
结构体内名为initializer的函数被视为构造器. 结构体可以通过构造器产生新的Object, 且Object的类型为结构体在定义时所使用的标识符.

*实际上, 这个标识符可以在运行时修改, 试试使用结构体的members()方法查看返回的内容吧.*

*笔者并不希望你这么做.*

构造器可以通过普通结构体成员的方式进行访问, 也可以通过结构体标识符进行访问.

构造器内通过'me'标识符可以访问正在构建的结构体实例, 以进行操作.
```
struct character
  name = null()

  fn initializer(name)
    me.name = name
  end

  fn greet()
    println('My name is ' + me.name)
  end
end

tsumugi = character('tsumugi wenders')
# tsumugi = character.initalizer('tsumugi wenders')
tsumugi.greet()
```

构造器目前不能使用return. 其默认返回值就是'me'所关联的Object, 返回操作由引擎隐式完成.

### 模块(module)
模块在本语言中提供了用于Mixin支持的基础设施.
以下的语句块定义了一个模块.
```
module base
  # content
end
```
与结构体相同, 模块内部可以定义模块成员, 此处不再赘述.

#### 与结构体区别
- 模块不可以继承其他的结构体, 也不能被继承.
- 模块不可以生成实例, 即使它存在initializer函数. (但是在混入结构体之后可以作为结构体的构造器函数存在)
### 一些细节
结构体可以被修改. 但是笔者不希望这一特性被滥用, 请用在必须动态修改类型定义/对结构体进行临时修补的场合.

## 单继承与Mixin
本语言不支持常规意义上的多重继承, 但支持单继承和用于弥补单继承不足的mixin机制.

笔者将本语言定义为"部分支持面向对象", 因为本语言目前不支持结构体成员的访问限制调整, 所有的结构体成员实际上都可以被外部访问, 需要通过撰写代码时的规范来保证封装.
### 结构体的继承
可以通过在结构体块的头部书写基结构体来继承一个已经存在的结构体.
```
struct sub < base
  # content
end
```
(暂时不支持被保存在容器内的结构体, 但是可以通过创建共享副本/创建引用的方式进行间接访问. 不过, 笔者可能不打算做这方面的支持, 因为这会让结构体头变得十分混乱而且行为难以预测.)

派生结构体具有基结构体的行为.
```
struct base
  fn act()
    println('acting')
  end
end

struct sub < base
end

sub.act()
=begin
输出:
acting
=end
```

可以在派生结构体中书写与基结构体中名称相同的方法函数进行覆盖.
```
struct base
  fn act()
    println('acting')
  end
end

struct sub < base
  fn act()
    println('acting(sub)')
  end
end

sub.act()
=begin
输出:
acting(sub)
=end
```
(目前没有实现直接在派生结构体中访问被覆盖的方法的途径, 待后续版本添加)

有关如何在派生结构体中访问基结构体的构造器可以参照"对基结构体构造器的访问"小节.
### 对结构体使用'include'进行模块混入
在结构体中使用include语句可以对结构体进行模块混入, 被混入的结构体具有混入的模块的行为.

如果同时书写了多个include语句进行模块混入, 越靠下的模块具有越高的混入优先级, 即高优先级的模块会覆盖低优先级的模块的行为.
```
module feature0
  fn jump()
    println('jumping')
  end
end

module feature1
  fn run()
    println('running')
  end
end

struct base
  fn act()
    println('acting')
  end
end

struct sub < base
  include feature0
  include feature1
end

sub.jump()
sub.run()
sub.act()
```
### 对基结构体构造器的访问
在派生结构体的构造器方法中, 可以使用'super'关键字对其所继承的基结构体的构造器进行访问. 这个关键字只能在构造器内使用.
```
struct base
  value = null()

  fn initializer(value)
    println('Super struct:' + value)
    me.value = value
  end
end

struct sub < base
  fn initializer()
    super('from sub')
    println('Sub struct')
    println(me.value)
  end
end

sample = sub()
```
## 反射接口
### 类型判断
### 行为判断
### 继承关系判断
## 结构体约束设施: 概论(Concept)
### 成员约束
### 类型约束
### 约束方法函数
### 在定义结构体时进行约束
### 对实例使用约束
## 多脚本文件
## 有关指针
是的, 这里提供两种指针: pointer与function_pointer. 但是, *任何内置类型/用户定义类型的Object都不可以在脚本层面被获取指针, 且扩展支持库(kagami-project/assist)大多数时候也只能通过描述器(Descriptor)来访问Object而不是直接使用指针.*

C++显然没有一个在同平台内统一的ABI, 目前的扩展支持库实现暂时只能对有限的内置类型进行转换后访问和操作, 也不能够直接通过扩展来添加新的类型. 因此, 笔者决定添加这两个类型, 并且通过对扩展侧已提供合适的接口, 且符合当前平台C ABI的前提下暂时保管指针用于对扩展的功能访问, 使得编写实现库包装的脚本能够编写起来更容易.

具体参照扩展库文档和内置类型文档(待补全).

