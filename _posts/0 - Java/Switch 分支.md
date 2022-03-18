### switch表达式

使用`switch`时，如果遗漏了`break`，就会造成严重的逻辑错误，而且不易在源代码中发现错误。从Java 12开始，`switch`语句升级为更简洁的表达式语法，使用类似模式匹配（Pattern Matching）的方法，保证只有一种路径会被执行，并且不需要`break`语句：

```Java
switch (fruit) {
  case "apple" -> System.out.println("Selected apple");
  case "pear" -> System.out.println("Selected pear");
  case "mango" -> {
    System.out.println("Selected mango");
    System.out.println("Good choice!");
  }
  default -> System.out.println("No fruit selected");
}
```

注意新语法使用`->`，如果有多条语句，需要用`{}`括起来。不要写`break`语句

使用新的`switch`语法，不但不需要`break`，还可以直接返回值。把上面的代码改写如下：

```Java
String fruit = "apple";
int opt = switch (fruit) {
  case "apple" -> 1;
  case "pear", "mango" -> 2;
  default -> 0;
}; // 注意赋值语句要以;结束
System.out.println("opt = " + opt);
```

### yield

大多数时候，在`switch`表达式内部，我们会返回简单的值。

但是，如果需要复杂的语句，我们也可以写很多语句，放到`{...}`里，然后，用`yield`返回一个值作为`switch`语句的返回值：

```Java
String fruit = "orange";
int opt = switch (fruit) {
  case "apple" -> 1;
  case "pear", "mango" -> 2;
  default -> {
    int code = fruit.hashCode();
    yield code; // switch语句返回值
  }
};
System.out.println("opt = " + opt);
```

