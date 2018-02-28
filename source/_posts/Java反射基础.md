---
title: Java反射基础
---


# 一、Class类的使用
all is object：类是对象，类是java.lang.Class类的实例对象

1. 在得知该类的类名时直接在后面.class就可以得到一个class类型的对象 Class c = Foo.class

2. 在已有该类的实例对象的时候，通过这个实例对象调用.getClass()方法，也可以得到一个class类型的对象

3. 在得知该类的路径的时候，调用Class的静态方法ClassForName()方法，就可以得到一个Class类型的对象

用该类的类类型可以创建该类实例对象，用newInstance()方法再强转为该类
Foo foo = (Foo)c.newInstance();


``` java
public class ClassDemo1 {
    public static void main(String[] args) {
        //Foo的实例对象如何表示
        Foo foo1 = new Foo();//foo1就表示出来了.
        //Foo这个类 也是一个实例对象，Class类的实例对象,如何表示呢
        //任何一个类都是Class的实例对象，这个实例对象有三种表示方式

        //第一种表示方式--->实际在告诉我们任何一个类都有一个隐含的静态成员变量class
        Class c1 = Foo.class;

        //第二中表达方式  已经知道该类的对象通过getClass方法
        Class c2 = foo1.getClass();

        /*官网 c1 ,c2 表示了Foo类的类类型(class type)
         * 万事万物皆对象，
         * 类也是对象，是Class类的实例对象
         * 这个对象我们称为该类的类类型
         */
        //不管c1  or c2都代表了Foo类的类类型，一个类只可能是Class类的一个实例对象
        System.out.println(c1 == c2);

        //第三种表达方式
        Class c3 = null;
        try {
            c3 = Class.forName("com.imooc.reflect.Foo");
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println(c2==c3);

        //我们完全可以通过类的类类型创建该类的对象实例---->通过c1 or c2 or c3创建Foo的实例对象
        try {
            Foo foo = (Foo)c1.newInstance();//需要有无参数的构造方法
            foo.print();
        } catch (InstantiationException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
class Foo{
    void print(){
        System.out.println("foo");
    }
}
```

# 二、动态加载类
- 静态加载类：编译时加载的类（用new关键字创建的类）

- 动态加载类：运行时加载的类（用类类型创建的类，需要有接口，活抽象父类）
如 Class c = Class.forName("类全名");就是动态加载。
接口的引用 = c.newInstance();//创建实例对象，需要有无参构造方法。

当其它的几个类我们不明确到底会使用哪一个的时候，可以定义一个接口让这些类都实现它，这样就可以调用对应的方法了。

# 三、获取方法、成员变量、构造函数信息
基本的数据类型 void关键字都存在类类型

``` java
public class ClassDemo2 {
    public static void main(String[] args) {
        Class c1 = int.class;//int 的类类型
        Class c2 = String.class;//String类的类类型   String类字节码（自己发明的)
        Class c3 = double.class;
        Class c4 = Double.class;
        Class c5 = void.class;
        //类类型的名称
        System.out.println(c1.getName());
        System.out.println(c2.getName());
        //不带包名的类类型名称
        System.out.println(c2.getSimpleName());//不包含包名的类的名称
        System.out.println(c5.getName());
    }
}
```
## Class类的基本API操作
Class类的常用方法：

方法名 | 说明
-----|-----
 getName()  | 基本数据类型得到的是类名，引用型得到的是引用全称（java.lang.String）
getMethods()/getFields() | 得到方法/字段的集合，包括父类继承而来的方法/字段,只限 公共的方法/字段
getDeclaredMethods()/getDeclaredFields() | 得到当前类的方法/字段，不包括父类的，不限公共的还是私有的
Method的getReturnType() | 得到方法的返回值的类类型
Method的getParmeterTypes() | 得到的是方法参数的类类型


### java.lang.reflect.Constructor——构造方法的类
- 构造函数也是对象
- java.lang. Constructor中封装了构造函数的信息
- getConstructors获取所有的public的构造函数
- getDeclaredConstructors得到所有的构造函数
### java.lang.reflect.Field——成员变量的类
- 成员变量也是对象
- Field类封装了关于成员变量的操作
- getFields()方法获取的是所有的public的成员变量的信息
- getDeclaredFields获取的是
### java.lang.reflect.Method——成员方法的类
- Method类，方法对象
- 一个成员方法就是一个Method对象
- getMethods()方法获取的是所有的public的函数，包括父类继承而来的
- getDeclaredMethods()获取的是所有该类自己声明的方法，不问访问权限

## 实例代码

ClassUtil

