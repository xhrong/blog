---
date: 2016-12-01 14:30:01
title: Android截屏技术总结
tags: [Android]
categories: [Android]
---
## Android截屏技术总结

### 一、概述
Android截屏的几种场景

 - 场景一：截取应用内的固定Activity界面
   - 针对普通View
   - 针对WebView
 - 场景二：截取全局屏幕界面
   - 基于Root方案
   - 针对Android5.0之后版本的方案
 - 场景三：通过PC对Android进行截屏
   - 使用ddmlib.jar
   - 使用ADB工具

### 二、截取应用内固定Activity的普通View
 >原理：利用View.getDrawingCache()方法获取View的绘制缓存，保存为图片
 
 Sample:
```java
    /**
     * 根据view来生成bitmap图片，可用于截图功能
     */
    public static Bitmap getViewBitmap(View v) {
        v.clearFocus();
        v.setPressed(false);
        // 能画缓存就返回false
        boolean willNotCache = v.willNotCacheDrawing();
        v.setWillNotCacheDrawing(false);
        int color = v.getDrawingCacheBackgroundColor();
        v.setDrawingCacheBackgroundColor(0);
        if (color != 0) {
            v.destroyDrawingCache();
        }
        v.buildDrawingCache();
        Bitmap cacheBitmap = v.getDrawingCache();
        if (cacheBitmap == null) {
            return null;
        }
        Bitmap bitmap = Bitmap.createBitmap(cacheBitmap);
        // Restore the view
        v.destroyDrawingCache();
        v.setWillNotCacheDrawing(willNotCache);
        v.setDrawingCacheBackgroundColor(color);
        return bitmap;
    }

    /**
     * 保存Bitmap图片为本地文件
     */
    public static void saveFile(Bitmap bitmap, String filename) {
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream(filename);
            if (fileOutputStream != null) {
                bitmap.compress(Bitmap.CompressFormat.PNG, 90, fileOutputStream);
                fileOutputStream.flush();
                fileOutputStream.close();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

这种截图方式仅能截取特点View的画面，如果设计合理，能够满足大部分应用需求了。另外，这种方式可以延伸一下，通过截取Activity而获取图片。代码如下：
```java
    public static Bitmap captureScreen(Activity activity) {
        activity.getWindow().getDecorView().setDrawingCacheEnabled(true);
        Bitmap bmp=getWindow().getDecorView().getDrawingCache();
        return bmp;
    }
```

备注：以上方案针对WebView会出现白屏或黑屏现象。

### 三、截取应用内固定Activity的WebView
>原理：针对View进行截屏适用于WebView会出现白屏或黑屏的问题，可以通过WebView的capturePicture()方法获取图片，解决该问题

Sample
```java
    /**
     * 截取webView可视区域的截图
     *
     * @param webView 前提：WebView要设置webView.setDrawingCacheEnabled(true);
     * @return
     */
    public static Bitmap captureWebViewVisibleSize(WebView webView) {
        Bitmap bmp = webView.getDrawingCache();
        return bmp;
    }

    /**
     * 截取webView快照(webView加载的整个内容的大小)
     *
     * @param webView
     * @return
     */
    public static Bitmap captureWebView(WebView webView) {
        Picture snapShot = webView.capturePicture();
        Bitmap bmp = Bitmap.createBitmap(snapShot.getWidth(),
                snapShot.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bmp);
        snapShot.draw(canvas);
        return bmp;
    }
