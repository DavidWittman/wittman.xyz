---
title: "Samsung Galaxy S21 Camera Crashing After Update"
date: 2021-07-17T21:13:02-05:00
tags: [android]
---

A few weeks ago I received a system update for my Samsung Galaxy S21 which caused my Camera app to crash every time I opened it. I tried the normal debugging steps like checking permissions and clearing cache/app storage, but it still didn't work. Even more annoying was that I couldn't scan QR codes because the built-in reader just uses the Camera app.

Eventually, I hooked my phone up to my computer and took a look at the logs with `adb logcat *:W`. The `*:W` part limits the log level to Warning or higher to reduce some of the noise in the logs. After a few attempts starting the Camera app and sifting through a disturbing amount of logs, I stumbled on the following traceback:

```
07-15 12:12:34.667 14798 14798 E AndroidRuntime: FATAL EXCEPTION: main                                                  
07-15 12:12:34.667 14798 14798 E AndroidRuntime: Process: com.sec.android.app.camera, PID: 14798                        
07-15 12:12:34.667 14798 14798 E AndroidRuntime: java.lang.SecurityException: Failed to find provider com.samsung.android.app.screenrecorder.provider for user 0; expected to find a valid ContentProvider for this authority
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Parcel.createExceptionOrNull(Parcel.java:2385)    
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Parcel.createException(Parcel.java:2369)          
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Parcel.readException(Parcel.java:2352)            
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Parcel.readException(Parcel.java:2294)            
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.content.IContentService$Stub$Proxy.registerContentObserver(IContentService.java:1250)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.content.ContentResolver.registerContentObserver(ContentResolver.java:2637)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.content.ContentResolver.registerContentObserver(ContentResolver.java:2625)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.Camera.onCameraEnterCompleted(Camera.java:2334)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.Camera.onStartPreviewCompleted(Camera.java:1034)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.engine.RequestEventManager.lambda$handlePreviewRequestApplied$1$RequestEventManager(RequestEventManager.java:221)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.engine.-$$Lambda$RequestEventManager$Bn_xBsxcRHSYb-bsNmOI0lTQwiU.run(Unknown Source:2)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.engine.CommonEngine.lambda$postToUiThread$3$CommonEngine(CommonEngine.java:1110)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.sec.android.app.camera.engine.-$$Lambda$CommonEngine$HApqQcCeMPs-rB9SjWj4YI4-ayQ.run(Unknown Source:4)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Handler.handleCallback(Handler.java:938)          
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Handler.dispatchMessage(Handler.java:99)          
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.os.Looper.loop(Looper.java:246)                      
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at android.app.ActivityThread.main(ActivityThread.java:8512)    
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at java.lang.reflect.Method.invoke(Native Method)               
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:602)
07-15 12:12:34.667 14798 14798 E AndroidRuntime:        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1130)
```

It was at this point which I remembered that the first thing which I do when I get a new Galaxy phone is to [uninstall the bloatware](https://github.com/khlam/debloat-samsung-android/blob/master/commands.txt) which Samsung includes on the device. I guess something in the most recent update requires a package which I previously removed. After some further Googling I was able to trace this package to the [Samsung Capture](http://galaxystore.samsung.com/prepost/000005060510?langCd=en) app, and after installing the package from the Galaxy App Store I was finally able to use my camera again.
                                                                                                                        
