---
title: Binder的学习
copyright: true
date: 2019-10-20 00:02:11
tags: [Binder]
category: Android
---
### 进程通讯的方式
1. 管道，耗性能
2. 共享内存，多进程访问，管理混乱
3. Socket适用于网络通讯，进程通讯不适用

### Binder
1. binder机制为每个进程都分配了UID和PID来作为鉴别身份的标识，安全性
2. 对数据只进行了一次拷贝，通过驱动的内核空间拷贝数据，不需要额外的同步处理
3. 使用简单C/S架构，实现面向对象的调用方式即在使用binder时，就和调用1个本地对象实例一样

#### Binder的四个重要角色
1. server
2. Client
3. ServiceMannager
4. Binder驱动(存在线程池 16个)，具体实现通过内存映射，内部调用了mmap()函数
前三者在用户空间，Binder驱动在内核空间

#### AIDL接口定义语言
是对Binder通讯的封装
IBinder代表有能力进行快进程的能力
IIterface 拥有了Binder机制的能力，只有一个asBinder的方法
Binder
Stub 本地对象

#### Binder的源码分析
1. 打开binder设备
```
frameworks /native /cmds /servicemanager /service_manager.c
driver = "/dev/binder";  //返回文件描述符
```
2. buffer的创建(用于进程间数据传递)
```
bs = binder_open(driver, 128*1024); binder，创建128k的那内存映射
```
3. 开辟内存呢映射128k
```
frameworks /native /cmds /servicemanager /binder.c
bs->mapped = mmap(NULL, mapsize, PROT_READ, MAP_PRIVATE, bs->fd, 0);
mmap()命令
device /google /cuttlefish_kernel /4.4-x86_64 /System.map
在系统启动的时候开辟了内存映射
```
4. serviceManager的启动
```
system /core /rootdir /init.rc
start servicemanager
在系统启动的时候做了启动服务
```
5. 打包Parcel,数据写入binder驱动，copy_from_user
```
打包Parcel
frameworks native libs binder IServiceManager.cpp
addService

数据写入binder驱动
frameworks native libs binder IPCThreadState.cpp
writeTransactionData
```
6. 服务注册，添加到链表SVClist中
7. 定义主线程中的线程池
8. 循环从mIn和mOut中读和写数据请求，发到binder设备中

### 三方登录实例
A应用client端，B应用为Server端。A调用B进行登录。两应用aidl文件的包名必须一致
#### A应用
1. 创建ILoginInterface.aidl
```
// ILoginInterface.aidl
package com.netease.binder;

// Declare any non-default types here with import statements

interface ILoginInterface {

    // 登录
    void login();

    // 登录返回
    void loginCallback(boolean loginStatus, String loginUser);
}
```
2. 在A应用中建立Binder连接,记得销毁
```
public void initBindService() {
        Intent intent = new Intent();
        // 设置Server应用Action
        intent.setAction("BinderB_Action");
        // 设置Server应用包名（5.1+要求）
        intent.setPackage("com.netease.binder.b");
        // 开始绑定服务
        bindService(intent, conn, BIND_AUTO_CREATE);
        // 标识跨进程绑定
        isStartRemote = true;
    }

    // 服务连接
    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iLogin = ILoginInterface.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

```
3. 因为需要接受B进程返回的数据，需要同样创建一个服务，
```
package com.netease.binder.a.service;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;

import com.netease.binder.ILoginInterface;

public class MyService extends Service {

    @Override
    public IBinder onBind(Intent intent) {

        return new ILoginInterface.Stub() {
            @Override
            public void login() throws RemoteException {
            }

            @Override
            public void loginCallback(boolean loginStatus, String loginUser) throws RemoteException {
                Log.e("netease >>> ", "loginStatus: " + loginStatus + " / loginUser: " + loginUser);
            }
        };
    }
}
```
#### B应用
1. 创建ILoginInterface.aidl
2. 创建对应的service服务
```
public class MyService extends Service {

    @Override
    public IBinder onBind(Intent intent) {

        return new ILoginInterface.Stub() {
            @Override
            public void login() throws RemoteException {
                Log.e("netease >>> ", "BinderB_MyService");
                // 单项通信有问题，真实项目双向通信，双服务绑定
                serviceStartActivity();
            }

            @Override
            public void loginCallback(boolean loginStatus, String loginUser) throws RemoteException {
            }
        };
    }

    /**
     * 在Service启动Activity，需要配置：.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
     */
    private void serviceStartActivity() {
        Intent intent = new Intent(this, MainActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    }

}
```
3. 在AndroidManifest配置文件中注册服务
```
<!--
                 代表在应用程序里，当需要该service时，会自动创建新的进程。
                 android:process=":remote"

                 是否可以被系统实例化
                 android:enabled="true"

                 代表是否能被其他应用隐式调用
                 android:exported="true"
        -->
        <service
            android:name=".service.MyService"
            android:enabled="true"
            android:exported="true"
            android:process=":remote_service">

            <intent-filter>
                <!-- 激活 MyService 唯一name，不能重名-->
                <action android:name="BinderB_Action" />
            </intent-filter>
        </service>
```
4. 当A请求登录时，会唤起B应用打开登录界面，当输入完登录信息后需要将登录结果返回给A应用，因此同样需要A应用有一个跨进程服务接受B进程的登录结果，所以同样在B中创建Ade服务连接
```
public void initBindService() {
        Intent intent = new Intent();
        // 设置Server应用Action
        intent.setAction("BinderA_Action");
        // 设置Server应用包名（5.1+要求）
        intent.setPackage("com.netease.binder.a");
        // 开始绑定服务
        bindService(intent, conn, BIND_AUTO_CREATE);
        // 标识跨进程绑定
        isStartRemote = true;
    }

    // 服务连接
    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iLogin = ILoginInterface.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
```