---
layout:     post
title:      "Java源码分析 Map"
subtitle:   " \"Map源码分析\""
date:       2018-04-21 17:00:00
author:     "Mike"
header-img: "img/2.png"
catalog: true
tags:
    - Java
---

> “Come on. ”

## 源码

    package java.lang;

	// final 修饰的 可序列化的类
	public final class Boolean implements java.io.Serializable,Comparable<Boolean>
	{
	    // 享元模式    
	    public static final Boolean TRUE = new Boolean(true);
	    public static final Boolean FALSE = new Boolean(false);
	
	    @SuppressWarnings("unchecked")
	    public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");
	    
	    private final boolean value;
	
	    // 序列化操作的时候系统会把当前类的serialVersionUID写入到序列化文件中，
	    // 当反序列化时系统会去检测文件中的serialVersionUID，判断它是否与当前类的serialVersionUID一致，
	    // 如果一致就说明序列化类的版本与当前类版本是一样的，可以反序列化成功，否则失败，抛出InvalidCastException异常。
	    private static final long serialVersionUID = -3665804199014368530L;
	
	    // 传入boolean的构造器
	    public Boolean(boolean value) {
	        this.value = value;
	    }
	
	    // 传入String的构造器
	    public Boolean(String s) {
	        this(parseBoolean(s));
	    }
	
	    // equalsIgnoreCase忽略大小的equal
	    public static boolean parseBoolean(String s) {
	        return ((s != null) && s.equalsIgnoreCase("true"));
	    }
	
	    public boolean booleanValue() {
	        return value;
	    }
	
	    // 将boolean转化为Boolean
	    public static Boolean valueOf(boolean b) {
	        return (b ? TRUE : FALSE);
	    }
	
	    // 将String转化成boolean
	    public static Boolean valueOf(String s) {
	        return parseBoolean(s) ? TRUE : FALSE;
	    }
	
	    // 转成String
	    public static String toString(boolean b) {
	        return b ? "true" : "false";
	    }
	
	    // 转成String
	    public String toString() {
	        return value ? "true" : "false";
	    }
	
	    // true的是1231，false是1237
	    @Override
	    public int hashCode() {
	        return Boolean.hashCode(value);
	    }
	
	    // 两个大素数就好了，减少hash碰撞
	    public static int hashCode(boolean value) {
	        return value ? 1231 : 1237;
	    }
	
	    // equals方法
	    public boolean equals(Object obj) {
	        if (obj instanceof Boolean) {
	            return value == ((Boolean)obj).booleanValue();
	        }
	        return false;
	    }
	
	
	    public static boolean getBoolean(String name) {
	        boolean result = false;
	        try {
	            result = parseBoolean(System.getProperty(name));
	        } catch (IllegalArgumentException | NullPointerException e) {
	        }
	        return result;
	    }
	
	    // compareTo 方法
	    public int compareTo(Boolean b) {
	        return compare(this.value, b.value);
	    }
	
	    // return 0， x = true， y = true 或 x = false, y = fasle;
	    // return 1,  x = true,  y = fasle
	    // return -1, x = false, y = true
	    public static int compare(boolean x, boolean y) {
	        return (x == y) ? 0 : (x ? 1 : -1);
	    }
	
	    // 且操作
	    public static boolean logicalAnd(boolean a, boolean b) {
	        return a && b;
	    }
	
	    // 或操作
	    public static boolean logicalOr(boolean a, boolean b) {
	        return a || b;
	    }
	
	    // 异或操作
	    public static boolean logicalXor(boolean a, boolean b) {
	        return a ^ b;
	    }
	}






如果你恰好逛到了这里，希望你也能喜欢这个博客主题。

如果你也喜欢我的项目，欢迎star和follow我，感激不尽。

—— Mike 后记于 2017.9.14
