---
title: 汉字转拼音的几种方案（JAVA）
tags: [汉字,拼音,Java]
grammar_cjkRuby: true
categories: [Java]
date: 2017-04-08
---


### 基于纯字典的方案

#### 原理

通过查询预先编制的汉字--->拼音的映射表，确定单字的拼音。

能够获取单字的多个拼音，但是无法确定哪一个适用于当前状态。

#### pinyin4j

##### 特点

1、支持多种拼音类型，如：Hanyu Pinyin、Tongyong Pinyin、Wade-Giles、MPS2 (Mandarin Phonetic Symbols II)、Yale Romanization、Gwoyeu Romatzyh。（其实除了Hanyu Pinyin,其他并没什么用途）

2、支持返回单字的多个拼音，如：“'和”字，可以获取“hé hè huó huò huo hāi he hú”

3、可以获取多种拼音格式，包括With tone numbers (lü3) or tone marks (lǚ) or without tone (lü)

##### 用法
pinyin4j提供的工具类为PinyinHelper,里边提供了静态方法toHanyuPinyinString()和toHanyuPinyinStringArray(),由于这两个方法都涉及到一个HanyuPinyinOutputFormat类型的参数，我们先看一下pinyin4j中的辅助类。

pinyin4j中有四个辅助类分别是：HanyuPinyinCaseType、HanyuPinyinToneType、HanyuPinyinVCharType、HanyuPinyinOutputFormat

(1)HanyuPinyinCaseType类，提供了两个静态常量UPPERCASE和LOWERCASE，表示转换后的拼音为大写和小写

(2)HanyuPinyinToneType类，提供了两个静态常量WITH_TONE_NUMBER和WITHOUT_TONE，表示转换后的拼音是否带声调，还拿汉字"解"说明，如果我们设置了HanyuPinyinToneType类型为WITH_TONE_NUMBER，我们会取到jie3；如果类型是WITHOUT_TONE，取到的是jie

(3)HanyuPinyinVCharType类，提供了三个静态常量WITH_U_AND_COLON、WITH_V和WITH_U_UNICODE，之所以有这个类是为了对拼音中的一种特殊情况做处理，比如"吕"，我们键盘打字是"lv"，而小学课本上是"lü",而在pinyin4j使用的属性文件中存储的是(lu:3)，所以该类提供了三种类型，分别表示转换后的格式为"lu:"、"lv"和"lü"

(4)HanyuPinyinOutputFormat类，该类就是用来定制转换后拼音格式的类，可以设置以上三种类型，默认设置为 HanyuPinyinVCharType.WITH_U_AND_COLON、 HanyuPinyinCaseType.LOWERCASE、 HanyuPinyinToneType.WITH_TONE_NUMBER
```java
HanyuPinyinOutputFormat format = new HanyuPinyinOutputFormat();  
  
// UPPERCASE：大写  (ZHONG)  
// LOWERCASE：小写  (zhong)  
format.setCaseType(HanyuPinyinCaseType.LOWERCASE);  
  
// WITHOUT_TONE：无音标  (zhong)  
// WITH_TONE_NUMBER：1-4数字表示英标  (zhong4)  
// WITH_TONE_MARK：直接用音标符（必须WITH_U_UNICODE否则异常）  (zhòng)  
format.setToneType(HanyuPinyinToneType.WITH_TONE_MARK);  
  
// WITH_V：用v表示ü  (nv)  
// WITH_U_AND_COLON：用"u:"表示ü  (nu:)  
// WITH_U_UNICODE：直接用ü (nü)  
format.setVCharType(HanyuPinyinVCharType.WITH_U_UNICODE);  
          
String[] pinyin = PinyinHelper.toHanyuPinyinStringArray('重', format);  
```
####  JPinyin
JPinyin是一个汉字转拼音的Java开源类库，在PinYin4j的功能基础上做了一些改进。支持添加用户自定义字典；

#### TinyPinyin

在PinYin4j的功能基础上做了一些改进。转换后的结果不包含声调和方言。支持自定义词典，方便处理多音字；

### 基于字典+中文分词的方案
#### 原理
1、通过分词技术将句子分解成单字和词语
2、通过查询预先编制的汉字词--->拼音的映射表，确定单字和词语的拼音

能够在词语的级别上解决多音词问题，但是对分词结果为单字的情况，仍然无法确定合适的拼音。例如：“朝 辞 白帝 彩云 间”的“朝”字无法正确处理。
#### Hanlp
##### 特点
HanLP是由一系列模型与算法组成的Java工具包，目标是普及自然语言处理在生产环境中的应用。HanLP具备功能完善、性能高效、架构清晰、语料时新、可自定义的特点。HanLP提供下列功能：

中文分词
最短路分词
N-最短路分词
CRF分词
索引分词
极速词典分词
用户自定义词典
词性标注
命名实体识别
中国人名识别
音译人名识别
日本人名识别
地名识别
实体机构名识别
关键词提取
TextRank关键词提取
自动摘要
TextRank自动摘要
短语提取
基于互信息和左右信息熵的短语提取
拼音转换
多音字
声母
韵母
声调
简繁转换
繁体中文分词
简繁分歧词（简体、繁体、臺灣正體、香港繁體）
文本推荐
语义推荐
拼音推荐
字词推荐
依存句法分析
基于神经网络的高性能依存句法分析器
MaxEnt依存句法分析
CRF依存句法分析
语料库工具
分词语料预处理
词频词性词典制作
BiGram统计
词共现统计
CoNLL语料预处理
CoNLL UA/LA/DA评测工具

##### 使用方法
```java
/**
 * 汉字转拼音
 * @author hankcs
 */
public class DemoPinyin
{
    public static void main(String[] args)
    {
        String text = "重载不是重任";
        List<Pinyin> pinyinList = HanLP.convertToPinyinList(text);
        System.out.print("原文,");
        for (char c : text.toCharArray())
        {
            System.out.printf("%c,", c);
        }
        System.out.println();

        System.out.print("拼音（数字音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin);
        }
        System.out.println();

        System.out.print("拼音（符号音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getPinyinWithToneMark());
        }
        System.out.println();

        System.out.print("拼音（无音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getPinyinWithoutTone());
        }
        System.out.println();

        System.out.print("声调,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getTone());
        }
        System.out.println();

        System.out.print("声母,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getShengmu());
        }
        System.out.println();

        System.out.print("韵母,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getYunmu());
        }
        System.out.println();

        System.out.print("输入法头,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getHead());
        }
        System.out.println();
    }
}
```
说明
HanLP不仅支持基础的汉字转拼音，还支持声母、韵母、音调、音标和输入法首字母首声母功能。
HanLP能够识别多音字，也能给繁体中文注拼音。
最重要的是，HanLP采用的模式匹配升级到AhoCorasickDoubleArrayTrie，性能大幅提升，能够提供毫秒级的响应速度！

https://github.com/hankcs/HanLP

### 基于TTS的方案
#### 原理

基于TTS能够比较好地解决多音字问题。

#### 科大讯飞TTS

目前没有开放的SDK或服务，但是BBT中已经有私有的成熟的应用了，效果很好。（不过，性能可能是个问题）