``` java
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.lang.reflect.Field;

public class ClassUtil {
    /**
     * 打印类的信息，包括类的成员函数、成员变量(只获取成员函数)
     * @param obj 该对象所属类的信息
     */
    public static void printClassMethodMessage(Object obj){
        //要获取类的信息  首先要获取类的类类型
        Class c = obj.getClass();//传递的是哪个子类的对象  c就是该子类的类类型
        //获取类的名称
        System.out.println("类的名称是:"+c.getName());
        /*
         * Method类，方法对象
         * 一个成员方法就是一个Method对象
         * getMethods()方法获取的是所有的public的函数，包括父类继承而来的
         * getDeclaredMethods()获取的是所有该类自己声明的方法，不问访问权限
         */
        Method[] ms = c.getMethods();//c.getDeclaredMethods()
        for(int i = 0; i < ms.length;i++){
            //得到方法的返回值类型的类类型
            Class returnType = ms[i].getReturnType();
            System.out.print(returnType.getName()+" ");
            //得到方法的名称
            System.out.print(ms[i].getName()+"(");
            //获取参数类型--->得到的是参数列表的类型的类类型
            Class[] paramTypes = ms[i].getParameterTypes();
            for (Class class1 : paramTypes) {
                System.out.print(class1.getName()+",");
            }
            System.out.println(")");
        }
    }
    /**
     * 获取成员变量的信息
     * @param obj
     */
    public static void printFieldMessage(Object obj) {
        Class c = obj.getClass();
        /*
         * 成员变量也是对象
         * java.lang.reflect.Field
         * Field类封装了关于成员变量的操作
         * getFields()方法获取的是所有的public的成员变量的信息
         * getDeclaredFields获取的是该类自己声明的成员变量的信息
         */
        //Field[] fs = c.getFields();
        Field[] fs = c.getDeclaredFields();
        for (Field field : fs) {
            //得到成员变量的类型的类类型
            Class fieldType = field.getType();
            String typeName = fieldType.getName();
            //得到成员变量的名称
            String fieldName = field.getName();
            System.out.println(typeName+" "+fieldName);
        }
    }
    /**
     * 打印对象的构造函数的信息
     * @param obj
     */
    public static void printConMessage(Object obj){
        Class c = obj.getClass();
        /*
         * 构造函数也是对象
         * java.lang. Constructor中封装了构造函数的信息
         * getConstructors获取所有的public的构造函数
         * getDeclaredConstructors得到所有的构造函数
         */
        //Constructor[] cs = c.getConstructors();
        Constructor[] cs = c.getDeclaredConstructors();
        for (Constructor constructor : cs) {
            System.out.print(constructor.getName()+"(");
            //获取构造函数的参数列表--->得到的是参数列表的类类型
            Class[] paramTypes = constructor.getParameterTypes();
            for (Class class1 : paramTypes) {
                System.out.print(class1.getName()+",");
            }
            System.out.println(")");
        }
    }
}
```
ClassDemo3

``` java
public class ClassDemo3 {
    public static void main(String[] args) {
        String s = "hello";
        ClassUtil.printClassMethodMessage(s);

        Integer n1 = 1;
        ClassUtil.printClassMethodMessage(n1);
    }
}
```

# 反射方法的基本操作
## 方法的反射：
1. 方法名称加参数列表可以唯一确定一个方法
2. 通过方法对象来实现方法的功能，即把实例对象当成参数传给方法对象

``` java
import java.lang.reflect.Method;

public class MethodDemo1 {
    public static void main(String[] args) {
       //要获取print(int ,int )方法  1.要获取一个方法就是获取类的信息，获取类的信息首先要获取类的类类型
        A a1 = new A();
        Class c = a1.getClass();
        /*
         * 2.获取方法 名称和参数列表来决定  
         * getMethod获取的是public的方法
         * getDelcaredMethod自己声明的方法
         */
        try {
            //Method m =  c.getMethod("print", new Class[]{int.class,int.class});
            Method m = c.getMethod("print", int.class,int.class);

            //方法的反射操作  
            //a1.print(10, 20);方法的反射操作是用m对象来进行方法调用 和a1.print调用的效果完全相同
            //方法如果没有返回值返回null,有返回值返回具体的返回值
            //Object o = m.invoke(a1,new Object[]{10,20});
            Object o = m.invoke(a1, 10,20);
            System.out.println("==================");
            //获取方法print(String,String)
             Method m1 = c.getMethod("print",String.class,String.class);
             //用方法进行反射操作
             //a1.print("hello", "WORLD");
             o = m1.invoke(a1, "hello","WORLD");
             System.out.println("===================");
           //  Method m2 = c.getMethod("print", new Class[]{});
                Method m2 = c.getMethod("print");
               // m2.invoke(a1, new Object[]{});
                m2.invoke(a1);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } 

    }
}
class A{
    public void print(){
        System.out.println("helloworld");
    }
    public void print(int a,int b){
        System.out.println(a+b);
    }
    public void print(String a,String b){
        System.out.println(a.toUpperCase()+","+b.toLowerCase());
    }
}
```
输出结果：

    30
    ==================
    HELLO,world
    ===================
    helloworld

# 通过反射了解集合泛型的本质
Java集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了

``` java
public class MethodDemo4 {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();

        ArrayList<String> list1 = new ArrayList<String>();
        list1.add("hello");
        //list1.add(20);错误的
        Class c1 = list.getClass();
        Class c2 = list1.getClass();
        System.out.println(c1 == c2);
        //反射的操作都是编译之后的操作
        /*
         * c1==c2结果返回true说明编译之后集合的泛型是去泛型化的
         * Java中集合的泛型，是防止错误输入的，只在编译阶段有效，
         * 绕过编译就无效了
         * 验证：我们可以通过方法的反射来操作，绕过编译
         */
        try {
            Method m = c2.getMethod("add", Object.class);
            m.invoke(list1, 20);//绕过编译操作就绕过了泛型
            System.out.println(list1.size());
            System.out.println(list1);
            /*for (String string : list1) {
                	System.out.println(string);
            }*///现在不能这样遍历
        } catch (Exception e) {
          e.printStackTrace();
        }
    }
}
```
