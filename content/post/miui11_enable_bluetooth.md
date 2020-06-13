---
title: "MIUI开启蓝牙耳机ACC模式"
date: 2020-06-13T12:47:46+08:00
---

> 手机比较旧了，刷回稳定版的系统，提高一下流畅度，结果发现蓝牙耳机没办法开启ACC模式。

# 起

在网络上查找资料，发现存在一个配置文件`/system/etc/bluetooth/interop_database.conf`

里面设置了黑白名单，来控制耳机是否开启ACC模式。

`[INTEROP_ENABLE_ACC_CODEC]`

很明显这个就是白名单，将我们的耳机计入这个条目下就可以开启acc模式，在xda论坛上大神给的是`00:00:00=Address_Based`，需要前6位的耳机mac地址。我们需要知道耳机的mac地址。

# 承

- step0

  很显然，MIUI把耳机的mac地址屏蔽掉了，在设置界面了没有办法查看。

- step1 

  开发者模式查找，并没有，AIDA64里也没有。

- 作为一个曾经开发过安卓app的开发者，当然想到了用代码的手段

  ```java

  //获取已经保存过的设备信息
      Set<BluetoothDevice> devices = mBluetoothAdapter.getBondedDevices();
      if (devices.size()>0) {  
          for(Iterator<BluetoothDevice> iterator=devices.iterator();iterator.hasNext();){  
                BluetoothDevice bluetoothDevice=(BluetoothDevice)iterator.next();  
                System.out.println("设备："+bluetoothDevice.getName() + " " + bluetoothDevice.getAddress());
          }  
      }
  ```

  但是需要安卓Android Studio，下载sdk编译等操作，甚为麻烦。

- 工具APP

  [Bluetooth Mac Address Finder](https://play.google.com/store/apps/details?id=com.codeweavers.bluetoothmacaddressfinder&hl=zh)，确实能找到耳机mac地址，但是尝试之后，发现没有效果，还是不能切换。

# 转

- 最后我们发现在默认的列表里有的后缀并不是Address_Based，而是Name_Based。我们是不是可以另辟蹊径，使用设备名字呢。答案是可以的

  ![](https://img.0xaa.top//1592032266.jpg)

# 折



  ![](https://img.0xaa.top//1592032321(1).jpg)

  可以看见我们成功的让手机支持了耳机的ACC模式。在看别人的解决方案的时候，还是需要多加思考。

  另外吐槽一下MIUI。