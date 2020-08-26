---
title: Android 10录制系统音及Demo
tags: [Android,Audio,AudioPlaybackCapture]
grammar_cjkRuby: true
categories: [Android]
date: 2020-08-20
---

### Demo获取

http://xhrong.github.io/attachments/AudioPlaybackDemo.7z

### 捕获播放的音频

Android 10 已引入 AudioPlaybackCapture API。应用可以借助此 API 复制其他应用正在播放的音频。此功能类似于屏幕采集，但采集对象是音频。主要用例是视频在线播放应用，这些应用希望捕获游戏正在播放的音频。

*请注意，对于其音频正在被捕获的应用，Capture API 不会影响该应用的延迟时间。*

#### 构建捕获应用

**先决条件**
为确保安全性和隐私，“捕获播放的音频”功能会施加一些限制。为了能够捕获音频，应用必须满足以下要求：

 - 应用必须具有 RECORD_AUDIO 权限。
 - 应用必须调出 MediaProjectionManager.createScreenCaptureIntent()显示的提示，用户必须批准该提示。
 - 捕获和播放音频的应用必须使用同一份用户个人资料。

**捕获音频**
如要从其他应用中捕获音频，您的应用必须构建 AudioRecord 对象，并向其添加 AudioPlaybackCaptureConfiguration。请按以下步骤操作：

 - 调用 AudioPlaybackCaptureConfiguration.Builder.build() 以构建 AudioPlaybackCaptureConfiguration。
 - 通过调用 setAudioPlaybackCaptureConfig 将配置传递到 AudioRecord。
 
 
 **按音频内容限制捕获**

 应用可以使用以下方法限制其可以捕获的音频：

 - 将 AUDIO_USAGE 传递到 AudioPlaybackCaptureConfiguration.addMatchingUsage() 以允许捕获特定用法。多次调用该方法可指定多个用法。
 - 将 AUDIO_USAGE 传递到 AudioPlaybackCaptureConfiguration.excludeUsage() 以禁止捕获该用法。多次调用该方法可指定多个用法。
 - 将 UID 传递到 AudioPlaybackCaptureConfiguration.addMatchingUid() 以仅捕获拥有特定 UID 的应用。多次调用该方法可指定多个 UID。
 - 将 UID 传递到 AudioPlaybackCaptureConfiguration.excludeUid() 以禁止捕获该 UID。多次调用该方法可指定多个 UID。

*请注意，您不能同时使用 addMatchingUsage() 和 excludeUsage() 方法。您必须选用其中一个。同样地，您也不能同时使用 addMatchingUid() 和 excludeUid()。*

#### 允许捕获播放的音频

您可以配置应用以防止其他应用捕获其音频。只有当应用满足以下要求时，才可以捕获来自该应用的音频：

**用法**
生成音频的播放器必须将其用法设置为 USAGE_MEDIA、USAGE_GAME或 USAGE_UNKNOWN。

**捕获政策**
播放器的捕获政策必须为 AudioAttributes.ALLOW_CAPTURE_BY_ALL，此政策允许其他应用捕获播放的音频。该操作可以通过许多方法完成：

 - 如要在所有播放器上启用捕获功能，请在应用的 manifest.xml 文件中包含 android:allowAudioPlaybackCapture="true"。
 - 您还可以通过调用 AudioManager.setAllowedCapturePolicy(AudioAttributes.ALLOW_CAPTURE_BY_ALL) 在所有播放器上启用捕获功能。
 - 使用 AudioAttributes.Builder.setAllowedCapturePolicy(AudioAttributes.ALLOW_CAPTURE_BY_ALL) 构建政策时，您可以针对单个播放器设置该政策。（如果您使用 AAudio，则调用 AAudioStreamBuilder_setAllowedCapturePolicy(AAUDIO_ALLOW_CAPTURE_BY_ALL)。）
 
 如果满足这些前提条件，则应用可以捕获播放器生成的任何音频。

注意：应用的音频能否被捕获也取决于应用的 targetSdkVersion。
 - 默认情况下，适配 Android 9.0 及之前版本的应用不允许捕获播放的音频。如要启用该功能，请在应用的 manifest.xml 文件中包含 android:allowAudioPlaybackCapture="true"。
 - 默认情况下，适配 Android 10 (API 级别 29) 或更高版本的应用允许其他应用捕获其音频。如要停用“捕获播放的音频”功能，请在应用的 manifest.xml 文件中包含 android:allowAudioPlaybackCapture="false"。
 
 
**停用系统捕获**
上述允许捕获的保护措施仅适用于应用。默认情况下，Android 系统组件可以捕获播放的音频。其中许多组件由 Android 供应商自定义，并支持无障碍和字幕等功能。因此，我们建议应用允许系统捕获其播放的音频。在极少数情况下，如果您不希望系统捕获应用播放的音频，则可以将该捕获政策设置为 ALLOW_CAPTURE_BY_NONE。

**在运行时设置政策**
在应用运行时，您可以调用 AudioManager.setAllowedCapturePolicy() 来更改捕获政策。如果您调用该方法时，MediaPlayer 或 AudioTrack 正在播放音频，则相应音频不受影响。您必须关闭播放器或曲目，然后再重新打开，这样政策变更才会生效。

**政策 = 清单 + AudioManager + AudioAttributes**
由于可以在多个位置指定捕获政策，因此请务必要了解如何确定有效政策。您应始终应用最严格的捕获政策。例如，即使将AudioManager#setAllowedCapturePolicy 设置为 ALLOW_CAPTURE_BY_ALL，清单中包含 setAllowedCapturePolicy="false" 的应用也决不会允许非系统应用捕获其音频。同样，如果将 AudioManager#setAllowedCapturePolicy 设置为 ALLOW_CAPTURE_BY_ALL，将清单设置为 setAllowedCapturePolicy="true"，但媒体播放器的 AudioAttributes 采用 AudioAttributes.Builder#setAllowedCapturePolicy(ALLOW_CAPTURE_BY_SYSTEM) 构建而成，则非系统应用将无法捕获此媒体播放器播放的音频。

下表总结了清单属性和有效政策的效果：

| allowAudioPlaybackCapture | ALLOW_CAPTURE_BY_ALL | ALLOW_CAPTURE_BY_SYSTEM | ALLOW_CAPTURE_BY_NONE |
| ------------------------- | -------------------- | ----------------------- | --------------------- |
| true                      | 任何应用             | 仅系统                  | 无捕获                |
| false                     | 仅系统               | 仅系统                  |                       |


