---
layout: post
title: Android 系统启动
date: 2018-05-23
category: Android
tag: Android
---

### 1，第一个系统进程init

&nbsp;&nbsp;Android设备的启动必须经历3个阶段，即Boot Loader, Linux Kernel和Android系统服务，默认情况下，他们都有各自的启动画面。  
&nbsp;&nbsp;严格来说，Android系统实际上是运行于Linux内核之上的一系列“服务进程”,并不算一个完整意义上的“操作系统”。这些进程是维持设备正常工作的关键，而他们的老祖宗就是init。  
&nbsp;&nbsp;作为Android中第一个被启动的进程，init的PID值为0。它通过解析init.rc脚本来构建出系统的运行形态--其他Android系统服务程序大多是在这个rc脚本中描述并相继启动的。
 init.rc源码目录：  
 ` system/core/rootdir/Init.rc `

&nbsp;&nbsp;作为Android系统的第一个进程， init将通过解析init.rc来陆续启动其他关键的系统服务进程--其中最重要的就是serviceManager，zygote, 和systemServer.  

### 2，Android的‘DNS服务器’ ServiceManager  

源码目录：` /frameworks/native/cmds/servicemanager `  

&nbsp;&nbsp;ServiceManager是在init.rc里描述并由init进程启动的。

&nbsp;&nbsp;ServiceManager是binder机制中的"DNS服务器",负责域名（某Binder服务在ServiceManager注册时提供的名称）到IP地址（由底层Binder驱动分配的值）的解析。

&nbsp;&nbsp;ServiceManager是一个Linux程序。它在设备中的存储路径是/system/bin/service-manager。  

&nbsp;&nbsp;ServiceManager所属的class是core，其他同类的系统进程包括，console（system/bin/sh）,adbd等。根据core组的特性，这些进程会同时被启动或停止。当ServiceManager每次重启时，其他关键进程如zygote,media,surfaceFlinger等也会被restart。   

### 3，“孕育”新的线程和进程 ---- Zygote  
&nbsp;&nbsp;Zygote 这个词的字面意思是”受精卵”,因而可以“孕育”出一个"新生命"。正如其名所示，Android中大多数应用进程和系统进程都是通过Zygote来生成的。  和ServiceManager类似，Zygote也是由init解析rc脚本时启动的。

&nbsp;&nbsp;Zygote所属的class为main，而不是core。和其同class的系统进程由netd, debuggerd, rild等。  
&nbsp;&nbsp;Zygote所在的程序名叫`app_process`， 而不像ServiceManager一样在一个独立的程序中。通过指定 --zygote参数，`app_process`可以识别出用户是否需要启动zygote。`app_process`扮演了“壳”的角色。那么它容纳了哪些内容呢？只要分析下`app_process`的主函数实现就知道答案了。

```
/*frameworks/base/cmds/app_process/app_main.cpp*/
int main(int argc, char* const argv[])
{
  ...
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));

  ...

    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    String8 niceName;
    String8 className;

    ++i;  // Skip unused "parent dir" argument.
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }

  ...

    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}

```

&nbsp;&nbsp;在这个场景中，init.rc中指定了--zygote选项，因而app_process接下来将启动“ZygoteInit” 并传入了“start-system-server”。之后ZygoteInit会运行在Java虚拟机上面。因为runtime这个变量就是AndroidRuntime对象，源码如下：

