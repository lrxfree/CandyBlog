---
title: Lambda表达式和委托
date: 2020-07-29 14:35:20
tags: C#
cover: ../image/c#.jpg
category: 编程语言 
---

### Lambda表达式和委托

其实，标题应该是委托和lambda表达式才合理，但是我写这个主要是为了综合记录一下C#里面Lambda表达式，我一直都在写，但是呢，lambda表达式无法和委托分开，所以就干脆放到一些吧。

> > #### 委托
>
> > 在C与C++中，能够利用“函数指针”将对方法的引用作为实参传给另一个方法，C#中的委托能够实现相同的功能，简单的说就是将方法当作普通参数一样传递；

以排序方法举例，我们实现了一个方法，这个方法能够实现升序排序，现在，我们需要再实现一个代码结构相同的，采用降序排序的方法，最简单直接的方法就是把原来那个方法复制，修改方法名和比较符，但是，这种方法，怎么能大量使用呢，低效、不优雅；	

为了减少重复代码和增强灵活性，可以将比较大小部分的代码提取为比较大小的方法，再将这个方法作为参数传递给主要的比较方法；为了能将方法传递，必须要有一个能够表示方法的数据类型，这个数据类型就是委托，

具体如代码示例，这里为了简洁，省略了具体的排序代码，只展示了代码调用的过程：

```c#
namespace Lambda
{
    //委托类型的声明；
public delegate bool ComparisonHandle(int first, int second);
    
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello word!");

            Sort(new int[] { 1, 23, 1, 2, 0 }, SortFounction);
            Console.ReadKey();
        }

        public static bool SortFounction(int arg1, int arg2)
        {
            return arg1 > arg2;
        }

        static void Sort(int[] item, ComparisonHandle comparisonMethon)
        {
            Console.WriteLine("这是使用了委托的方法内部");
            Console.WriteLine(comparisonMethon(item[0], item[2]));
        }
    }
}
```

> > 注意，委托也能嵌套在类中，如代码所示：

```c#
 class Delegatesample
 {
    public delegate bool ComparisonHandle(int first, int second);
 }
```

在这个例子里，声明的委托数据类型是`Delegatesample.ComparisonHandle`，因为它被定义成`Delegatesample`中的嵌套类型。

也可以这样处理委托：

```c#
ComparisonHandle handle = (arg1, arg2) => { return arg1 < arg2; };
```

在需要`ComparisonHandle`委托的地方直接把`handle`放过去就行了；

`备注:`记录一下 `CompareTo()`方法，int temp= arg1.CompareTo(arg2)；它会根据比较的结果返回一个数值，arg1>arg2 返回 1；arg1<arg2，返回 -1；arg1=arg2，返回 0；

> > #### Lambda表达式
>
> > 为了能使用委托类型，有时候我们需要为一个简单的方法主题写一个冗长的方法声明，比如上面的`SortFounction`方法，这个时候，就可以采用lambda表达式了，lambda表达式分为两种类型：
>
> > 语句Lambda

```c#
Sort(new int[] { 10,3,1,2,5},(arg1,arg2) => { return arg1 > arg2; });
```

通常，只要编译器能从Lambda表达式所转换成的委托推断出类型，就不需要在表达式里面显式声明参数的类型。

只有一个参数时，还能再省略：

```c#
Function(arg1 => { Console.WriteLine(arg1); });
```

> > 表达式Lambda
> >
> > 表达式Lambda更加简洁，

```c#
Sort(new int[] { 10,3,1,2,5},(arg1,arg2)=>  arg1 < arg2);
```

匿名方法没有必要掌握，要避免在新代码中使用匿名方法语法，应该优先使用更简洁Lambda表达式语法。