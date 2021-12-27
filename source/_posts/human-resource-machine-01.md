---
title: 人力资源机器 (Human Resource Machine) 01 - 15
date: 2021-6-18 00:00:01
categories: Games
tags:
  - Games
  - Programming
  - Learn

---

>《人力资源机器》是一款模拟编程的游戏，适合用于学习编程的基本思维，该系列包含了这个游戏的个人解答和思路。可直接复制下面内容，进入游戏相应的关卡之后，点击右侧代码编辑栏下方的粘贴即可看到对应的运行代码，然后通过运行来看一下实际的执行顺序。

<!-- more -->

>强迫症诊断中心说明：包含两个指标，优化诊断指的是用到的代码行数，效率诊断则是实际运行的平均代码步数。需要注意的是，代码行数最少不等于效率最高，具体可以看 02 的思路。说明在某些情况下，为了效率可以牺牲代码行数，当然大部分情况下这并不需要我们手动去做，编译器已经帮我们进行了优化。（如果有两个解答，第一个解答为行数优化，第二个为效率优化）

## 01 收发室

* 目标优化：`6`，目标效率：`6`
* 实际优化：`6`，实际效率：`6`

```
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
```
思路：基本的输入输出操作，重复 3 次即可。

## 02 繁忙的收发室

* 目标优化：`3`，目标效率：`25`
* 实际优化：`3`，实际效率：`20`

```
a:
    INBOX   
    OUTBOX  
    JUMP     a
```
思路：增加了 JUMP 指令来进行循环的操作，行数减少至 3 行。

```
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
    INBOX   
    OUTBOX  
```
思路：JUMP 跳转循环解开为只有输入输出，减少了每一次的 JUMP，最终得到了实际执行步骤只有 10 * 2  的结果，比起 JUMP 循环的 3 * 10 行的代码少的多。

## 03 复印楼层

* 目标优化：`6`，目标效率：`6`
* 实际优化：`6`，实际效率：`6`

```
    COPYFROM 4
    OUTBOX  
    COPYFROM 0
    OUTBOX  
    COPYFROM 3
    OUTBOX  
```
思路：新增了 COPYFROM 指令，直接拷贝 3 并输出 3 次即可。

## 04 扰码处理器

* 目标优化：`7`，目标效率：`21`
* 实际优化：`7`，实际效率：`21`

```
a:
    INBOX   
    COPYTO   0
    INBOX   
    OUTBOX  
    COPYFROM 0
    OUTBOX  
    JUMP     a
```
思路：以相反的顺序输出，因此第二次输入不需要 COPYTO，直接输出即可。

## 05 咖啡时间（略）

## 06 多雨之夏

* 目标优化：`6`，目标效率：`24`
* 实际优化：`6`，实际效率：`24`

```
a:
    INBOX   
    COPYTO   0
    INBOX   
    ADD      0
    OUTBOX  
    JUMP     a
```

思路：新增了 ADD 指令，用手中数字加地板的，所以第 2 数字依然不需要拷贝到地板，第 2 个数字拿到手上直接相加并输出即可。


## 07 零扑灭行动

* 目标优化：`4`，目标效率：`23`
* 实际优化：`4`，实际效率：`23`

```
a:
b:
    INBOX   
    JUMPZ    b
    OUTBOX  
    JUMP     a
```

思路：新增了 JUMPZ 指令，为 0 则跳转。


## 08 三倍扩大室

* 目标优化：`6`，目标效率：`24`
* 实际优化：`6`，实际效率：`24`

```
a:
    INBOX   
    COPYTO   0
    ADD      0
    ADD      0
    OUTBOX  
    JUMP     a
```

思路：将手中数字同地板相同数字执行 ADD ，相当于 i + i，执行 n 次等于 n * i，也就是相当于乘法。

## 09 零保护行动

* 目标优化：`5`，目标效率：`24`
* 实际优化：`5`，实际效率：`24`

```
a:
b:
    INBOX   
    JUMPZ    c
    JUMP     a
c:
    OUTBOX  
    JUMP     b
```