```
/*frameworks/base/core/jni/AndroidRuntime.cpp*/
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    ALOGD(">>>>>> START %s uid %d <<<<<<\n",
            className != NULL ? className : "(unknown)", getuid());

    static const String8 startSystemServer("start-system-server");

    /*
     * 'startSystemServer == true' means runtime is obsolete and not run from
     * init.rc anymore, so we print out the boot start event here.
     */
    for (size_t i = 0; i < options.size(); ++i) {
        if (options[i] == startSystemServer) {
           /* track our progress through the boot sequence */
           const int LOG_BOOT_PROGRESS_START = 3000;
           LOG_EVENT_LONG(LOG_BOOT_PROGRESS_START,  ns2ms(systemTime(SYSTEM_TIME_MONOTONIC)));
        }
    }

    const char* rootDir = getenv("ANDROID_ROOT");
    if (rootDir == NULL) {
        rootDir = "/system";
        if (!hasDir("/system")) {
            LOG_FATAL("No root directory specified, and /android does not exist.");
            return;
        }
        setenv("ANDROID_ROOT", rootDir, 1);
    }

    //const char* kernelHack = getenv("LD_ASSUME_KERNEL");
    //ALOGD("Found LD_ASSUME_KERNEL='%s'\n", kernelHack);

    /* start the virtual machine */
    //启动虚拟机
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
    }
    onVmCreated(env);

    /*
     * Register android functions.
     */
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }

    /*
     * We want to call main() with a String array with arguments in it.
     * At present we have two arguments, the class name and an option string.
     * Create an array to hold them.
     */
    jclass stringClass;
    jobjectArray strArray;
    jstring classNameStr;

    stringClass = env->FindClass("java/lang/String");
    assert(stringClass != NULL);
    strArray = env->NewObjectArray(options.size() + 1, stringClass, NULL);
    assert(strArray != NULL);
    classNameStr = env->NewStringUTF(className);
    assert(classNameStr != NULL);
    env->SetObjectArrayElement(strArray, 0, classNameStr);

    for (size_t i = 0; i < options.size(); ++i) {
        jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
        assert(optionsStr != NULL);
        env->SetObjectArrayElement(strArray, i + 1, optionsStr);
    }

    /*
     * Start VM.  This thread becomes the main thread of the VM, and will
     * not return until the VM exits.
     */
    char* slashClassName = toSlashClassName(className != NULL ? className : "");
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
        /* keep going */
    } else {
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V");
        if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
            /* keep going */
        } else {
        //通过JNI调用ZygoteInit的main函数
            env->CallStaticVoidMethod(startClass, startMeth, strArray);

#if 0
            if (env->ExceptionCheck())
                threadExitUncaughtException(env);
#endif
        }
    }
  ...
}

```

&nbsp;&nbsp;JNI调用了ZygoteInit的main函数之后，Zygote便进入了Java层，此前没有任何代码进入Java框架层，是zygote开创了Java框架层。

&nbsp;&nbsp;除了装载各种系统类外，Zygote的另一个重要工作就是启动SystemServer-----这是大部分Android系统服务的所在地。这些Android系统服务由Java语言编写。

```
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

public static void main(String argv[]) {
       ...
        try {
         ...       
            //注册Zygote用的Socket
            registerZygoteSocket(socketName);//1
           ...
           //预加载类和资源
           preload();//2
           ...
            if (startSystemServer) {
            //启动SystemServer进程
                startSystemServer(abiList, socketName);//3
            }
            Log.i(TAG, "Accepting command socket connections");
            //等待客户端请求
            runSelectLoop(abiList);//4
            closeServerSocket();
        } catch (MethodAndArgsCaller caller) {
            caller.run();
        } catch (RuntimeException ex) {
            Log.e(TAG, "Zygote died with exception", ex);
            closeServerSocket();
            throw ex;
        }
    }

```

Zygote启动流程就讲到这，Zygote进程共做了如下几件事：   
1.创建AppRuntime并调用其start方法，启动Zygote进程。   
2.创建DVM并为DVM注册JNI.   
3.通过JNI调用ZygoteInit的main函数进入Zygote的Java框架层。   
4.通过registerZygoteSocket函数创建服务端Socket，并通过runSelectLoop函数等待ActivityManagerService的请求来创建新的应用程序进程。   
5.启动SystemServer进程。  


### 4, Android系统服务 ------ SystemServer
&nbsp;&nbsp;SystemServer是Android进入Launcher前的最后准备。SystemServer提供了众多由Java语言编写的“系统服务”。  
[SystemServer进程的启动过程](https://blog.csdn.net/itachi85/article/details/55053356)


---
---
---

[推荐博客，Android系统启动流程](https://blog.csdn.net/itachi85/article/details/54783506)  
[Android系统启动过程](https://blog.csdn.net/dd864140130/article/details/57624948)


本教程大部分知识，摘抄自《深入理解Android内核设计思想》一书
