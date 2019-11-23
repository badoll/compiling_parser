# 编译原理作业
### 环境
Apple clang version 11.0.0 (clang-1100.0.33.12)
### 语言
C++11
### 用法
```shell
make
bin/main testfile
```
# 词法分析
### 目的
分析文本中是否有非法词语。
### 实现
使用lexical_parser类逐行分析文本，每一行文本读取到string中，然后一个字符一个字符分析，合法则添加到words向量中，非法则报错，然后跳过分析下一个词语。
# 语法分析
### 目的
分析简单的赋值语句
```
begin
x := 11 ; y := (x+1*y)+(1);
end
```
### 实现
递归下降的语法分析。用syntactic_parser类继承lexical_parser类，调用继承过来的scan函数来一个个读取词语，读取的同时进行词法分析，分析完没有错误就再进行语法分析，有词法错误就报错然后返回一个带非法标志的词语。
语法分析使用简单的错误恢复方法，若缺少赋值号，则循环取下一个词语直到遇到分号（语句结束符）或变量（语句开始符），若循环结束前没有赋值号，则报错缺少赋值号，然后开始分析下一条语句。若匹配到赋值号，则将前面出现的词语报错，然后开始分析赋值号后面的表达式。缺少分号的错误分析方法同赋值号。
### 测试数据
- 样例1
```
  begin
  x  11 ; y := (x+1*y)+(1);
  end
```

```
Words: (1,begin) (10,x) (11,11) (27,;) (10,y) (17,:=) (28,() (10,x) (13,+) (11,1) (15,*) (10,y) (29,)) (13,+) (28,() (11,1) (29,)) (27,;) (6,end) 
ERROR:
Line 2: lack of ':='
```
- 样例2
```
begin
x  11 ; y := (x+1*y)+(1);
end
```
```
Words: (1,begin) (10,x) (11,11) (27,;) (10,y) (17,:=) (28,() (10,x) (13,+) (11,1) (15,*) (10,y) (29,)) (13,+) (28,() (11,1) (29,)) (27,;) (6,end) 
ERROR:
Line 2: lack of ':='
```
- 样例3
```
begin
x := 11 ; y := (x+1*y)();
end
```
```
Words: (1,begin) (10,x) (17,:=) (11,11) (27,;) (10,y) (17,:=) (28,() (10,x) (13,+) (11,1) (15,*) (10,y) (29,)) (28,() (29,)) (27,;) (6,end) 
ERROR:
Line 2: error symbol ()
```
- 样例4
```
begin
x := 11 ; y := );
end
```
```
Words: (1,begin) (10,x) (17,:=) (11,11) (27,;) (10,y) (17,:=) (29,)) (27,;) (6,end) 
ERROR:
Line 2: lack of factor
Line 2: error symbol )
```
- 样例5
```
begin
x := 11 ; y)) := (x+1*y)+(1);
end
```
```
Words: (1,begin) (10,x) (17,:=) (11,11) (27,;) (10,y) (29,)) (29,)) (17,:=) (28,() (10,x) (13,+) (11,1) (15,*) (10,y) (29,)) (13,+) (28,() (11,1) (29,)) (27,;) (6,end) 
ERROR:
Line 2: error symbol ))
```
# 语义分析
### 目的
对简单赋值语句生成四元式和三地址中间代码。
### 实现
若一个赋值语句语法分析没有错误，则在语法分析的递归中将一个表达式中的元素（包括操作数、运算符和暂时存放运算结果的一个中间元素）加入到四元组（tuple）中，最后再将表达式赋值号左边的变量和赋值号右边最终的中间元素（如果没有复杂运算则是一个变量或数字）放入四元组中。
### 测试数据
```
begin
a := 2 + 3 * 4; x := (a + b)/c;
end
```
```
Words: (1,begin) (10,a) (17,:=) (11,2) (13,+) (11,3) (15,*) (11,4) (27,;) (10,x) (17,:=) (28,() (10,a) (13,+) (10,b) (29,)) (16,/) (10,c) (27,;) (6,end) 

success

Intermediate Code:
T1 = 3 * 4
T2 = 2 + T1
a = T2  
T3 = a + b
T4 = T3 / c
x = T4
```
