---
title: Android抽取PPT、视频、图片缩略图
tags: [缩略图]
grammar_cjkRuby: true
categories: [Android]
date: 2018-08-11
---


### 使用POI抽取PPT缩略图

方案：将POI库向Andoid迁移，并用来获取PPT的缩略图。
（POI还可以用户解析和创建Office系列文档）

```java
package com.xhr.lib;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Environment;
import android.util.Log;

import org.apache.poi.poifs.crypt.EncryptionInfoBuilder;
import org.apache.poi.xslf.usermodel.XMLSlideShow;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.UUID;

/**
 * Created by xhrong on 2018/8/10.
 *
 * 使用POI生成PPT的缩略图
 */

public class PPTThumbGenerator extends ThumbGenerator {

    private static final String SD_PATH = Environment.getExternalStorageDirectory()+"/thumb/ppt/";
    static {
        System.setProperty("org.apache.poi.javax.xml.stream.XMLInputFactory", "com.fasterxml.aalto.stax.InputFactoryImpl");
        System.setProperty("org.apache.poi.javax.xml.stream.XMLOutputFactory", "com.fasterxml.aalto.stax.OutputFactoryImpl");
        System.setProperty("org.apache.poi.javax.xml.stream.XMLEventFactory", "com.fasterxml.aalto.stax.EventFactoryImpl");
    }

    @Override
    public String generatorThumbnail(String filePath) {
        try {
            XMLSlideShow xmlSlideShow = new XMLSlideShow(new FileInputStream(filePath));
            InputStream inputStream = xmlSlideShow.getProperties().getThumbnailImage();
            Bitmap bitmap = BitmapFactory.decodeStream(inputStream);

            return saveBitmap(bitmap);

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    /**
     * 随机生产文件名
     *
     * @return
     */
    private static String generateFileName() {
        return UUID.randomUUID().toString();
    }

    /**
     * 保存bitmap到本地
     *
     * @param mBitmap
     * @return
     */
    private String saveBitmap(Bitmap mBitmap) {
        String savePath = SD_PATH + generateFileName() + ".jpg";
        return saveThumbnail(savePath, mBitmap);
    }
}
```

依赖库：https://github.com/centic9/poi-on-android

### 使用ThumbnailUtils抽取视频缩略图

```java
package com.xhr.lib;

import android.graphics.Bitmap;
import android.media.ThumbnailUtils;
import android.os.Environment;
import android.provider.MediaStore.Video.Thumbnails;

import java.util.UUID;

/**
 * Created by xhrong on 2018/8/10.
 *
 *  使用ThumbnailUtils生成视频缩略图
 */

public class VideoThumbGenerator extends ThumbGenerator {

    private static final String SD_PATH = Environment.getExternalStorageDirectory() + "/thumb/video/";

    @Override
    public String generatorThumbnail(String filePath) {
        Bitmap bitmap = ThumbnailUtils.createVideoThumbnail(filePath, Thumbnails.MICRO_KIND);

        bitmap = ThumbnailUtils.extractThumbnail(bitmap, 100, 100,
                ThumbnailUtils.OPTIONS_RECYCLE_INPUT);

        return saveBitmap(bitmap);
    }

    /**
     * 随机生产文件名
     *
     * @return
     */
    private static String generateFileName() {
        return UUID.randomUUID().toString();
    }

    /**
     * 保存bitmap到本地
     *
     * @param mBitmap
     * @return
     */
    private String saveBitmap(Bitmap mBitmap) {
        String savePath = SD_PATH + generateFileName() + ".jpg";
        return saveThumbnail(savePath, mBitmap);
    }

}
```
### 使用ThumbnailUtils抽取图片缩略图

```java
package com.xhr.lib;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.media.ThumbnailUtils;
import android.os.Environment;

import java.util.UUID;

/**
 * Created by xhrong on 2018/8/10.
 *
 * 使用ThumbnailUtils生成图片缩略图
 */

public class ImageThumbGenerator extends ThumbGenerator {

    private static final String SD_PATH = Environment.getExternalStorageDirectory() + "/thumb/image/";

    @Override
    public String generatorThumbnail(String filePath) {
        Bitmap source= BitmapFactory.decodeFile(filePath);

       Bitmap bitmap = ThumbnailUtils.extractThumbnail(source, 100, 100,
                ThumbnailUtils.OPTIONS_RECYCLE_INPUT);

        return saveBitmap(bitmap);
    }

    /**
     * 随机生产文件名
     *
     * @return
     */
    private static String generateFileName() {
        return UUID.randomUUID().toString();
    }

    /**
     * 保存bitmap到本地
     *
     * @param mBitmap
     * @return
     */
    private String saveBitmap(Bitmap mBitmap) {
        String savePath = SD_PATH + generateFileName() + ".jpg";
        return saveThumbnail(savePath, mBitmap);
    }

}

```


Demo：https://github.com/xhrong/ThumbGen