---
title: JAVA创建和销毁对象--《Effective Java》
date: 2017-02-06 10:17:06
categories: [后台开发, JAVA开发]
tags: [JAVA, Effective Java]
---
## 前言
本文为阅读《Effective Java》第二章的笔记。

## 用静态工厂方法代替构造器
### 静态工厂方法优势
* 有名称-调用更清晰
* 每次调用时不会创建一个新对象
* 可以返回原返回类型的任何子类型的对象
* 创建参数化类型实例的时候，使代码更简洁
### 静态工厂方法缺点
* 类如果不含共有的或受保护的构造器，就不能被子类化
* 与其他静态方法实际上没有任何区别

---

## 遇到多个构造器参数时要考虑用构建器
### ** 重叠构造器模式** 
但是，在有很多参数时，客户端代码难以编写且难以阅读。
### ** JavaBeans模式** 。
调用一个无参构造器来创建对象，调用`setter`方法来设置参数。
缺点：构造过程被分到了几个调用，导致可能处于不一致状态。
### Builder模式
让客户端利用所有必要参数调用构造器/静态工厂，得到builder对象，再调用类似于setter方法，最后调用无参的build方法来生成不可变对象。

    public class NutritionFacts {
        private final int servingSize;
        private final int servings;
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;

        public static class Builder {
            //Required parameters
            private final int servingSize;
            private final int servings;

            //Optional parameters - initialized to default values
            private int calories     = 0;
            private int fat          = 0;
            private int carbohydrate = 0;
            private int sodium       = 0;


            public Builder(int servingSize, int servings) {
                this.servingSize = servingSize;
                this.servings = servings;
            }

            public Builder calories(int val){
                calories = val; return this;
            }
            public Builder fat(int val){
                fat = val; return this;
            }
            public Builder carbohydrate(int val){
                carbohydrate = val; return this;
            }
            public Builder sodium(int val){
                sodium = val; return this;
            }

            public NutritionFacts build(){
                return new NutritionFacts(this);
            }
        }

        private NutritionFacts(Builder builder) {
            servings     = builder.servings;
            servingSize  = builder.servingSize;
            calories     = builder.calories;
            fat          = builder.fat;
            sodium       = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    }

    //Client
    NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
        calories(100).sodium(35).carbohydrate(27).build();

builder可以检验约束条件，并且可有多个可变参数。
但是Builder模式更为冗长，只有在很多参数（>=4）时才使用。

---

## 用私有构造器或枚举类型强化Singleton属性
单元素的枚举类型已经成为实现Singleton的最佳方法。

    public enum Elvis {
        INSTANCE;

        public void leaveTheBuilding() {...}
    }
这种方式提供了序列化机制，并可防止多次实例化。

---

## 通过私有构造器强化不可实例化的能力
为了不被实例化，可使用私有构造器来实现。

    //Noninstantiable utility class
    public class UtilityClass {
        private UtilityClass() {
            throw new AssertionError();
        }

        ... // Remainder omitted
    }

这种方式使得一个类不能被子类化。
所有构造器必须显式或隐式调用超类构造器，在这种情形下，子类就没有可访问的超类构造器调用。

---

## 避免创建不必要的对象
重用不可变对象，重用已知不会被修改的可变对象。

对于同时提供了静态工厂方法和构造器的不可变类，通常可以使用静态工厂方法，避免创建不必要的对象。构造器每次被调用时都会创建一个新对象。

自动装箱（autoboxing）-- 创建多余对象的新方法。要优先使用基本类型而不是装箱基本类型。


---

## 消除过期的对象调用
只要类是自己管理内存，就应该警惕内存泄漏问题。一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空。
常见的内存泄漏还有： 缓存、监听器和其他回调。
确保回调立即被当作垃圾回收的最佳方法是只保存它们的弱引用。

---

## 避免使用终结方法
终结方法的缺点在于不能保证会被及时地执行。
不应该依赖终结方法来更新重要的持久状态。
显式的终止方法通常与try-finally结构结合使用，以确保及时终止。
#### 终止方法的用途
* 当对象所有者忘记调用显式终止方法时，终结方法可充当“安全网” -- 终结方法发现资源还未被终止，应在日志中记录一条警告。
* 终止非关键的本地资源。

使用了终结方法就要记住调用`super.finalize`。