```
备注：capturePicture(）方法在API Level 19后被标记为deprecated，但是目前还可以使用

### 四、截取全局屏幕：Root方案
以下方案都Root之后才能成功
>**原理**：在一般的linux文件系统中，通过/dev/fb0设备文件来提供给应用程序对framebuffer进行读写的访问。这里，如果有多个显示设备，就将依次出现fb1,fb2,…等文件。而在我们所说的android系统中，这个设备文件被放在了/dev/graphics/fb0，而且往往只有这一个。这种方法使用的最为普遍，linux系统中经常使用这种方法实现截屏，一般步骤是：
 1. 读取framebuffer
 2. Framebuffer转换为bitmap
 3. bitmap生成图像文件

Sample:
```java
    /**
     * 截屏
     *
     * @param activity
     * @return
     */
    public static Bitmap captureScreen(Activity activity) {
        // 获取屏幕大小：
        DisplayMetrics metrics = new DisplayMetrics();
        WindowManager WM = (WindowManager) activity
                .getSystemService(Context.WINDOW_SERVICE);
        Display display = WM.getDefaultDisplay();
        display.getMetrics(metrics);
        int height = metrics.heightPixels; // 屏幕高
        int width = metrics.widthPixels; // 屏幕的宽
        // 获取显示方式
        int pixelformat = display.getPixelFormat();
        PixelFormat localPixelFormat1 = new PixelFormat();
        PixelFormat.getPixelFormatInfo(pixelformat, localPixelFormat1);
        int deepth = localPixelFormat1.bytesPerPixel;// 位深
        byte[] piex = new byte[height * width * deepth];
        try {
            Runtime.getRuntime().exec(
                    new String[]{"/system/bin/su", "-c",
                            "chmod 777 /dev/graphics/fb0"});
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            // 获取fb0数据输入流
            InputStream stream = new FileInputStream(new File(
                    "/dev/graphics/fb0"));
            DataInputStream dStream = new DataInputStream(stream);
            dStream.readFully(piex);
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 保存图片
        int[] colors = new int[height * width];
        for (int m = 0; m < colors.length; m++) {
            int r = (piex[m * 4] & 0xFF);
            int g = (piex[m * 4 + 1] & 0xFF);
            int b = (piex[m * 4 + 2] & 0xFF);
            int a = (piex[m * 4 + 3] & 0xFF);
            colors[m] = (a << 24) + (r << 16) + (g << 8) + b;
        }
        // piex生成Bitmap
        Bitmap bitmap = Bitmap.createBitmap(colors, width, height,
                Bitmap.Config.ARGB_8888);
        return bitmap;
    }
```
在AndroidManifest.xml加上两行权限：
```xml
<uses-permission android:name=”android.permission.WRITE_EXTERNAL_STORAGE” />
<uses-permission android:name=”android.permission.READ_FRAME_BUFFER” />
```
这方案有个变种，直接利用screencap命令进行截屏,本质上该命令读取系统的framebuffer，需要获得系统权限（而直接通过ADB调用screencap命令是不需要ROOT的）
```java
    /**
     * 使用screencap命令进行截屏，本质上该命令读取系统的framebuffer，需要获得系统权限
     */
    public void takeScreenShot() {
        String mSavedPath = Environment.getExternalStorageDirectory() + File.separator + "screenshot.png";
        try {
            Runtime.getRuntime().exec("screencap -p " + mSavedPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

### 五、截取全局屏幕：Android5.0之后版本的方案
　　Google从Android 5.0 开始，给出了截屏案例ScreenCapture，在同版本的examples的Media类别中可以找到。给需要开发手机或平板截屏应用的小伙伴提供了非常有意义的参考资料，由于以前版本的API是隐藏的，要想开发一个截屏应用需要费一番心思且有局限性。当然了，这里说的截屏不是应用程序本身，而是包括状态栏在内的整个屏幕，不管当前运行的是什么程序，效果同按下手机自带截屏快捷键一样
　　Sample：
```java
public class Service1 extends Service
{
    private LinearLayout mFloatLayout = null;
    private WindowManager.LayoutParams wmParams = null;
    private WindowManager mWindowManager = null;
    private LayoutInflater inflater = null;
    private ImageButton mFloatView = null;

    private static final String TAG = "MainActivity";

    private SimpleDateFormat dateFormat = null;
    private String strDate = null;
    private String pathImage = null;
    private String nameImage = null;

    private MediaProjection mMediaProjection = null;
    private VirtualDisplay mVirtualDisplay = null;

    public static int mResultCode = 0;
    public static Intent mResultData = null;
    public static MediaProjectionManager mMediaProjectionManager1 = null;

    private WindowManager mWindowManager1 = null;
    private int windowWidth = 0;
    private int windowHeight = 0;
    private ImageReader mImageReader = null;
    private DisplayMetrics metrics = null;
    private int mScreenDensity = 0;

    @Override
    public void onCreate()
    {
        // TODO Auto-generated method stub
        super.onCreate();

        createFloatView();

        createVirtualEnvironment();
    }

    @Override
    public IBinder onBind(Intent intent)
    {
        // TODO Auto-generated method stub
        return null;
    }

    private void createFloatView()
    {
        wmParams = new WindowManager.LayoutParams();
        mWindowManager = (WindowManager)getApplication().getSystemService(getApplication().WINDOW_SERVICE);
        wmParams.type = LayoutParams.TYPE_PHONE;
        wmParams.format = PixelFormat.RGBA_8888;
        wmParams.flags = LayoutParams.FLAG_NOT_FOCUSABLE;
        wmParams.gravity = Gravity.LEFT | Gravity.TOP;
        wmParams.x = 0;
        wmParams.y = 0;
        wmParams.width = WindowManager.LayoutParams.WRAP_CONTENT;
        wmParams.height = WindowManager.LayoutParams.WRAP_CONTENT;
        inflater = LayoutInflater.from(getApplication());
        mFloatLayout = (LinearLayout) inflater.inflate(R.layout.float_layout, null);
        mWindowManager.addView(mFloatLayout, wmParams);
        mFloatView = (ImageButton)mFloatLayout.findViewById(R.id.float_id);

        mFloatLayout.measure(View.MeasureSpec.makeMeasureSpec(0,
                View.MeasureSpec.UNSPECIFIED), View.MeasureSpec
                .makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));

        mFloatView.setOnTouchListener(new OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                // TODO Auto-generated method stub
                wmParams.x = (int) event.getRawX() - mFloatView.getMeasuredWidth() / 2;
                wmParams.y = (int) event.getRawY() - mFloatView.getMeasuredHeight() / 2 - 25;
                mWindowManager.updateViewLayout(mFloatLayout, wmParams);
                return false;
            }
        });

        mFloatView.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                // hide the button
                mFloatView.setVisibility(View.INVISIBLE);

                Handler handler1 = new Handler();
                handler1.postDelayed(new Runnable() {
                    public void run() {
                        //start virtual
                        startVirtual();
                    }
                }, 500);

                Handler handler2 = new Handler();
                handler2.postDelayed(new Runnable() {
                    public void run() {
                        //capture the screen
                        startCapture();
                    }
                }, 1500);

                Handler handler3 = new Handler();
                handler3.postDelayed(new Runnable() {
                    public void run() {
                        mFloatView.setVisibility(View.VISIBLE);
                        //stopVirtual();
                    }
                }, 1000);
            }
        });

        Log.i(TAG, "created the float sphere view");
    }

    private void createVirtualEnvironment(){
        dateFormat = new SimpleDateFormat("yyyy_MM_dd_hh_mm_ss");
        strDate = dateFormat.format(new java.util.Date());
        pathImage = Environment.getExternalStorageDirectory().getPath()+"/Pictures/";
        nameImage = pathImage+strDate+".png";
        mMediaProjectionManager1 = (MediaProjectionManager)getApplication().getSystemService(Context.MEDIA_PROJECTION_SERVICE);
        mWindowManager1 = (WindowManager)getApplication().getSystemService(Context.WINDOW_SERVICE);
        windowWidth = mWindowManager1.getDefaultDisplay().getWidth();
        windowHeight = mWindowManager1.getDefaultDisplay().getHeight();
        metrics = new DisplayMetrics();
        mWindowManager1.getDefaultDisplay().getMetrics(metrics);
        mScreenDensity = metrics.densityDpi;
        mImageReader = ImageReader.newInstance(windowWidth, windowHeight, 0x1, 2); //ImageFormat.RGB_565

        Log.i(TAG, "prepared the virtual environment");
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public void startVirtual(){
        if (mMediaProjection != null) {
            Log.i(TAG, "want to display virtual");
            virtualDisplay();
        } else {
            Log.i(TAG, "start screen capture intent");
            Log.i(TAG, "want to build mediaprojection and display virtual");
            setUpMediaProjection();
            virtualDisplay();
        }
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public void setUpMediaProjection(){
        mResultData = ((ShotApplication)getApplication()).getIntent();
        mResultCode = ((ShotApplication)getApplication()).getResult();
        mMediaProjectionManager1 = ((ShotApplication)getApplication()).getMediaProjectionManager();
        mMediaProjection = mMediaProjectionManager1.getMediaProjection(mResultCode, mResultData);
        Log.i(TAG, "mMediaProjection defined");
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void virtualDisplay(){
        mVirtualDisplay = mMediaProjection.createVirtualDisplay("screen-mirror",
                windowWidth, windowHeight, mScreenDensity, DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
                mImageReader.getSurface(), null, null);
        Log.i(TAG, "virtual displayed");
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void startCapture(){
        strDate = dateFormat.format(new java.util.Date());
        nameImage = pathImage+strDate+".png";

        Image image = mImageReader.acquireLatestImage();
        int width = image.getWidth();
        int height = image.getHeight();
        final Image.Plane[] planes = image.getPlanes();
        final ByteBuffer buffer = planes[0].getBuffer();
        int pixelStride = planes[0].getPixelStride();
        int rowStride = planes[0].getRowStride();
        int rowPadding = rowStride - pixelStride * width;
        Bitmap bitmap = Bitmap.createBitmap(width+rowPadding/pixelStride, height, Bitmap.Config.ARGB_8888);
        bitmap.copyPixelsFromBuffer(buffer);
        bitmap = Bitmap.createBitmap(bitmap, 0, 0,width, height);
        image.close();
        Log.i(TAG, "image data captured");

        if(bitmap != null) {
            try{
                File fileImage = new File(nameImage);
                if(!fileImage.exists()){
                    fileImage.createNewFile();
                    Log.i(TAG, "image file created");
                }
                FileOutputStream out = new FileOutputStream(fileImage);
                if(out != null){
                    bitmap.compress(Bitmap.CompressFormat.PNG, 100, out);
                    out.flush();
                    out.close();
                    Intent media = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
                    Uri contentUri = Uri.fromFile(fileImage);
                    media.setData(contentUri);
                    this.sendBroadcast(media);
                    Log.i(TAG, "screen image saved");
                }
            }catch(FileNotFoundException e) {
                e.printStackTrace();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void tearDownMediaProjection() {
        if (mMediaProjection != null) {
            mMediaProjection.stop();
            mMediaProjection = null;
        }
        Log.i(TAG,"mMediaProjection undefined");
    }

    private void stopVirtual() {
        if (mVirtualDisplay == null) {
            return;
        }
        mVirtualDisplay.release();
        mVirtualDisplay = null;
        Log.i(TAG,"virtual display stopped");
    }

    @Override
    public void onDestroy()
    {
        // to remove mFloatLayout from windowManager
        super.onDestroy();
        if(mFloatLayout != null)
        {
            mWindowManager.removeView(mFloatLayout);
        }
        tearDownMediaProjection();
        Log.i(TAG, "application destroy");
    }
}
```
　　从onCreate()方法开始，其调用了两个方法：createFloatView()和createVirtualEnvironment()；createFloatView()方法负责浮动小球的生成、拖动及其点击事件的响应；createVirtualEnvironment()方法定义截屏所需的变量（包括屏幕信息、图像格式、保存格式等等）。另外，各个方法利用Log日志类输出了运行过程中的状态信息，便于观察代码执行过程。
　　关键之处就在于对浮动小球点击事件的响应实现，而拖动只是附带的一个辅助功能而已。浮动小球点击事件的响应代码也完成了4件事情，下面一一进行分析。
　　1、隐藏小球
```java
mFloatView.setVisibility(View.INVISIBLE);
```
　　2、初始化截屏环境
```java
public void startVirtual(){
　　if (mMediaProjection != null) {
　　　　Log.i(TAG, "want to display virtual");
　　　　virtualDisplay();
　　} else {
　　　　Log.i(TAG, "start screen capture intent");
　　　　Log.i(TAG, "want to build mediaprojection and display virtual");
　　　　setUpMediaProjection();
　　　　virtualDisplay();
　　}
}
```
　　可以看出，这个过程先是对MediaProjection类实例mMediaProjection的值进行了判断，若之前没有初始化（即值为null），则调用setUpMediaProjection()方法获取共享数据并对其进行赋值；若已初始化，则直接调用virtualDisplay()方法利用之前定义的变量对截屏环境进行初始化，而真正执行最终操作的方法为createVirtualDisplay()。
　　3、屏幕截取
　　截屏环境初始化完成之后，便可以开始获取屏幕信息了，所以接下来调用的是startCapture()方法。该方法的实现和之前不同，也是容易出错的地方在于以下两句代码：
```java
int rowPadding = rowStride - pixelStride * width;
Bitmap bitmap = Bitmap.createBitmap(width+rowPadding/pixelStride, height, Bitmap.Config.ARGB_8888);
```
　　值得注意的是调用方法createBitmap()创建Bitmap对象时所用的第1、3个参数，分别对应于图像的宽度、格式。实现过程中发现，只有将格式设置为ARGB_8888才能获取想要的图像质量；而对于宽度，后面会解释为什么要为其设置偏移值。

　　4、显示小球
```java
mFloatView.setVisibility(View.VISIBLE);
```
　　备注：该截屏方式仅适用于Android5.0以上系统
### 六、其他
#### 1、在PC端使用Android ddmlib进行截屏
在DDMS的Devices工具栏上我们可以看到有一个Screen Capture功能按钮，我们可以获取当前屏幕信息。使用高效的ddms截图方式，于是就需要使用ddmlib.jar这个库。ddmlib.jar包在 tools/lib文件夹下

```java
public class ScreenShot {

    public IDevice device ;

    /**
     * 构造函数，默认获取第一个设备
     */
    public ScreenShot(){
        AndroidDebugBridge.init(false);
        device = this.getDevice(0);
    }

    /**
     * 构造函数，指定设备序号
     * @param deviceIndex 设备序号
     */
    public ScreenShot(int deviceIndex){
        AndroidDebugBridge.init(false); //
        device = this.getDevice(deviceIndex);
    }

    /**
     * 直接抓取屏幕数据
     * @return 屏幕数据
     */
    public RawImage getScreenShot(){
        RawImage rawScreen = null;
        if(device!=null){
            try {
                rawScreen = device.getScreenshot();
            } catch (TimeoutException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } catch (AdbCommandRejectedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }else{
            System.err.print("没有找到设备");
        }
        return rawScreen;
    }


    /**
     * 获取图片byte[]数据
     * @return 图片byte[]数据
     */
    public byte[] getScreenShotByteData(){
        RawImage rawScreen = getScreenShot();
        if(rawScreen != null){
            return rawScreen.data;
        }
        return null;
    }


    /**
     * 抓取图片并保存到指定路径
     * @param path 文件路径
     * @param fileName 文件名
     */
    public void getScreenShot(String path,String fileName){
        RawImage rawScreen = getScreenShot();
        if(rawScreen!=null){
            Boolean landscape = false;
            int width2 = landscape ? rawScreen.height : rawScreen.width;
            int height2 = landscape ? rawScreen.width : rawScreen.height;
            BufferedImage image = new BufferedImage(width2, height2,
                    BufferedImage.TYPE_INT_RGB);
            if (image.getHeight() != height2 || image.getWidth() != width2) {
                image = new BufferedImage(width2, height2,
                        BufferedImage.TYPE_INT_RGB);
            }
            int index = 0;
            int indexInc = rawScreen.bpp >> 3;
            for (int y = 0; y < rawScreen.height; y++) {
                for (int x = 0; x < rawScreen.width; x++, index += indexInc) {
                    int value = rawScreen.getARGB(index);
                    if (landscape)
                        image.setRGB(y, rawScreen.width - x - 1, value);
                    else
                        image.setRGB(x, y, value);
                }
            }
            try {
                ImageIO.write((RenderedImage) image, "PNG", new File(path + "/" + fileName + ".png"));
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

    /**
     * 获取得到device对象
     * @param index 设备序号
     * @return 指定设备device对象
     */
    private IDevice getDevice(int index) {
        IDevice device = null;
        AndroidDebugBridge bridge = AndroidDebugBridge
                .createBridge("adb", true);// 如果代码有问题请查看API，修改此处的参数值试一下
        waitDevicesList(bridge);
        IDevice devices[] = bridge.getDevices();

        for (int i = 0; i < devices.length; i++) {
            System.out.println(devices[i].toString());
        }

        if(devices.length < index){
            //没有检测到第index个设备
            System.err.print("没有检测到第" + index + "个设备");
        }
        else
        {
            if (devices.length-1>=index) {
                device = devices[index];
            }
            else
            {
                device = devices[0];
            }
        }
        return device;
    }

    /**
     * 等待查找device
     * @param bridge
     */
    private void waitDevicesList(AndroidDebugBridge bridge) {
        int count = 0;
        while (bridge.hasInitialDeviceList() == false) {
            try {
                Thread.sleep(500);
                count++;
            } catch (InterruptedException e) {
            }
            if (count > 60) {
                System.err.print("等待获取设备超时");
                break;
            }
        }
    }
}
```
使用的话很简单（注意这是计算机上的截取Android屏幕的工具，并不是Android上截屏的工具），创建一个Java项目就行了
```java
public class PCScreenShot {
    public static void main(String[] args) {
        ScreenShot screenShot = new ScreenShot();     //支持多个手机端设备管理。
        screenShot.getScreenShot("e://wts//", "screencapture_"+System.currentTimeMillis());
    }
}
```
#### 2、在PC端使用ADB进行截屏
>adb shell /system/bin/screencap -p /sdcard/screenshot.png（保存到SDCard）
adb pull /sdcard/screenshot.png d:/screenshot.png（保存到电脑）

备注：以上两种截屏方式不需要ROOT权限。原因是：ddms 通过 adb 发送信号给设备上的 adbd 守护进程，adbd 里面的 framebuffer service  负责整个截屏过程。所以，实际上是 adbd 守护进程启动了 screencap。adbd 是以 shell 用户执行的， 而系统为 shell 用户分配 graphics 组，所以 shell 用户是有权限调用 surfaceflinger 的接口的。
### 参考资源

 1. [Android5.0之后的截屏方案][1]
 2. [ANDROID新姿势：截屏代码整理][2]
 3. [Android中使用代码截图的各种方法总结][3]
  [1]: http://www.cnblogs.com/tgyf/p/4851092.html
  [2]: http://www.cnphp6.com/archives/61717
  [3]: http://blog.csdn.net/woshinia/article/details/11520403
