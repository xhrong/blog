---
title: Android录屏与传输
tags: [ScreenRecord]
grammar_cjkRuby: true
categories: [Android]
date: 2019-10-05

---

### 基于MediaProjection+MediaCodec
（1）通过ImageReader创建Surface，获取图片后编码
存在的问题：只能获取RGBA格式的图片，不能直接进行编码，需要先转换【像素级数据转换在1080P下非常耗时，JAVA版和C语言版均要几十毫秒才行，基本不可行】
好处：可以逐张保存图片为Bitmap，实现截屏功能；可以自由的丢弃图片，控制提供给MediaCodec的图片速率，即帧率

（2）通过MediaCodec创建Surface直接编码
存在的问题：无法控制MediaCodec的输入输出帧率，动态画面时往往帧较高，不适合传输

（3）通过ImageReader创建Surface获取图片Bitmap，提供给MediaCodec创建的Surface进行编码
该方案综合了（1）、（2）两个方案的优点。建议采用。

（4）通过Jni获取数据后在C层编码
该方案是前3种方案的C层化，本质是一样的


```java
package com.encode.androidencode;

import android.annotation.SuppressLint;
import android.media.MediaCodec;
import android.media.MediaCodecInfo;
import android.media.MediaFormat;
import android.os.Build;
import android.os.Bundle;
import android.util.Log;
import android.view.Surface;

import com.interfaces.androidencode.ScreenCastService;

import java.io.IOException;
import java.nio.ByteBuffer;

public class ScreenAvcEncoder {


    private MediaCodec mediaCodec;
    int m_width;
    int m_height;
    byte[] m_info = null;

    private int mColorFormat;
    private MediaCodecInfo codecInfo;
    private static final String MIME_TYPE = "video/avc"; // H.264 Advanced Video
    private byte[] yuv420 = null;
    private byte[] nv21 = null;

    MediaFormat mediaFormat;

    private Surface surface;

    @SuppressLint("NewApi")
    public ScreenAvcEncoder(int width, int height, int framerate, int bitrate) {
        m_width = width;
        m_height = height;
        Log.v("xmc", "ScreenAvcEncoder:" + m_width + "+" + m_height);
        yuv420 = new byte[width * height * 3 / 2];
        nv21 = new byte[width * height * 3 / 2];
        try {
            mediaCodec = MediaCodec.createEncoderByType("video/avc");
        } catch (IOException e) {
            e.printStackTrace();
        }
        MediaFormat mediaFormat = MediaFormat.createVideoFormat("video/avc", width, height);
        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, bitrate);
        mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, framerate);
        if(ScreenCastService.captureMode== ScreenCastService.CaptureMode.useImageReader) {
            mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420SemiPlanar);
        }else {
            mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface);
        }
        mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, 1);//关键帧间隔时间 单位s
     //   mediaFormat.setInteger(MediaFormat.KEY_BITRATE_MODE, MediaCodecInfo.EncoderCapabilities.BITRATE_MODE_CBR);

        mediaCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);

        if(ScreenCastService.captureMode== ScreenCastService.CaptureMode.useSurface ||
                ScreenCastService.captureMode== ScreenCastService.CaptureMode.useImageReaderWithSurface) {
            this.surface = mediaCodec.createInputSurface();
        }
        mediaCodec.start();

    }

    public Surface getSurface(){
        return this.surface;
    }


    @SuppressLint("NewApi")
    public void close() {
        try {
            mediaCodec.stop();
            mediaCodec.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    long timestamp;
    private void updateFrame(){
        if (Build.VERSION.SDK_INT >= 23) {
            if (System.currentTimeMillis()-timestamp>=1000) {
                timestamp=System.currentTimeMillis();
                Bundle params = new Bundle();
                params.putInt(MediaCodec.PARAMETER_KEY_REQUEST_SYNC_FRAME, 0);
                mediaCodec.setParameters(params);
            }
        }
    }


    public int getOutputBuffer(byte[] output){
        Log.v("xmc", "getOutputBuffer" );
    //    updateFrame();
        int pos = 0;
        MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
        int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);

        while (outputBufferIndex >= 0) {
            ByteBuffer outputBuffer = mediaCodec.getOutputBuffer(outputBufferIndex);
            byte[] outData = new byte[bufferInfo.size];
            outputBuffer.get(outData);

            if (m_info != null) {
                System.arraycopy(outData, 0, output, pos, outData.length);
                pos += outData.length;

            } else {//保存pps sps 只有开始时 第一个帧里有， 保存起来后面用
                ByteBuffer spsPpsBuffer = ByteBuffer.wrap(outData);

                if (spsPpsBuffer.getInt() == 0x00000001) {
                    m_info = new byte[outData.length];
                    System.arraycopy(outData, 0, m_info, 0, outData.length);
                } else {
                    return -1;
                }
            }
            if ((output[4] & 0x1F) == 5) {//key frame 编码器生成关键帧时只有 00 00 00 01 65 没有pps sps， 要加上
                System.arraycopy(m_info, 0, output, 0, m_info.length);
                System.arraycopy(outData, 0, output, m_info.length, outData.length);
                pos += m_info.length;
            }
            mediaCodec.releaseOutputBuffer(outputBufferIndex, false);
            outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
        }


        Log.v("xmc", "getOutputBuffer+pos:" + pos);
        return pos;
    }

    /**
     * 硬编码器，需要喂NV12的数据，如果不是的话，需要提前转换。
     * @param input
     * @param output
     * @return
     */
    @SuppressLint("NewApi")
    public int offerEncoder(byte[] input, byte[] output) {
   //     Log.v("xmc", "offerEncoder:input & output" + input.length + "+" + output.length);
        int pos = 0;

    //    Log.e("xmc", "RGBAtoNV12 start");

        //下面4个方法功能是一样的，适用于基于RGBA_8888的图片流编码。（RGBA_8888实现的数值顺序是BGRA）
        // 分别采用JAVA实现、纯C实现、FFMPEG实现、LibYUV实现。1080P测试效果：
        // 前3者性能相当，都比较差。大约80ms或以上可转一帧，不实用
        //LibYUV实现性能相对较好。大约20-40ms可转一帧，基本实用。但仍然不够理想

        // rgb2YCbCr420(input, nv21, m_width, m_height);
        // ColorHelper.rgb2YCbCr420(input,nv21,m_width,m_height);
        // ColorHelper.ffmpegRGB2YUV(input,nv21,m_width,m_height);
        ColorHelper.BGRA2YUV(input, nv21, m_width, m_height);

     //   Log.e("xmc", "RGBAtoNV12 end");


        //这个转换适用于相机。大约10ms可转一帧，已经可实用了。
        //NV21ToNV12(nv21,yuv420,m_width,m_height);


    //    Log.e("xmc", "offerEncoder:input & yuv420" + input.length + "+" + yuv420.length);
        try {
            int inputBufferIndex = mediaCodec.dequeueInputBuffer(-1);
            if (inputBufferIndex >= 0) {
                ByteBuffer inputBuffer = mediaCodec.getInputBuffer(inputBufferIndex);
                inputBuffer.clear();
                inputBuffer.put(nv21);
                mediaCodec.queueInputBuffer(inputBufferIndex, 0, nv21.length, 0, 0);

            }

            MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
            int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);

            while (outputBufferIndex >= 0) {
                ByteBuffer outputBuffer = mediaCodec.getOutputBuffer(outputBufferIndex);
                byte[] outData = new byte[bufferInfo.size];
                outputBuffer.get(outData);

                if (m_info != null) {
                    System.arraycopy(outData, 0, output, pos, outData.length);
                    pos += outData.length;

                } else {//保存pps sps 只有开始时 第一个帧里有， 保存起来后面用
                    ByteBuffer spsPpsBuffer = ByteBuffer.wrap(outData);

                    if (spsPpsBuffer.getInt() == 0x00000001) {
                        m_info = new byte[outData.length];
                        System.arraycopy(outData, 0, m_info, 0, outData.length);
                    } else {
                        return -1;
                    }
                }
                if ((output[4] & 0x1F) == 5) {//key frame 编码器生成关键帧时只有 00 00 00 01 65 没有pps sps， 要加上
                    System.arraycopy(m_info, 0, output, 0, m_info.length);
                    System.arraycopy(outData, 0, output, m_info.length, outData.length);
                    pos += m_info.length;
                }
                mediaCodec.releaseOutputBuffer(outputBufferIndex, false);
                outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
            }

        } catch (Throwable t) {
            t.printStackTrace();
        }
    //    Log.v("xmc", "offerEncoder+pos:" + pos);
        return pos;
    }


    private void NV21ToNV12(byte[] nv21, byte[] nv12, int width, int height) {
        if (nv21 == null || nv12 == null) return;
        int framesize = width * height;
        int i = 0, j = 0;
        System.arraycopy(nv21, 0, nv12, 0, framesize);
        for (j = 0; j < framesize / 2; j += 2) {
            nv12[framesize + j] = nv21[j + framesize + 1];
            nv12[framesize + j + 1] = nv21[j + framesize];
        }
    }


    //如果swapYV12toI420方法颜色不对可以试下这个方法，不同机型有不同的转码方式
    private void NV21toI420SemiPlanar(byte[] nv21bytes, byte[] i420bytes, int width, int height) {
        Log.v("xmc", "NV21toI420SemiPlanar:::" + width + "+" + height);
        final int iSize = width * height;
        System.arraycopy(nv21bytes, 0, i420bytes, 0, iSize);

        for (int iIndex = 0; iIndex < iSize / 2; iIndex += 2) {
            i420bytes[iSize + iIndex / 2 + iSize / 4] = nv21bytes[iSize + iIndex]; // U
            i420bytes[iSize + iIndex / 2] = nv21bytes[iSize + iIndex + 1]; // V
        }
    }

    //yv12 转 yuv420p  yvu -> yuv
    private void swapYV12toI420(byte[] yv12bytes, byte[] i420bytes, int width, int height) {
        Log.v("xmc", "swapYV12toI420:::" + width + "+" + height);
        Log.v("xmc", "swapYV12toI420:::" + yv12bytes.length + "+" + i420bytes.length + "+" + width * height);
        System.arraycopy(yv12bytes, 0, i420bytes, 0, width * height);
        System.arraycopy(yv12bytes, width * height + width * height / 4, i420bytes, width * height, width * height / 4);
        System.arraycopy(yv12bytes, width * height, i420bytes, width * height + width * height / 4, width * height / 4);
    }


    /**
     * RGB图片转YUV420数据
     * 宽、高不能为奇数
     * 这个方法太耗时了，需要优化。用C实现（C实现的结果并没有JAVA的性能好。奇怪。）
     *
     * @param width  宽
     * @param height 高
     * @return
     */
    public void rgb2YCbCr420(byte[] byteArgb, byte[] yuv, int width, int height) {
        Log.e("xmc", "encodeYUV420SP start");
        int len = width * height;
        int r, g, b, y, u, v, c;
        for (int i = 0; i < height; i++) {
            for (int j = 0; j < width; j++) {
                c = (i * width + j) * 4;
                r = byteArgb[c] & 0xFF;
                g = byteArgb[c + 1] & 0xFF;
                b = byteArgb[c + 2] & 0xFF;
                //   int a = byteArgb[(i * width + j) * 4 + 3]&0xFF;
                //套用公式
                y = ((66 * r + 129 * g + 25 * b + 128) >> 8) + 16;
                u = ((-38 * r - 74 * g + 112 * b + 128) >> 8) + 128;
                v = ((112 * r - 94 * g - 18 * b + 128) >> 8) + 128;
                //调整
                y = y < 16 ? 16 : (y > 255 ? 255 : y);
                u = u < 0 ? 0 : (u > 255 ? 255 : u);
                v = v < 0 ? 0 : (v > 255 ? 255 : v);
                //赋值
                yuv[i * width + j] = (byte) y;
                yuv[len + (i >> 1) * width + (j & ~1) + 0] = (byte) u;
                yuv[len + +(i >> 1) * width + (j & ~1) + 1] = (byte) v;
            }
        }
        Log.e("xmc", "encodeYUV420SP end");
    }
}

```
```java
package com.interfaces.androidencode;

import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.content.res.Configuration;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.ImageFormat;
import android.graphics.Paint;
import android.graphics.PixelFormat;
import android.graphics.YuvImage;
import android.hardware.display.DisplayManager;
import android.hardware.display.VirtualDisplay;
import android.media.Image;
import android.media.ImageReader;
import android.media.projection.MediaProjection;
import android.media.projection.MediaProjectionManager;
import android.os.Environment;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.os.Messenger;
import android.util.Log;
import android.util.Size;
import android.view.Surface;

import com.encode.androidencode.AvcEncoder;
import com.encode.androidencode.ScreenAvcEncoder;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.concurrent.atomic.AtomicBoolean;

/**
 * Created by xhrong on 2019/9/16.
 */

public class ScreenCastService extends Service {

    private final String TAG = "ScreenCastService";

    public enum CaptureMode {
        useImageReader,
        useImageReaderWithSurface,
        useSurface
    }

    public static CaptureMode captureMode = CaptureMode.useImageReaderWithSurface;


    private Handler handler;
    private Messenger crossProcessMessenger;

    private Object lock = new Object();
    private AtomicBoolean isCasting = new AtomicBoolean(false);
    private byte[] bytes;

    private MediaProjectionManager mediaProjectionManager;
    private MediaProjection mediaProjection;
    private Surface inputSurface;
    private VirtualDisplay virtualDisplay;
    private ImageReader imageReader;
    private Surface encodeSurface;
    private Paint paint = new Paint();

    private int screenWidth = 1920;
    private int screenHeight = 1080;
    //向引擎喂的帧率
    private int fps = 15;
    private int bitrate = screenHeight * screenWidth * 3;//编码比特率，
    private byte[] h264 = new byte[screenWidth * screenHeight * 3];

    private int DATA_LENGTH = 1480;

    private String remoteIp = "";
    private String localIp = "";

    private RtpDataSender rtpDataSender;
    private ScreenAvcEncoder avcCodec;

    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind");

        handler = new Handler(new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                Log.i(TAG, "Handler got message. what:" + msg.what);
                switch (msg.what) {
                    case RealTimeScreenActivity.CONNECTED:
                    case RealTimeScreenActivity.DISCONNECTED:
                        break;
                    case RealTimeScreenActivity.STOP:
                        stopScreenCapture();
                        stopSelf();
                        break;
                }
                return false;
            }
        });
        crossProcessMessenger = new Messenger(handler);
        return crossProcessMessenger.getBinder();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e(TAG, "onCreate");
        localIp = DeviceHelper.getIpAddress();
        Size size = DeviceHelper.getScreenSize();
        // screenWidth = size.getWidth();
        // screenHeight = size.getHeight();
    }

    @Override
    public void onStart(Intent intent, int startId) {
        Log.d(TAG, "onStart");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");

        stopScreenCapture();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        if (intent == null) {
            return START_NOT_STICKY;
        }

        remoteIp = intent.getStringExtra(RealTimeScreenActivity.REMOTE_IP_PARAM);
        final int resultCode = intent.getIntExtra(RealTimeScreenActivity.STATE_RESULT_CODE, -1);
        final Intent resultData = intent.getParcelableExtra(RealTimeScreenActivity.STATE_RESULT_DATA);

        Log.i(TAG, "Start casting with screen:" + screenWidth + "x" + screenHeight);


        avcCodec = new ScreenAvcEncoder(screenWidth, screenHeight, fps, bitrate);
        rtpDataSender = new RtpDataSender();
        mediaProjectionManager = (MediaProjectionManager) getSystemService(Context.MEDIA_PROJECTION_SERVICE);


        startScreenCapture(resultCode, resultData, screenWidth, screenHeight, 1);


        return START_STICKY;
    }

    private Bitmap bmp = Bitmap.createBitmap(screenWidth, screenHeight, Bitmap.Config.ARGB_8888);
    private int i = 0;

    private void startScreenCapture(int resultCode, Intent resultData, final int width, final int height, int dpi) {
        this.mediaProjection = mediaProjectionManager.getMediaProjection(resultCode, resultData);
        Log.d(TAG, "startRecording...");

        try {
            if (captureMode == CaptureMode.useImageReader) {
                imageReader = ImageReader.newInstance(width, height, PixelFormat.RGBA_8888, 5);
                imageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
                    @Override
                    public void onImageAvailable(ImageReader reader) {
                        Image image = reader.acquireLatestImage();
                        if (image != null) {
                            Image.Plane[] planes = image.getPlanes();
                            //因为我们要求的是RGBA格式的数据，所以全部的存储在planes[0]中
                            Image.Plane plane = planes[0];
                            //由于Image中的缓冲区存在数据对齐，所以其大小不一定是我们生成ImageReader实例时指定的大小，
                            //ImageReader会自动为画面每一行最右侧添加一个padding，以进行对齐，对齐多少字节可能因硬件而异，
                            //所以我们在取出数据时需要忽略这一部分数据。
                            ByteBuffer buffer = plane.getBuffer();
                            synchronized (lock) {
                                bytes = new byte[buffer.remaining()];
                                buffer.get(bytes, 0, bytes.length);//将数据读到数组中
                            }
                            image.close();
                        }
                    }
                }, null);
                this.inputSurface = imageReader.getSurface();
                this.encodeSurface = this.avcCodec.getSurface();
            } else if (captureMode == CaptureMode.useImageReaderWithSurface) {
                imageReader = ImageReader.newInstance(width, height, PixelFormat.RGBA_8888, 1);
                imageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
                    @Override
                    public void onImageAvailable(ImageReader reader) {
                        Image image = reader.acquireLatestImage();
                        if (image != null) {
                            Image.Plane[] planes = image.getPlanes();

                            if (planes.length > 0) {
                                final ByteBuffer buffer = planes[0].getBuffer();
                                int pixelStride = planes[0].getPixelStride();
                                int rowStride = planes[0].getRowStride();
                                int rowPadding = rowStride - pixelStride * screenWidth;
                                synchronized (lock) {
                                    bmp.copyPixelsFromBuffer(buffer);
                                }
                            }
                            image.close();
                        }
                    }
                }, null);
                this.inputSurface = imageReader.getSurface();
                this.encodeSurface = this.avcCodec.getSurface();
            } else if (captureMode == CaptureMode.useSurface) {
                this.inputSurface = avcCodec.getSurface();
            }
            this.virtualDisplay = this.mediaProjection.createVirtualDisplay(
                    "Recording Display",
                    width, height, dpi,
                    DisplayManager.VIRTUAL_DISPLAY_FLAG_PUBLIC,
                    this.inputSurface,
                    null,
                    null);

            isCasting.set(true);
            //启动一个独立线程发送数据，以便控制发送频率
            sendData();
        } catch (Exception e) {
            Log.e(TAG, "Failed to initial encoder, e: " + e);
            releaseCapture();
        }
    }

    /**
     * 返回当前屏幕是否为竖屏。
     *
     * @return 当且仅当当前屏幕为竖屏时返回true, 否则返回false。
     */
    public boolean isScreenOriatationPortrait() {
        return getResources().getConfiguration().orientation == Configuration.ORIENTATION_PORTRAIT;
    }

    public void savePicture(Bitmap bm, String fileName) {
        Log.i("xing", "savePicture: ------------------------");
        if (null == bm) {
            Log.i("xing", "savePicture: ------------------图片为空------");
            return;
        }
        //建立指定文件夹
        File foder = new File(Environment.getExternalStorageDirectory(), "zzp_sale");
        if (!foder.exists()) {
            foder.mkdirs();
        }
        File myCaptureFile = new File(foder, fileName);
        try {
            if (!myCaptureFile.exists()) {
                myCaptureFile.createNewFile();
            }
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(myCaptureFile));
            //压缩保存到本地
            bm.compress(Bitmap.CompressFormat.JPEG, 90, bos);
            bos.flush();
            bos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    long time;

    //启动一个独立线程发送数据，以便控制发送频率
    private void sendData() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (isCasting.get()) {
                    if (captureMode == CaptureMode.useImageReader) {
                        doSendDataImageReader();
                    } else if (captureMode == CaptureMode.useSurface) {
                        doSendDataEncodeSurface();
                    } else if (captureMode == CaptureMode.useImageReaderWithSurface) {
                        doSendDataEncodeImageReaderWithSurface();
                    }
                }
            }
        }).start();
    }

    byte[][] sendData = null;
    boolean[] marks = null;

    private void doSendDataEncodeSurface() {
        int ret = avcCodec.getOutputBuffer(h264);

        if (ret > 0) {
            //实时发送数据流
            Log.v("xmc", "ret =" + ret);
            byte[] h264Data = new byte[ret];
            System.arraycopy(h264, 0, h264Data, 0, ret);
            int dataLength = (h264Data.length - 1) / DATA_LENGTH + 1;
            sendData = new byte[dataLength][];
            marks = new boolean[dataLength];
            marks[marks.length - 1] = true;
            int x = 0, y = 0;
            int length = h264Data.length;
            for (int i = 0; i < length; i++) {
                if (y == 0) {
                    sendData[x] = new byte[length - i > DATA_LENGTH ? DATA_LENGTH : length - i];
                }
                sendData[x][y] = h264Data[i];
                y++;
                if (y == sendData[x].length) {
                    y = 0;
                    x++;
                }
            }

            time = System.currentTimeMillis();

            rtpDataSender.sendData(sendData, marks);
        } else {
            try {
                Thread.sleep(10);

//                if (System.currentTimeMillis() - time < 1000 / fps) {
//                    return;
//                } else {
//                    time = System.currentTimeMillis();
//                }
//
//                if (sendData != null) {
//                    rtpDataSender.sendData(sendData, marks);
//                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void doSendDataEncodeImageReaderWithSurface() {
        //这里控制帧率
        if (System.currentTimeMillis() - time < 1000 / fps) {
            return;
        } else {
            time = System.currentTimeMillis();
        }
        synchronized (lock) {
            if (encodeSurface != null && bmp != null) {
                Canvas canvas = encodeSurface.lockCanvas(null);
                canvas.drawColor(Color.BLACK);
                canvas.drawBitmap(bmp, 0f, 0f, paint);
                encodeSurface.unlockCanvasAndPost(canvas);
            }
        }
        int ret = avcCodec.getOutputBuffer(h264);

        if (ret > 0) {
            //实时发送数据流
            Log.v("xmc", "ret =" + ret);
            byte[] h264Data = new byte[ret];
            System.arraycopy(h264, 0, h264Data, 0, ret);
            int dataLength = (h264Data.length - 1) / DATA_LENGTH + 1;
            sendData = new byte[dataLength][];
            marks = new boolean[dataLength];
            marks[marks.length - 1] = true;
            int x = 0, y = 0;
            int length = h264Data.length;
            for (int i = 0; i < length; i++) {
                if (y == 0) {
                    sendData[x] = new byte[length - i > DATA_LENGTH ? DATA_LENGTH : length - i];
                }
                sendData[x][y] = h264Data[i];
                y++;
                if (y == sendData[x].length) {
                    y = 0;
                    x++;
                }
            }

            time = System.currentTimeMillis();

            rtpDataSender.sendData(sendData, marks);
        } else {
            try {
                Thread.sleep(10);

//                if (System.currentTimeMillis() - time < 1000 / fps) {
//                    return;
//                } else {
//                    time = System.currentTimeMillis();
//                }
//
//                if (sendData != null) {
//                    rtpDataSender.sendData(sendData, marks);
//                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }


    private void doSendDataImageReader() {
        synchronized (lock) {
            if (bytes != null && bytes.length > 0) {
                //这里控制帧率
                if (System.currentTimeMillis() - time < 1000 / fps) {
                    return;
                } else {
                    time = System.currentTimeMillis();
                }
                int ret = avcCodec.offerEncoder(bytes, h264);

                if (ret > 0) {
                    //实时发送数据流
                    Log.v("xmc", "ret =" + ret);
                    byte[] h264Data = new byte[ret];
                    System.arraycopy(h264, 0, h264Data, 0, ret);
                    int dataLength = (h264Data.length - 1) / DATA_LENGTH + 1;
                    final byte[][] sendData = new byte[dataLength][];
                    final boolean[] marks = new boolean[dataLength];
                    marks[marks.length - 1] = true;
                    int x = 0, y = 0;
                    int length = h264Data.length;
                    for (int i = 0; i < length; i++) {
                        if (y == 0) {
                            sendData[x] = new byte[length - i > DATA_LENGTH ? DATA_LENGTH : length - i];
                        }
                        sendData[x][y] = h264Data[i];
                        y++;
                        if (y == sendData[x].length) {
                            y = 0;
                            x++;
                        }
                    }

                    rtpDataSender.sendData(sendData, marks);
                }
            }
        }
    }

    private void stopScreenCapture() {
        isCasting.set(false);
        releaseCapture();
        if (virtualDisplay == null) {
            return;
        }
        virtualDisplay.release();
        virtualDisplay = null;

        avcCodec.close();
    }

    private void releaseCapture() {
        if (inputSurface != null) {
            inputSurface.release();
            inputSurface = null;
        }
        if (mediaProjection != null) {
            mediaProjection.stop();
            mediaProjection = null;
        }
    }
}

```

