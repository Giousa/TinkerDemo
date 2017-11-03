# Tinker热修复

一、AS配置：

1、app  build.gradle

注意：核心依赖：

<pre>
//可选，用于生成application类
provided('com.tencent.tinker:tinker-android-anno:1.7.7')
//tinker的核心库
compile('com.tencent.tinker:tinker-android-lib:1.7.7')
</pre>

需要混淆：

<pre>
minifyEnabled true
</pre>

<pre>
apply plugin: 'com.android.application'
android {
    compileSdkVersion 26
    buildToolsVersion "26.0.1"
    defaultConfig {
        applicationId "com.zmm.tinkerdemo"
        minSdkVersion 18
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
    //可选，用于生成application类
    provided('com.tencent.tinker:tinker-android-anno:1.7.7')
    //tinker的核心库
    compile('com.tencent.tinker:tinker-android-lib:1.7.7')
}
</pre>

2、Tinker混淆：因为是混淆状态，若是不导入这个，运行会报错，无法打开app，因为找不到相关的类
<pre>
-keepattributes *Annotation*
-dontwarn com.tencent.tinker.anno.AnnotationProcessor
-keep @com.tencent.tinker.anno.DefaultLifeCycle public class *
-keep public class * extends android.app.Application {
    *;
}
-keep public class com.tencent.tinker.loader.app.ApplicationLifeCycle {
    *;
}
-keep public class * implements com.tencent.tinker.loader.app.ApplicationLifeCycle {
    *;
}
-keep public class com.tencent.tinker.loader.TinkerLoader {
    *;
}
-keep public class * extends com.tencent.tinker.loader.TinkerLoader {
    *;
}
-keep public class com.tencent.tinker.loader.TinkerTestDexLoad {
    *;
}
-keep public class com.tencent.tinker.loader.TinkerTestAndroidNClassLoader {
    *;
}
#for command line version, we must keep all the loader class to avoid proguard mapping conflict
#your dex.loader pattern here
-keep public class com.tencent.tinker.loader.** {
    *;
}
-keep class tinker.sample.android.app.SampleApplication {
    *;
}
</pre>

3、Application：不要忘记build一下哦
<pre>
package com.zmm.tinkerdemo;
import android.app.Application;
import android.content.Context;
import android.content.Intent;
import com.tencent.tinker.anno.DefaultLifeCycle;
import com.tencent.tinker.lib.tinker.TinkerInstaller;
import com.tencent.tinker.loader.app.ApplicationLike;
import com.tencent.tinker.loader.shareutil.ShareConstants;
/**
 * Description:
 * Author:zhangmengmeng
 * Date:2017/11/2
 * Time:下午3:30
 */
@DefaultLifeCycle(application = ".SimpleTinkerInApplication",
        flags = ShareConstants.TINKER_ENABLE_ALL,
        loadVerifyFlag = false)
public class SimpleTinkerInApplicationLike extends ApplicationLike {
    public SimpleTinkerInApplicationLike(Application application, int tinkerFlags, boolean tinkerLoadVerifyFlag, long applicationStartElapsedTime, long applicationStartMillisTime, Intent tinkerResultIntent) {
        super(application, tinkerFlags, tinkerLoadVerifyFlag, applicationStartElapsedTime, applicationStartMillisTime, tinkerResultIntent);
    }
    @Override
    public void onBaseContextAttached(Context base) {
        super.onBaseContextAttached(base);
    }
    @Override
    public void onCreate() {
        super.onCreate();
        TinkerInstaller.install(this);
    }
}
</pre>

4、清单文件：

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/mani.png)


5、打包apk：将生成apk命名为：old.apk

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/old.png)


6、copy mapping.txt

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/mapping.png)

7、mapping.txt 存入 app文件目录下，和proguard-rules.pro文件同级

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/mapp_pro.png)

8、proguard-rules.pro下添加：
<pre>
-applymapping mapping.txt
</pre>

9、更改项目功能

10、打包apk，命名为new.apk

11、配置，将签名文件jks、old.apk、new.apk等文件存入以下文件夹

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/setting.png)

12、修改tinker_config.xml配置

①修改成我们自己的Application的地址

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/application.png)

②使用我们自己的jks和密码

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/jks.png)


13、执行命令行，生成patch
<pre>
java -jar tinker-patch-cli-1.7.7.jar -old old.apk -new new.apk -config tinker_config.xml -out output
</pre>

14、文件在此：

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/patch.png)

15、使用：案例中是因为有个点击事件：
<pre>
public void loadPatch(View view) {
        TinkerInstaller.onReceiveUpgradePatch(getApplicationContext(),
                Environment.getExternalStorageDirectory().getAbsolutePath() + "/patch_signed.apk");
    }

</pre>
所以，我们只需要将这个patch文件导入/mnt/sdcard/ 目录下，点击后，app重启，然后，就是新的apk了。


展示图:

![image](https://github.com/Giousa/TinkerDemo/blob/master/screenshot/tinkerdemo.gif)
