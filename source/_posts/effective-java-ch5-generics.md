---
title: 泛型--《Effective Java》
date: 2017-02-09 13:30:14
categories: [后台开发, JAVA开发]
tags: [JAVA, Effective Java]
---

## 前言
本文为阅读《Effective Java》第五章阅读笔记。

## 不要在新代码中使用原生态类型
声明中具有一个或多个类型参数的类/接口 -- 泛型类/接口 

使用原生态类型会在运行时导致异常。原生态只是为了与引入泛型之前的遗留代码进行兼容和互用而提供的。

`Set<Object>`是个参数化类型，表示可以包含任何对象类型的一个集合。

`Set<?>`是一个通配符类型，表示只能包含某种未知对象类型的一个集合。

`Set`是个原生态类型，脱离了泛型系统，是不安全的。

| 术语 | 实例 |
| --- | --- |
| 参数化的类型 | List<String> |
| 实际类型参数 | String |
| 泛型 | List<E> |
| 形式类型参数 | E |
| 无限制通配符类型 | List<?> |
| 原生态类型 | List |
| 有限制类型参数 | < E extends Number> |
| 递归类型限制 | < T extends Comparable<T>> |
| 有限制通配符类型 | List<? extends Number> |
| 泛型方法 | static <E> List<E> asList(E[] a) |
| 类型令牌 | String class |

---

##  消除非受检警告
不要忽略非受检警告。每条警告都表示可能在运行时抛出ClassCastException异常，要尽最大努力消除这些警告。

如果无法消除非受检警告，同时可以证明引起警告的代码是类型安全的，就可以在尽可能小的范围中，用`@SuppressWarnings("unchecked")`注解禁止该警告，要用注释把禁止该警告的原因记录下来。

---

## 列表优先于数组
数组是具体化的，在运行时才知道并检查它们的元素类型约束。

泛型是通过erasure(使泛型可以与没有使用泛型的代码随意进行互用)来实现的。只在编译时强化它们的类型信息，并在运行时丢弃元素类型信息。

数组提供了运行时的类型安全，但是没有编译时的类型安全。反之泛型。

---

## 优先考虑泛型
使用泛型比使用需要在客户端代码中进行转换的类型更为安全，也更简单。

---

## 优先考虑泛型方法
静态工具方法尤其适合于泛型化。

泛型方法 无需明确指定类型参数的值。

泛型静态工厂方法

    //Generic static factory method
    public static <K, V> HashMap<K, V> newHashMap() {
        return new HashMap<K,V>();
    }

    //Parameterized type instance creation with static factory
    Map<String, List<String>> anagrams = newHashMap();

---

## 利用有限制通配符来提升API的灵活性
参数化类型是不可变的。

PECS表示producer-extends, consumer-super

如果参数化类型表示一个T生产者，就使用`<? extends T>`; 如果表示一个T消费者，就使用`<? super T>`。 

不要用通配符类型作为返回类型。

如果类型参数旨在方法声明中出现一次，就可以用通配符取代它。

    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    //private helper method for wildcard capture
    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

---

## 优先考虑类型安全的异构容器

    //Typesafe heterogeneous container pattern - client
    public static void main(String[] args){
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
        
        System.out.printf("%s %x %s%n, favoriteString,
            favoriteInteger, favoriteClass.getName());
        //Java cafebabe Favorites.
    }


    //Typesafe heterogeneous container pattern - implementation
    public class Favorites {
        private Map<Class<?>, Object> favorites = 
            new HashMap<Class<?>, Object>();

        public <T> void putFavorite(Class<T> type, T instance) {
            if(type == null)
                throw new NullPointerException("Type is null");
            favorites.put(type, instance);
        }

        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }

利用asSubclass在编译时读取类型未知的注解。

    //Use of asSubclass to safely cast to a bounded type token
    static Annotation getAnnotation(AnnotatedElement element,
                                    String annotationTypeName) {
        Class<?> annotationType = null; //Unbounded type token
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch(Excetion ex) {
            throw new IllegalArgumentExcetion(ex);
        }
        return element.getAnnotation(
            annotationType.asSubclass(Annotation.class);
        )
    }

集合API说明了泛型的一般用法，限制你每个容器只能有固定数目的类型参数，可以通过将类型参数放在键上而不是容器上来避开这一限制。

对于这种类型安全的异构容器，可以用Class对象作为键。以这种方式使用的Class对象称作类型令牌。