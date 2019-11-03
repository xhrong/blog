---
title: PPT to Image方案
tags: [java,C#,PPT]
grammar_cjkRuby: true
date: 2019-11-02
categories: [Windows]
---

PPT to Image可通过PPT直接转成Image，或先转成PDF、HTML、TIF等格式后，再转成Image。下述内容，总结PPT转Image的几种方案。

### 基于DCOM的方案

```C#
 private void button1_Click(object sender, RoutedEventArgs e)
        {

           //选择一个ppt文件
            System.Windows.Forms.OpenFileDialog OpenFileDialog1 = new System.Windows.Forms.OpenFileDialog();
    
            OpenFileDialog1.RestoreDirectory = true;
            string file;
            if (OpenFileDialog1.ShowDialog() == System.Windows.Forms.DialogResult.OK)
            {
                file = OpenFileDialog1.FileName;


            }
            else
                file = null;
            if (file != null)
            {
                string rootdir = Environment.CurrentDirectory;
                string path = rootdir + @"\PPT";
                Convert(file, path, PpSaveAsFileType.ppSaveAsJPG);
            }

        }
        private bool Convert(string sourcePath, string targetPath, PpSaveAsFileType targetFileType)
        {
            bool result;
            object missing = Type.Missing;
            Microsoft.Office.Interop.PowerPoint.Application application = null;

            Presentation persentation = null;
            Presentation persentationCopy = null;
            
           
                application = new Application();
                persentation = application.Presentations.Open(sourcePath, MsoTriState.msoTrue, MsoTriState.msoFalse, MsoTriState.msoFalse);
             //   persentation.SaveAs(targetPath, targetFileType, Microsoft.Office.Core.MsoTriState.msoTrue);                    //整个ppt的文件转换为其他的格式
              //  persentation.Slides[1].Export(targetPath + "\\ppt.jpg", "JPG", 800, 600);                   //将ppt中的某张转换为图片文件
                persentation.SaveAs(targetPath, targetFileType, Microsoft.Office.Core.MsoTriState.msoTrue);    //整个ppt的文件转换为其他的格式
                result = true;
          
           
                if (persentation != null)
                {
                    persentation.Close();
                    persentation = null;
                }
                if (application != null)
                {
                    application.Quit();
                    application = null;
                }
                GC.Collect();
                GC.WaitForPendingFinalizers();
                GC.Collect();
                GC.WaitForPendingFinalizers();
            
            return result;
        }
    }
```

DCOM方案可通过C#、C++或Python语言实现，但本质上都是调用COM组件，对PPT本身的使用是有影响的。

### 基于POI开源库的方案

```java
/*
 *  ====================================================================
 *    Licensed to the Apache Software Foundation (ASF) under one or more
 *    contributor license agreements.  See the NOTICE file distributed with
 *    this work for additional information regarding copyright ownership.
 *    The ASF licenses this file to You under the Apache License, Version 2.0
 *    (the "License"); you may not use this file except in compliance with
 *    the License.  You may obtain a copy of the License at
 *
 *        http://www.apache.org/licenses/LICENSE-2.0
 *
 *    Unless required by applicable law or agreed to in writing, software
 *    distributed under the License is distributed on an "AS IS" BASIS,
 *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *    See the License for the specific language governing permissions and
 *    limitations under the License.
 * ====================================================================
 */

package com.poi.test;

import java.awt.Dimension;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.awt.image.BufferedImage;
import java.io.File;
import java.util.List;
import java.util.Locale;
import java.util.Set;
import java.util.TreeSet;

import javax.imageio.ImageIO;

import org.apache.poi.sl.draw.DrawFactory;
import org.apache.poi.sl.usermodel.Slide;
import org.apache.poi.sl.usermodel.SlideShow;
import org.apache.poi.sl.usermodel.SlideShowFactory;

/**
 * An utility to convert slides of a .pptx slide show to a PNG image
 *
 * @author Yegor Kozlov
 */
public class Main {

    static void usage(String error){
        String msg =
                "Usage: PPTX2PNG [options] <ppt or pptx file>\n" +
                        (error == null ? "" : ("Error: "+error+"\n")) +
                        "Options:\n" +
                        "    -scale <float>   scale factor\n" +
                        "    -slide <integer> 1-based index of a slide to render\n" +
                        "    -format <type>   png,gif,jpg (,null for testing)" +
                        "    -outdir <dir>    output directory, defaults to origin of the ppt/pptx file" +
                        "    -quiet           do not write to console (for normal processing)";

        System.out.println(msg);
        // no System.exit here, as we also run in junit tests!
    }

    public static void main(String[] args) throws Exception {

        String slidenumStr = "-1";
        float scale = 1.5f;
        File file = new File("C:\\Users\\xhrong\\Desktop\\13.pptx");
        String format = "jpg";
        File outdir =  new File("C:\\Users\\xhrong\\Desktop\\poi");
        boolean quiet = false;

        try (SlideShow<?, ?> ss = SlideShowFactory.create(file, null, true)) {
            List<? extends Slide<?, ?>> slides = ss.getSlides();

            Set<Integer> slidenum = slideIndexes(slides.size(), slidenumStr);

            if (slidenum.isEmpty()) {
                usage("slidenum must be either -1 (for all) or within range: [1.." + slides.size() + "] for " + file);
                return;
            }

            Dimension pgsize = ss.getPageSize();
            int width = (int) (pgsize.width * scale);
            int height = (int) (pgsize.height * scale);

            long time=System.currentTimeMillis();
            long start =time;
            for (Integer slideNo : slidenum) {
                Slide<?, ?> slide = slides.get(slideNo);
                String title = slide.getTitle();
                if (!quiet) {
                    System.out.println("Rendering slide " + slideNo + (title == null ? "" : ": " + title));
                }

                BufferedImage img = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
                Graphics2D graphics = img.createGraphics();
                DrawFactory.getInstance(graphics);


               graphics.scale(scale, scale);

                // draw stuff
                slide.draw(graphics);

                // save the result
                if (!"null".equals(format)) {
                    String outname = file.getName().replaceFirst(".pptx?", "");
                    outname = String.format(Locale.ROOT, "%1$s-%2$04d.%3$s", outname, slideNo, format);
                    File outfile = new File(outdir, outname);
                    ImageIO.write(img, format, outfile);
                }

                System.out.println("用时："+(System.currentTimeMillis()-time));
                time = System.currentTimeMillis();
                graphics.dispose();
                img.flush();
            }

            System.out.println("总用时："+(System.currentTimeMillis()-start));
        }

        if (!quiet) {
            System.out.println("Done");
        }
    }

    private static Set<Integer> slideIndexes(final int slideCount, String range) {
        Set<Integer> slideIdx = new TreeSet<>();
        if ("-1".equals(range)) {
            for (int i=0; i<slideCount; i++) {
                slideIdx.add(i);
            }
        } else {
            for (String subrange : range.split(",")) {
                String idx[] = subrange.split("-");
                switch (idx.length) {
                    default:
                    case 0: break;
                    case 1: {
                        int subidx = Integer.parseInt(idx[0]);
                        if (subrange.contains("-")) {
                            int startIdx = subrange.startsWith("-") ? 0 : subidx;
                            int endIdx = subrange.endsWith("-") ? slideCount : Math.min(subidx,slideCount);
                            for (int i=Math.max(startIdx,1); i<endIdx; i++) {
                                slideIdx.add(i-1);
                            }
                        } else {
                            slideIdx.add(Math.max(subidx,1)-1);
                        }
                        break;
                    }
                    case 2: {
                        int startIdx = Math.min(Integer.parseInt(idx[0]), slideCount);
                        int endIdx = Math.min(Integer.parseInt(idx[1]), slideCount);
                        for (int i=Math.max(startIdx,1); i<endIdx; i++) {
                            slideIdx.add(i-1);
                        }
                        break;
                    }
                }
            }
        }
        return slideIdx;
    }
}
```
该方案不依赖PPT本身，独立性较好，但是转换后的图片存在一定的画面错位等问题

### 基于商业库的方案

#### Aspose

```C#
  public static class PdfHelper
    {
        public static bool PrintPdfFile()
         {
            String filePPT = "C:\\Users\\xhrong\\Desktop\\12.pptx";
             String file = "C:\\Users\\xhrong\\Desktop\\out.tif";

            String jpgF = "C:\\Users\\xhrong\\Desktop\\jpg";

            Console.Out.WriteLine("TIF is Start" + System.DateTime.Now.Second);

            ConvertToPDFWithAspose(filePPT, file);

            Console.Out.WriteLine("TIF is OK" + System.DateTime.Now.Second);


            ConvertTiff2Jpeg(file, jpgF);

            Console.Out.WriteLine("JPG is OK" + System.DateTime.Now.Second);


            Console.Read();
            return true;
        }


        public static void ConvertTiff2Jpeg(string tiffFileName, string jpegFileName)
        {
            var img = Image.FromFile(tiffFileName);
            var count = img.GetFrameCount(FrameDimension.Page);
            for (int i = 0; i < count; i++)
            {
                img.SelectActiveFrame(FrameDimension.Page, i);
                img.Save(jpegFileName + ".part" + i + ".jpg");
            }
    
            img.Dispose();
        }

        public static void ConvertToPDFWithAspose(string _lstrInputFile, string _lstrOutFile)
        {
            Aspose.Slides.Presentation ppt = new Aspose.Slides.Presentation(_lstrInputFile);
          //  ppt.Save(_lstrInputFile,)
            ppt.Save(_lstrOutFile, Aspose.Slides.Export.SaveFormat.Tiff);
            
        }
    }

```

Aspose方案不依赖PPT本身，转换效果较好，但仍然存在错位问题。


#### OpenOffice

```java

/*
 *  ====================================================================
 *    Licensed to the Apache Software Foundation (ASF) under one or more
 *    contributor license agreements.  See the NOTICE file distributed with
 *    this work for additional information regarding copyright ownership.
 *    The ASF licenses this file to You under the Apache License, Version 2.0
 *    (the "License"); you may not use this file except in compliance with
 *    the License.  You may obtain a copy of the License at
 *
 *        http://www.apache.org/licenses/LICENSE-2.0
 *
 *    Unless required by applicable law or agreed to in writing, software
 *    distributed under the License is distributed on an "AS IS" BASIS,
 *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *    See the License for the specific language governing permissions and
 *    limitations under the License.
 * ====================================================================
 */

package com.poi.test;

import com.artofsolving.jodconverter.DefaultDocumentFormatRegistry;
import com.artofsolving.jodconverter.DocumentConverter;
import com.artofsolving.jodconverter.DocumentFormat;
import com.artofsolving.jodconverter.openoffice.connection.OpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.connection.SocketOpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.converter.OpenOfficeDocumentConverter;
import org.apache.commons.io.FilenameUtils;
import org.icepdf.core.pobjects.Document;
import org.icepdf.core.pobjects.Page;
import org.icepdf.core.util.GraphicsRenderingHints;

import javax.imageio.ImageIO;
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Rectangle2D;
import java.awt.image.BufferedImage;
import java.awt.image.RenderedImage;
import java.io.*;
import java.util.*;
import java.util.List;
/**
 * An utility to convert slides of a .pptx slide show to a PNG image
 *
 * @author Yegor Kozlov
 */
public class MainNN {


   // private static Logger logger = Logger.getLogger(MainNN.class);
    private final static long MAXSIZE = 80000000; // 最大限制转化大小8MB

    public static File convertFileToPdf(File sourceFile, String pdfTempPath) throws IOException {
        if (sourceFile.length() > MAXSIZE) {
            throw new IOException("the size of file  out of range");
        }

       // String fileType = FileOperation.getFileType(sourceFile.getName());

        // 临时pdf文件
        File pdfFile = new File(pdfTempPath);
        if (!pdfFile.exists()) {
            pdfFile.createNewFile();
        }
        InputStream inputStream = null;
        OutputStream outputStream = null;

     //   if (!fileType.equals("pdf")) {

            // 获得文件格式
            DefaultDocumentFormatRegistry formatReg = new DefaultDocumentFormatRegistry();
            DocumentFormat pdfFormat = formatReg.getFormatByFileExtension("pdf");
            DocumentFormat docFormat = formatReg.getFormatByFileExtension("ppt");

            /**
             * 在此之前需先开启openoffice服务，用命令行打开cd C:\Program Files\OpenOffice.org 3\program （openoffice安装的路径）
             * 输入 soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard
             */
            OpenOfficeConnection connection = new SocketOpenOfficeConnection(8100);

            try {
                // stream 流的形式
                inputStream = new FileInputStream(sourceFile);
                outputStream = new FileOutputStream(pdfFile);
                connection.connect();
                DocumentConverter converter = new OpenOfficeDocumentConverter(connection);

                converter.convert(inputStream, docFormat, outputStream, pdfFormat);

            } catch (Exception e) {
                e.printStackTrace();
         //       throw new BusinessException(e.getMessage());
            } finally {
                if (connection != null) {
                    connection.disconnect();
                    connection = null;
                }

                try {
                    inputStream.close();
                    outputStream.close();
                } catch (IOException e) {

                    e.printStackTrace();
                }

            }
            System.out.println("转化pdf成功....");


        try {
            pdf2Imgs(pdfTempPath,"C:/Users/xhrong/Desktop/poi","mmg");
        } catch (Exception e) {
            e.printStackTrace();
        }
        //logger.info("转化pdf成功....");

//        } else {
//            // 复制pdf文件到新的文件
//            FileUtils.copyFile(sourceFile, pdfFile);
//
//        }

        return pdfFile;
    }

    /**
     * 将pdf转换成图片
     *
     * @param pdfPath
     * @param imagePath
     * @return 返回转换后图片的名字
     * @throws Exception
     */
    private static List<String> pdf2Imgs(String pdfPath, String imgDirPath,String fileName) throws Exception {
        Document document = new Document();
        document.setFile(pdfPath);
        float scale = 2f;//放大倍数
        float rotation = 0f;//旋转角度
        List<String> imgNames = new ArrayList<String>();
        int pageNum = document.getNumberOfPages();
        File imgDir = new File(imgDirPath);
        if (!imgDir.exists()) {
            imgDir.mkdirs();
        }
        for (int i = 0; i < pageNum; i++) {
            BufferedImage image = (BufferedImage) document.getPageImage(i, GraphicsRenderingHints.SCREEN,
                    Page.BOUNDARY_CROPBOX, rotation, scale);
            RenderedImage rendImage = image;
            try {
                String filePath = imgDirPath + File.separator +fileName+i + ".jpg";
                File file = new File(filePath);
                ImageIO.write(rendImage, "jpg", file);
                imgNames.add(FilenameUtils.getName(filePath));
            } catch (IOException e) {
                e.printStackTrace();
                return null;
            }
            image.flush();
        }
        document.dispose();
        return imgNames;
    }
    public static void main(final String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
				/*try {
					FileToPdfUtils.setup("D:/pdf-test/FreeMarker_Manual_zh_CN.pdf", "D:/pdf-test");
				} catch (IOException ex) {
					ex.printStackTrace();
				}*/
                try {
                    convertFileToPdf(new File("C:\\Users\\xhrong\\Desktop\\12.pptx"), "C:/Users/xhrong/Desktop/poi/5.pdf");
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        });
    }
}

```

OpenOffice方案，需要安装OpenOffice，并且需要启动服务进程，通过RPC方式调用本地服务。比较适合服务端做PPT转换，不太适用于PC端个人使用。

#### Spire.Presentation

```java

package com.poi.test;


import com.spire.presentation.Presentation;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;

/**
 * An utility to convert slides of a .pptx slide show to a PNG image
 */
public class MainPP {

    public static void main(String[] args) throws Exception {
        //加载PPT
        String file = "C:\\Users\\xhrong\\Desktop\\13.pptx";
        String format = "jpg";
        File outdir =  new File("C:\\Users\\xhrong\\Desktop\\poi");
        Presentation ppt = new Presentation();
        ppt.loadFromFile(file);

        long time=System.currentTimeMillis();
//保存为图片
        for (int i = 0; i < ppt.getSlides().getCount(); i++) {
            BufferedImage image = ppt.getSlides().get(i).saveAsImage();
            String fileName = "C:/Users/xhrong/Desktop/poi/" + String.format("ToImage-%1$s.jpg", i);
            ImageIO.write(image, "jpg", new File(fileName));

            System.out.println("用时："+(System.currentTimeMillis()-time));
            time = System.currentTimeMillis();
        }
        ppt.dispose();
    }

}
```

Spire方案不依赖于PPT本身，可独立使用。转出来的PPT效果较好，但是转换过程较慢，可能是免费版本的问题。

### 方案比较

		
![enter description here](./images/1572698753325.png)


Demo   : https://github.com/xhrong/blog/tree/master/source/attachments/ppt2img.rar