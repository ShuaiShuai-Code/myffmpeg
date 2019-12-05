# ffmpeg简单实用，初学者可以看，高手走请走开
**安卓开发者万里长城又走了一步，真是痛快**


![image.png](https://upload-images.jianshu.io/upload_images/4752376-6464a4ab71503774.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**昨天生成了动态库今天就新建了一个项目，测试了一下**


![image.png](https://upload-images.jianshu.io/upload_images/4752376-86ddf5b297841137.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**目录结构如下图：**


![image.png](https://upload-images.jianshu.io/upload_images/4752376-80f399d4e55820ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**首先新建一个c++项目，新建完了会有一个cpp的文件夹，新建项目的Mainactivity要用java不要用kotlin，这个很坑的，kotlin目前好像不行，不知道是不是我没找到方法，卡这里至少掉了十根头发**
![image.png](https://upload-images.jianshu.io/upload_images/4752376-d8b72b477e4f9657.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个CMakeLists.txt也很坑的，开始看的网上的Adroid.mk和Application.mk,技不如人，看着人家一步一步搞的，自己的有问题，最后发现CMakeLists.txt就是替换Adroid.mk和Application.mk这两个东西的
**还有include这个文件夹，开始放到cpp下面，native-lib可以引入头文件，放到jniLibs目录下就无法引入了，这个最后写的绝对路径```"./../jniLibs/include/libavcodec/avcodec.h"```这样的*，然后include头文件的.h文件很挫有错误的，就改，原本```#include "libavutil/rational.h"```修改为```#include "../libavutil/rational.h"```,有十来个文件把一个一个改**

![image.png](https://upload-images.jianshu.io/upload_images/4752376-e822ebd1061b8234.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**CMakeLists.txt配置**
```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

#add_library( # Sets the name of the library.
#        native-lib
#
#        # Sets the library as a shared library.
#        SHARED
#
#        # Provides a relative path to your source file(s).
#        src/main/cpp/native-lib.cpp)

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

#target_link_libraries( # Specifies the target library.
#        native-lib
#
#        # Links the target library to the log library
#        # included in the NDK.
#        ${log-lib})


set(distribution_DIR ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})
include_directories(/src/main/jniLibs/include)

add_library( avutil-55
        SHARED
        IMPORTED )
set_target_properties( avutil-55
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libavutil-55.so)

add_library( swresample-2
        SHARED
        IMPORTED )
set_target_properties( swresample-2
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libswresample-2.so)

add_library( avcodec-57
        SHARED
        IMPORTED )
set_target_properties( avcodec-57
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libavcodec-57.so)

add_library( avfilter-6
        SHARED
        IMPORTED )
set_target_properties( avfilter-6
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libavfilter-6.so)

add_library( swscale-4
        SHARED
        IMPORTED )
set_target_properties( swscale-4
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libswscale-4.so)

add_library( avdevice-57
        SHARED
        IMPORTED)
set_target_properties( avdevice-57
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libavdevice-57.so )

add_library( avformat-57
        SHARED
        IMPORTED )
set_target_properties( avformat-57
        PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/libavformat-57.so)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")

add_library( native-lib
        SHARED
        src/main/cpp/native-lib.cpp)



target_link_libraries(native-lib swresample-2 avcodec-57 avfilter-6 swscale-4 avdevice-57 avformat-57 avutil-55
        ${log-lib})

```
**native-lib.cpp原码**
```
#include <jni.h>
#include <string>


extern "C" {
#include "./../jniLibs/include/libavutil/avutil.h"
#include "./../jniLibs/include/libavfilter/avfilter.h"
#include "./../jniLibs/include/libavformat/avformat.h"
#include "./../jniLibs/include/libavcodec/avcodec.h"
jstring
Java_com_ffmpeg_myffmpeg_MainActivity1_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
    }

jstring
Java_com_ffmpeg_myffmpeg_MainActivity1_avfilterinfo(
        JNIEnv *env, jobject) {
    char info[40000] = {0};
    avfilter_register_all();

    AVFilter *f_temp = (AVFilter *)avfilter_next(NULL);
    while(f_temp != NULL) {
        sprintf(info, "%s%s\n", info, f_temp->name);
        f_temp = f_temp->next;
    }
    return env->NewStringUTF(info);
}

jstring
Java_com_ffmpeg_myffmpeg_MainActivity1_urlprotocolinfo(
        JNIEnv *env, jobject) {
    char info[40000] = {0};
    av_register_all();

    struct URLProtocol *pup = NULL;

    struct URLProtocol **p_temp = &pup;
    avio_enum_protocols((void **) p_temp, 0);

    while ((*p_temp) != NULL) {
        sprintf(info, "%sInput: %s\n", info, avio_enum_protocols((void **) p_temp, 0));
    }
    pup = NULL;
    avio_enum_protocols((void **) p_temp, 1);
    while ((*p_temp) != NULL) {
        sprintf(info, "%sInput: %s\n", info, avio_enum_protocols((void **) p_temp, 1));
    }
    return env->NewStringUTF(info);
}

jstring
Java_com_ffmpeg_myffmpeg_MainActivity1_avformatinfo(
        JNIEnv *env, jobject) {
    char info[40000] = {0};

    av_register_all();

    AVInputFormat *if_temp = av_iformat_next(NULL);
    AVOutputFormat *of_temp = av_oformat_next(NULL);
    while (if_temp != NULL) {
        sprintf(info, "%sInput: %s\n", info, if_temp->name);
        if_temp = if_temp->next;
    }
    while (of_temp != NULL) {
        sprintf(info, "%sOutput: %s\n", info, of_temp->name);
        of_temp = of_temp->next;
    }
    return env->NewStringUTF(info);
}

jstring
Java_com_ffmpeg_myffmpeg_MainActivity1_avcodecinfo(
        JNIEnv *env, jobject) {
    char info[40000] = {0};

    av_register_all();

    AVCodec *c_temp = av_codec_next(NULL);

    while (c_temp != NULL) {
        if (c_temp->decode != NULL) {
            sprintf(info, "%sdecode:", info);
        } else {
            sprintf(info, "%sencode:", info);
        }
        switch (c_temp->type) {
            case AVMEDIA_TYPE_VIDEO:
                sprintf(info, "%s(video):", info);
                break;
            case AVMEDIA_TYPE_AUDIO:
                sprintf(info, "%s(audio):", info);
                break;
            default:
                sprintf(info, "%s(other):", info);
                break;
        }
        sprintf(info, "%s[%10s]\n", info, c_temp->name);
        c_temp = c_temp->next;
    }

    return env->NewStringUTF(info);
}



}





```
**MainActivity原码，这里开始是Mainactivity.kt,最后修改为Mainactivity.java,报错，又修改为了MainActivity1.java的**
```
package com.ffmpeg.myffmpeg;

import android.os.Bundle;
import android.widget.TextView;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity1 extends AppCompatActivity {
    private TextView textView;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView=findViewById(R.id.sample_text);
        textView.setText(stringFromJNI()+
                urlprotocolinfo()+
                avformatinfo()+
                avfilterinfo());
    }
    public native String stringFromJNI();
    public native String urlprotocolinfo();
    public native String avformatinfo();
    public native String avfilterinfo();

    static {
            System.loadLibrary("native-lib");
            System.loadLibrary("avcodec-57");
            System.loadLibrary("avdevice-57");
            System.loadLibrary("avfilter-6");
            System.loadLibrary("avformat-57");
            System.loadLibrary("avutil-55");
            System.loadLibrary("postproc-54");
            System.loadLibrary("swresample-2");
            System.loadLibrary("swscale-4");
    }
}





```

最后上传代码一份： 别忘点个赞哈
https://github.com/gethub-json/myffmpeg
