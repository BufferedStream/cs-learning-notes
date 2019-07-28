## Java中field和variable区别及相关术语解释

先说一下field和variable之间的区别：

```
Class variables and instance variables are fields while local varables and parameter variables are not.All fields are variables.
```

成员变量（field）是指类的数据成员，而方法内部的局部变量（local variable）、参数变量（parameter variable）不能称作 field。field 属于 variable，也就是说 **variable 的范围更大**。



#### 术语解释：

1. **域或字段、实例变量、成员变量**（field, instance variable, member variable, non-static field）

   > field: A data member of a class. Unless specified otherwise, a field is not static.

   - 非 static 修饰的变量。

   - 虽然有如上定义，但是一般在使用时，成员变量（field）包括 instance variable 和 class variable。为了区分，个人认为，用实例变量/非静态变量（instance variable / non-static field）描述上面的定义更佳。

   - 成员变量与特定的对象相关联，只能通过对象（new 出）访问。

   - 声明在类中，但不在方法或构造方法中。

   - 如果有多个对象的实例，则每一个实例都会持有一份成员变量，实例之间不共享成员变量的数据。

   - 作用域比静态变量小，可以在类中或者非静态方法中使用以及通过生成实例对象使用。（访问限制则不可用）

   - JVM 在初始化类的时候会给成员变量赋初始值。

     

2. **类字段、静态字段、静态变量**（class variable, static field, staic variable）

- 使用 static 修饰的字段，一般叫做静态变量。
- 声明在类中，但不在方法或构造方法中。
- 多个实例对象共享一份静态变量
- JVM在准备类的时候会给静态变量赋初始值。
- 作用域最大，类中都可以访问，或通过 类名.变量名 的方式调用（访问限制则不可用）。



3. **局部变量**（local variable）
   定义在一个区块内（通常会用大括号包裹），区块外部无法使用的变量。

- 定义在一个区块内（通常会用大括号包裹），没有访问修饰符，区块外部无法使用的变量。
- 没有默认值，所以必须赋初始值
- 生命周期即为方法的生命周期



​	4. **参数**（input parameter, parameter (variable), argument）

​		这个就不多说了，要注意的是 argument 和 parameter 的区别（下文）。
​	另外，Oracle 官方文档中将参数分为了构造参数、方法参数和异常参数三部分。

```
Strictly speaking, a parameter is a variable within the definition of a method. An argument would be the data or actual value which is passed to the method. An example of parameter usage: int numberAdder(first, second) An example of argument usage: numberAdder(4,2)
```



5. **不可变量、常量**（final variable, constant）

​		即为使用 final 关键词修饰的变量。不可变量属于成员变量。

​	

 6. **成员**（member）

    ```
    A field or method of a class. Unless specified otherwise, a member is not static.
    ```

    指的是类中非静态的成员变量或方法。（用法同field）



7. **属性**（property）

   ```
   Characteristics of an object that users can set, such as the color of a window.
   ```

   可以被用户设置或获取的对象特征即为属性。
   POJO 或 JavaBean 中的成员变量也称作属性（具有set、getter方法）。



   **最后，总结一下国内目前的惯用法**（英文取其一，序号对应上文）：
	
1. field -> 成员变量， instance variable / non-static field -> 实例变量/非静态变量
2. class variable -> 静态变量
3. local variable -> 本地变量
4. input parameter -> 参数
5. final variable -> 常量
6. member -> 成员（用法同field）
7. property -> 属性