思路：是 0 则输出，否则 JUMP 到开头。最终效率 28。

```
a:
    INBOX   
    JUMPZ    b
    INBOX   
    JUMPZ    c
    INBOX   
    JUMPZ    d
    INBOX   
    JUMPZ    e
    INBOX   
    JUMPZ    f
    INBOX   
    JUMPZ    g
    INBOX   
    JUMPZ    h
b:
c:
d:
e:
f:
g:
h:
    OUTBOX  
    JUMP     a
```

思路：解开了上个解答当中的第 3 步 JUMP ，修改为输入，判断为 0 则马上到输出的序列。题目中有 7 个数字，因此为了确保遇到完全没有 0 也能跳转，所谓要重复 7 次。平均效率 23.

## 10 八倍扩大装置

* 目标优化：`9`，目标效率：`36`
* 实际优化：`9`，实际效率：`36`

```
a:
    INBOX   
    COPYTO   0
    ADD      0
    COPYTO   0
    ADD      0
    COPYTO   0
    ADD      0
    OUTBOX  
    JUMP     a
```

思路：同地板自增之后覆盖并循环的行为相当于不断执行 i = i + i 的操作，每次执行之后 i 都会翻倍，执行 3 次之后为 8 倍。

## 11 加运算走廊

* 目标优化：`10`，目标效率：`40`
* 实际优化：`10`，实际效率：`40`

```
a:
    INBOX   
    COPYTO   0
    INBOX   
    COPYTO   1
    SUB      0
    OUTBOX  
    COPYFROM 0
    SUB      1
    OUTBOX  
    JUMP     a
```

思路：新增 SUB 指令，由于需要后减前，前再减去后，所以两个值都要存放到地板。后减前之后重新从地板取得前值用于减去后值。

## 12 四十倍扩大器

* 目标优化：`14`，目标效率：`56`
* 实际优化：`14`，实际效率：`56`

```
a:
    INBOX   
    COPYTO   0
    ADD      0
    COPYTO   0
    ADD      0
    COPYTO   0
    ADD      0
    COPYTO   0
    ADD      0
    ADD      0
    ADD      0
    ADD      0
    OUTBOX  
    JUMP     a
```

思路：四十的倍数我们可以拆开来看，因此我们可以先自增覆盖三次得到 8 倍，然后自增但是不覆盖 5 次来得到 5 * 8 = 40 倍。

## 13 均衡之间

* 目标优化：`9`，目标效率：`27`
* 实际优化：`9`，实际效率：`27`

```
a:
b:
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPZ    c
    JUMP     b
c:
    COPYFROM 0
    OUTBOX  
    JUMP     a
```

思路：两个数字相减为 0 则相当，因此先按照正常流程假设不为 0 写下前面 4 步并启动 JUMP 循环，然后在第 4 步后面加入 是否为 0 判断，是的话则跳转到拷贝地板值并输出。效率为 28 步。

```
a:
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPZ    b
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPZ    c
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPZ    d
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPZ    e
b:
c:
d:
e:
    COPYFROM 0
    OUTBOX  
    JUMP     a
```

思路：解开第 6 步的 JUMP 跳转循环，重复 1-4，每次都判断成功都跳到最后的输出，最终效率 27。

## 14 最大值室

* 目标优化：`10`，目标效率：`34`
* 实际优化：`10`，实际效率：`34`

```
a:
    INBOX   
    COPYTO   0
    INBOX   
    SUB      0
    JUMPN    b
    ADD      0
    JUMP     c
b:
    COPYFROM 0
c:
    OUTBOX  
    JUMP     a
```

思路：用第二个数减去第一个数，得到负数则说明第二个数小于第一个数，负数的话则拷贝第一个存放的值，然后输出，如果是非负数的话则输出拿到的第二个值。需要注意的是，在获取第二个值之后并没有存放到地板，而是直接拿来用，然后当判断为非负数之后通弄过 ADD 来获得原来的值，这样做效率更高。

## 15 斗志注入（略）