---
layout:     post
title:      "浅谈 Java 反射"
subtitle:   "架构师的必经路"
date:       2016-11-10 02:08:00
author:     "chenzhijun"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Java
---

Java 反射
============================

### 1.反射的作用？
在程序运行期间通过反射可以获取加载类的字段，方法和构造方法的信息。说人话就是在运行期间，jvm 加载了一个类，你可以通过反射获取到这个类的所有信息。

---

### 2.知识点

* java.lang.reflect.* 包
* Class 类--java.lang.Class

Class 类的实例表示正在运行的 Java 应用程序中的类和接口。我们可以通过对象的类型来获取到 Class 对象。这个有点绕口。
比如： Object A ； A 是 Object 类型的一个对象，而 Object 又是 Class 类型的一个对象。

下面我们通过实际例子来获取 Class。

首先我们先定义一个类：Book，包括属性 name：书名；type：类型。

```
package lang.reflect.practice;

/**
 * title:
 * description:
 * Package: lang.reflect.practice
 * ClassName: Book
 * Author zhijun.chen
 * CreateTime: 2016/11/9 16:56
 * version 1.0.0
 */
public class Book {

    private int id;
    private String name;
    private String type;

    public Book(){

    }

    public Book(int id, String name, String type) {
        this.id = id;
        this.name = name;
        this.type = type;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        System.out.println(name);
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", type='" + type + '\'' +
                '}';
    }

    public void test(String name, int i){
        System.out.println("name: "+name+" ,i:"+i);
    }
}

```
接下来我们获取这个类的 Class 对象：

```
    public static Class getClassWays() {
        try {
            Class book1 = Class.forName("lang.reflect.practice.Book");// 获取 class
            System.out.println(book1);

            Book book = new Book();
            Object object = book;
            Class book2 = object.getClass();//也可以 booke.getClass();
            System.out.println(book2);

            Class book3 = Book.class;
            System.out.println(book3);


            ClassLoader loader = Thread.currentThread().getContextClassLoader();
            Class clazz = loader.loadClass("lang.reflect.practice.Book");

            return book1;

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }
```
上面是我实践过的获取 class 对象，如果还有其它的方式，可以在底下留言一起讨论。

获取到 Class 对象之后，最重要的一步就已经完成了。之后我们可以通过 java.lang.reflect.*包来完成我们需要的操作。
通过反射获取到我们需要的属性，方法，父类和接口的信息：

```
public class Main {
    public static void main(String[] args) {
        Class c = getClassWay();
//        showField(c);
//
        Book book = new Book(1, "奋斗吧，骚年", "鸡汤");
//
//        show(book);

//        methodInvoke(book);

    }

    private static void methodInvoke(Book book) {
        Object obj = book;
        Class cl = obj.getClass();

        try {
            Method m = cl.getMethod("setName",String.class);
            m.invoke(obj,"测试测试，开发开发");


            Method m2 = cl.getMethod("getName");
            m2.invoke(obj,new Object[0]);

            Method m3 = cl.getMethod("test",new Class[]{String.class,int.class});
            m3.invoke(obj,new Object[]{"哈哈哈",8});

            Book a = (Book) cl.newInstance();
            System.out.println(a);
            System.out.println(obj);

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }

    }

    private static void show(Book book) {
        Object obj = book;
        Class c = obj.getClass();
        Method[] methods = c.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("方法名称："+ method.getName());
            System.out.println("返回值类型："+method.getReturnType());
            System.out.println("方法修饰符号："+method.getModifiers());
            System.out.println("方法修饰符："+ Modifier.toString(method.getModifiers()));
//            System.out.println(" 方法注解信息：");
//            Annotation[] annotations = method.getDeclaredAnnotations();
//            for(Annotation annotation : annotations){
//                System.out.println(annotation.toString());
//            }

            System.out.println("方法参数列表");
            Class[] classes = method.getParameterTypes();
            for(Class cls : classes){
                System.out.print(" "+cls.getName());
            }
            System.out.println();
        }

    }



    private static void showField(Class c) {
        Field[] fields = c.getDeclaredFields();
        System.out.println(fields.length);
        try {
            Object obj = c.newInstance();
            for (Field ff : fields) {
                ff.setAccessible(true);//如果有私有属性，没有设置这个值得话，会报错IllegalAccessException
                System.out.println("属性名：" + ff.getName() + ", 属性值：" + ff.get(obj));
            }
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    private static Class getClassWay() {
        try {
            Class book1 = Class.forName("lang.reflect.practice.Book");
            System.out.println(book1);

            Book book = new Book();
            Object object = book;
            Class book2 = object.getClass();
            System.out.println(book2);

            Class book3 = Book.class;
            System.out.println(book3);


            Book book4 = (Book) book3.newInstance();
            System.out.println(book4);

            ClassLoader loader = Thread.currentThread().getContextClassLoader();
            Class clazz = loader.loadClass("lang.reflect.practice.Book");
            Constructor cons = clazz.getDeclaredConstructor(new Class[]{int.class,String.class,String.class});
            Book loaderBook = (Book) cons.newInstance(new Object[]{1,"loader 加载","类型"});
            System.out.println(loaderBook);


            return book1;

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }
}

```
这里有个注意的地方，获取私有属性值，需要设置setAccessible(true);不然会报IllegalAccessException。