```java
package com.encode.androidencode;

public class ColorHelper {

    static {
        System.loadLibrary("native-lib");
    }

    public static native void rgb2YCbCr420(byte[] byteArgb, byte[] yuv, int width, int height);
    public static native void ffmpegRGB2YUV(byte[] byteArgb, byte[] yuv, int width, int height);
    public static native void BGRA2YUV(byte[] byteArgb, byte[] yuv, int width, int height);
}
```


```c++
//
// Created by H on 2019/10/3.
//

#include <zconf.h>
#include "com_encode_androidencode_ColorHelper.h"

extern "C" {
#include "include/libavutil/frame.h"
#include "include/libavcodec/avcodec.h"
#include "include/libswscale/swscale.h"
#include "include/libyuv.h"
}
/////=============Start 纯C版本========
//这个方法的性能并不比JAVA版本的好。
JNIEXPORT void JNICALL Java_com_encode_androidencode_ColorHelper_rgb2YCbCr420
        (JNIEnv *env, jclass clazz, jbyteArray jinput, jbyteArray joutput, jint jwidth,
         jint jheight) {

    int width = (int) jwidth;
    int height = (int) jheight;

    jbyte *input = env->GetByteArrayElements(jinput, 0);

    jbyte *output = env->GetByteArrayElements(joutput, 0);

    int len = width * height;
    int r, g, b, y, u, v, c;
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            c = (i * width + j) * 4;
            r = input[c] & 0xFF;
            g = input[c + 1] & 0xFF;
            b = input[c + 2] & 0xFF;
            //     int a = byteArgb[(i * width + j) * 4 + 3]&0xFF;
            //套用公式
            y = ((66 * r + 129 * g + 25 * b + 128) >> 8) + 16;
            u = ((-38 * r - 74 * g + 112 * b + 128) >> 8) + 128;
            v = ((112 * r - 94 * g - 18 * b + 128) >> 8) + 128;
            //调整
            y = y < 16 ? 16 : (y > 255 ? 255 : y);
            u = u < 0 ? 0 : (u > 255 ? 255 : u);
            v = v < 0 ? 0 : (v > 255 ? 255 : v);
            //赋值
            output[i * width + j] = (Byte) y;
            output[len + (i >> 1) * width + (j & ~1) + 0] = (Byte) u;
            output[len + +(i >> 1) * width + (j & ~1) + 1] = (Byte) v;
        }
    }

    env->ReleaseByteArrayElements(jinput, input, 0);
    env->ReleaseByteArrayElements(joutput, output, 0);
}

/////=============End 纯C版本========

/////=========Start FFMPEG版本===============
//这个方法的性能并不比JAVA版本的好。
JNIEXPORT void JNICALL Java_com_encode_androidencode_ColorHelper_ffmpegRGB2YUV
        (JNIEnv *env, jclass clazz, jbyteArray jinput, jbyteArray joutput, jint jwidth,
         jint jheight) {

//uint8_t *buf_argb, uint8_t *buf_420p, int w, int h)
    int w = (int) jwidth;
    int h = (int) jheight;
    jbyte *input = env->GetByteArrayElements(jinput, 0);

    jbyte *output = env->GetByteArrayElements(joutput, 0);

    AVFrame *frmArgb = av_frame_alloc();
    AVFrame *frm420p = av_frame_alloc();



//绑定数据缓冲区
    avpicture_fill((AVPicture *) frmArgb, (uint8_t *) input, AV_PIX_FMT_BGRA, w, h);
    avpicture_fill((AVPicture *) frm420p, (uint8_t *) output, AV_PIX_FMT_YUV420P, w, h);

//指定原数据格式，分辨率及目标数据格式，分辨率
    struct SwsContext *sws = sws_getContext(w, h, AV_PIX_FMT_BGRA, w, h, AV_PIX_FMT_YUV420P,
                                            SWS_BILINEAR, NULL, NULL, NULL);

//转换
    int ret = sws_scale(sws, frmArgb->data, frmArgb->linesize, 0, h, frm420p->data,
                        frm420p->linesize);
    av_frame_free(&frmArgb);
    av_frame_free(&frm420p);
    sws_freeContext(sws);

    env->ReleaseByteArrayElements(jinput, input, 0);
    env->ReleaseByteArrayElements(joutput, output, 0);
}

/////=========End  FFMPEG版本===============


/////// ==============Start LibYUV版本================
int rgbaToI420(JNIEnv *env, jclass clazz, jbyteArray rgba, jint rgba_stride,
               jbyteArray yuv, jint y_stride, jint u_stride, jint v_stride,
               jint width, jint height,
               int (*func)(const uint8 *, int, uint8 *, int, uint8 *, int, uint8 *, int, int,
                           int)) {
    size_t ySize = (size_t) (y_stride * height);
    size_t uSize = (size_t) (u_stride * height >> 1);
    jbyte *rgbaData = env->GetByteArrayElements(rgba, JNI_FALSE);
    jbyte *yuvData = env->GetByteArrayElements(yuv, JNI_FALSE);
    int ret = func((const uint8 *) rgbaData, rgba_stride, (uint8 *) yuvData, y_stride,
                   (uint8 *) (yuvData) + ySize, u_stride, (uint8 *) (yuvData) + ySize + uSize,
                   v_stride, width, height);
    env->ReleaseByteArrayElements(rgba, rgbaData, JNI_OK);
    env->ReleaseByteArrayElements(yuv, yuvData, JNI_OK);
    return ret;
}


static int
(*rgbaToI420Func[])(const uint8 *, int, uint8 *, int, uint8 *, int, uint8 *, int, int, int) ={
        libyuv::ABGRToI420, libyuv::RGBAToI420, libyuv::ARGBToI420, libyuv::BGRAToI420,
        libyuv::RGB24ToI420, libyuv::RGB565ToI420
};


void i420ToNv12(jbyte *src_i420_data, jint width, jint height, jbyte *src_nv12_data) {
    jint src_y_size = width * height;
    jint src_u_size = (width >> 1) * (height >> 1);

    jbyte *src_nv12_y_data = src_nv12_data;
    jbyte *src_nv12_uv_data = src_nv12_data + src_y_size;

    jbyte *src_i420_y_data = src_i420_data;
    jbyte *src_i420_u_data = src_i420_data + src_y_size;
    jbyte *src_i420_v_data = src_i420_data + src_y_size + src_u_size;

    libyuv::I420ToNV12(
            (const uint8 *) src_i420_y_data, width,
            (const uint8 *) src_i420_u_data, width >> 1,
            (const uint8 *) src_i420_v_data, width >> 1,
            (uint8 *) src_nv12_y_data, width,
            (uint8 *) src_nv12_uv_data, width,
            width, height);
}

//libyuv没有提供RGBA to NV12的直接接口，需要转两次。但性能仍然优于JAVA版本、纯C版本和FFMpeg版本。
JNIEXPORT JNICALL void Java_com_encode_androidencode_ColorHelper_BGRA2YUV(
        JNIEnv *env, jclass clazz, jbyteArray jinput, jbyteArray joutput, jint jwidth,
        jint jheight) {

    int width = (int) jwidth;
    int height = (int) jheight;


//    //0-3 表示转换类型
//    //4-7 表示rgba_stride的宽度的倍数
//    //8-11 表示yuv_stride宽度移位数
//    //12-15 表示uv左移位数
//
//    public static final int RGBA_TO_I420=0x01001040;
//    public static final int ABGR_TO_I420=0x01001041;
//    public static final int BGRA_TO_I420=0x01001042;
//    public static final int ARGB_TO_I420=0x01001043;
//    public static final int RGB24_TO_I420=0x01001034;
//    public static final int RGB565_TO_I420=0x01001025;

 /////   https://blog.csdn.net/junzia/article/details/76315120


    uint8 cType = (uint8) (0x01001040 & 0x0F);
    int rgba_stride = ((0x01001040 & 0xF0) >> 4) * width;
    int y_stride = width;
    int u_stride = width >> 1;
    int v_stride = u_stride;

    //先是RGBA to I420
    jbyteArray  tmpDst =env->NewByteArray(env->GetArrayLength(joutput));
    rgbaToI420(env, clazz, jinput, rgba_stride, tmpDst, y_stride, u_stride, v_stride, width,
               height, rgbaToI420Func[cType]);

    jbyte *tDst = env->GetByteArrayElements(tmpDst, 0);
    jbyte *output = env->GetByteArrayElements(joutput, 0);

    //再I420 to  Nv12
    i420ToNv12(tDst,jwidth,jheight,output);
    env->ReleaseByteArrayElements(tmpDst, tDst, JNI_OK);
    env->ReleaseByteArrayElements(joutput, output, JNI_OK);
}


/////// ==============End LibYUV版本================
```

### 基于MediaRecoder
MediaRecoder可直接将屏幕录制成文件，录屏过程比较简单。但是无法逐帧获取视频数据流进行网络传输，所以只适合本地录制，不适合网络传输相关场景。 