---
title: 枚举与注解--《Effective Java》
date: 2017-02-10 20:26:11
categories: [后台开发, JAVA开发]
tags: [JAVA, Effective Java]
---
## 前言
本文为阅读《Effective Java》第六章笔记。

---

## 用enum代替int常量
枚举类型（enum type）是值由一组固定的常量组成合法值的类型。

    public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
    public enum Orange { NAVEL, TEMPLE, BLOOD }

Java的枚举本质上是int值。通过公有的静态final域为每个枚举常量导出实例的类。

枚举类型是实例操控的，是单例的泛型化，本质上是单元素的枚举。

枚举类型还允许添加任意的方法和域，并实现任意的接口。

    //Enum type with data and behavior
    public enum Planet {
        MERCURY(3.302e+23, 2.439e6),
        VENUS(4.869E+24, 6.052E6),
        EARTH(5.975E+24, 6.378E6),
        MARS(6.419E+23, 3.393E6);
        ...

        private final double mass;
        private final double radius;
        private final double surfaceGravity;

        private static final double G = 6.67300E-11;

        //Constructor
        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
            surfaceGravity = G * mass / (radius * radius);
        }

        public double mass() { return mass; }
        public double radius() { return radius; }
        public double surfaceGravity() { return surfaceGravity; }

        public double surfaceWeighty(double mass) {
            return mass * surfaceGravity;
        }

    }

为了将数据与枚举常量关联起来，得声明实例域，并编写一个带有数据并将数据保存在域中的构造器。

    public class WeightTable {
        public static void main(String[] args) {
            double earthWeight = Double.parseDouble(args[0]);
            double mass = earthWeight / Planet.EARTH.surfaceGravity();
            for(Planet p : Planet.values()){
                System.out.printf("Weight on %s is %f%n",
                    p, p.surfaceWeighty(mass));
            }
        }
    }

    //Result
    //Weight on MERCURY is 66.133672
    //Weight on VENUS is 158.383926
    //Weight on EARTH is 175.000000
    //Weight on MARS is 66.430699

策略枚举来实现工资枚举。

    enum PayrollDay {
        MONDAY(PayType.WEEKDAY), TUEDAY(PayType.WEEKDAY),
        WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY),
        FRIDAY(PayType.WEEKDAY),
        SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

        private final PayType payType;
        PayrollDay(PayType payType) {
            this.payType = payType;
        }

        double pay(double hoursWorked, double payRate) {
            return payType.pay(hoursWorked, payRate);
        }

        //The strategy enum type
        private enum PayType {
            WEEKDAY {
                double overtimePay(double hours, double payRate) {
                    return hours <= HOURS_PER_SHIFT ? 0 :
                        (hours - HOURS_PER_SHIFT) * payRate / 2; 
                }
            },
            WEEKEND {
                double overtimePay(double hours, double payRate) {
                    return hours * payRate / 2;
                }
            };
            
            private static final int HOURS_PER_SHIFT = 8;

            abstract double overtimePay(double hrs, double payRate);

            double pay(double hoursWorked, double payRate) {
                double basePay = hoursWorked * payRate;
                return basePay + overtimePay(hoursWorked, payRate);
            }
        }
    }

枚举要更加易读，也更加安全，功能更为强大。许多枚举都不需要显式的构造器或成员，但许多其他枚举则受益于“每个常量与属性的关联”以及“提供行为受这个属性影响的方法”。如果多个枚举常量同时共享相同的行为，则考虑策略枚举。

---

## 用实例域代替序数
永远不要根据枚举的序数导出与它关联的值，而是要将它保存在一个实例域中。

    public enum Ensemble {
        SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
        SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
        NONET(9), DECTET(10), TRIPLE_QUARTET(12);

        private final int numberOfMusicians;
        Ensemble(int size) { this.numberOfMusicians = size; }
        public int numberOfMusicians() { return numberOfMusicians; }

    }

---

## 用EnumSet代替位域
待续……