---
title: Android开发规范
grammar_cjkRuby: true
tags: [Android,开发规范]
categories: [Android]
date: 2017-01-12
---


### 代码规范



#### 命名规范

**1、类名、接口命名规则**

类和接口的名称应该是一个名词，与设计文档保持一致；采用大小写混和的方式，每个单词的首字母大写；
Example:
``` java
    class UserTest;//各单词首字母大写
    interface IUserTest;//接口以大写I开头
    abstract class AbsUserTest; //抽象类以Abs开头
```
  类的大小尽量不要超过500行，如果过大，应该考虑拆分成多个类。
  
**2、方法函数命名规则**

方法名应该是动词/动名词，采用大小写混和方式，第一个单词首字母小写，其后单词的首字线大写。回调方法以on开头，初始化方法以init开头，方法内尽量不要使用用匿名内部类。
Example:
```java
    run()
    runFase()
    onClick
    initView()
```
  一个方法尽量不要超过 50 行，如果方法太长，说明当前方法业务逻辑已经非常复杂，那么就需要进行方法拆分，保证每个方法只作一件事。
  
**3、变量命名规则**

变量命名采用大小写混和的方式，第一个单词首字母小写，其后单词的首字母大写。变量名应简短有意义；
尽量避免单个字符的变量名（临时变量除外）；
所有变量要显式的赋值；
静态变量命名采用全大写字母，中间用下划线分隔。
Example:
```java
    int total = 0;
    String name = null;
    Button bnStart = new Button();
    static int MAX_CODE_LINE = 200;
```

**4、资源文件命名规则**

因为该文件名不识别大写字符，所有单词间以下划线分割。布局文件以layout开头，选择器用selector开头
Example:
```java
    layout_main
    selector_btn_click
```
> 附：常用控件缩写对应表
TextView ---> tv
EditText ---> et
WebView ---> wv
ImageView ---> iv
VideoView ---> vv
MediaController ---> mc
ListView ---> lv
GridView ---> gv
Gallery ---> gly
Button ---> btn
ImageButton ---> ib
CheckBox ---> cb
RadioButton ---> rb
SeekBar ---> sb
ProgressBar ---> pb
Spinner ---> spr
SearchView ---> sv
AnalogClock ---> ac
TimePicker ---> tp
DatePicker ---> dp

#### 注释规范

**1、类说明注释**

该注释一般位于package/import语句之前，class描述之前。要求至少写出内容说明、创建者、创建时间和特别注意事项等内容。
Example:
 ```java
/**
 * 对类的说明
 * @author 张三
 * @time ${date}${time}
 * @version
 */
 ```
 
**2、方法说明注释**

对每个方法都应有适当的说明，位于方法声明之前，包括：说明、参数说明、异常说明、返回值说明和特别说明等
Example:
 ```java
/* 方法的一句话概述 
 * 方法详述（简单方法可不必详述） 
 * @param s 说明参数含义 
 * @return 说明返回值含义  
 * @throws IOException 说明发生此异常的条件  
 * @throws NullPointerException 说明发生此异常的条件
 */
 ```
**3、变量说明注释**

 对于类的全局变量，如果变量名不能直接说明其用途的，需要添加变量说明。格式如下
 Example:
 ```java
/*
 *是否是超级管理员
 */
private boolean isAdmin = false;
```

**4、方法体内代码的注释**

对于较复杂的方法，需要说明其处理思路。方法内部的注释：如果为多行，使用/*...*/形式，如果为单行，使用//形式的注释。不要在方法内部使用java doc形式的注释

**5、资源文件代码注释**

资源文件也需要添加注释。
 Example:
 ```java
<!-- QSB -->
    <dimen name="toolbar_button_vertical_padding">4dip</dimen>
    <dimen name="toolbar_button_horizontal_padding">12dip</dimen>
    <!-- External toolbar icon size (for bounds) -->
    <dimen name="toolbar_external_icon_width">36dp</dimen>
    <dimen name="toolbar_external_icon_height">36dp</dimen>

<!-- AllApps/Customize/AppsCustomize -->
    <!-- The height of the tab bar -->
    <dimen name="apps_customize_tab_bar_height">52dp</dimen>
```
	
**特别说明：多使用IDE自带的注释补全和格式化功能，保证提交时，代码是格式化了的**



### 目录结构

#### 按逻辑组件

![enter description here][1]
久忆日记

#### 按功能模块

![enter description here][2]
微盘
#### 应用与反思

![enter description here][3]

![enter description here][4]

![enter description here][5]

1).功能模块和类型模块均可以划分，如果没有需要的话，模块的划分都可以省略。

2).activity和service这类组件划分，如果没有需要的话，组件的划分都可以省略。

3).所有的划分，如果没有需要的话，所有的划分都可以省略。



  [1]: ./images/1484185563991.jpg "1484185563991.jpg"
  [2]: ./images/1484185552446.jpg "1484185552446.jpg"
  [3]: ./images/1484186152752.jpg "1484186152752.jpg"
  [4]: ./images/1484186257047.jpg "1484186257047.jpg"
  [5]: ./images/1484186659730.jpg "1484186659730.jpg"