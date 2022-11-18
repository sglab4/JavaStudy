参考 https://www.cnblogs.com/max-home/p/12270183.html

```java
public class Test {
    public static void main(String[] args) {
        Student student = new Student("abcd");
        student.sayName();
    }
}

class Student {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public void sayName() {
        System.out.println(name);
    }
}
```

main 方法的执行过程如下：

1. 首先堆 Test 类进行加载，执行 Test 类的加载过程
2. 执行 `Student student = new Student("abcd");` 发现没有加载 Student 类，对 Student 进行加载
3. 执行 `new Student("abcd");` ，包括分配内存区域，初始化等
4. 执行 sayName 时，先根据 student 的引用找到 student 对象，然后根据 student 对象持有的引用定位到方法区中 student 类的类型信息的方法表，获得 sayName() 的字节码地址。再执行 sayName 方法

主要考察了类的加载过程和 new 分配内存的过程

