# FiND Pro (FiND with Fingerprint-based Positioning)

设备感知时延大是万场互联应用的痛点。FiND(Fingerprint-based Neighbor Discovery) 采用指纹进行快速发现，能够在干扰情况下，降低设备感知尾时延。但是，简单地应用FiND会降低设备感知的准确性。FiND pro在FiND的基础上，增加轻量级、自标注的室内指纹定位技术，从而提高感知准确性。

本项目是FiND中的指纹定位模块demo，通过机器学习的方法，实现指纹定位技术，提升信标发现的准确度。

## 运行环境

Spyder(Scientific PYthon Development EnviRonment)：Python 语言开发环境，用于运行本项目代码
Android Studio: 安卓程序开发环境，用于编译FiND demo apk

## 数据格式说明

* PHONEID：扫描设备的型号，例如 TAS-AL00
* LONGITUDE: 经度，取值范围-7695.9387549299299000 到 -7299.786516730871000
* LATITUDE: 纬度，取值范围4864745.7450159714 到 4865017.3646842018
* CITYID: 所在城市的ID，大于等于0的整数
* BUILDINGID: 所在建筑的ID，大于等于0的整数
* FLOOR: 所在楼层，整数
* BEACONID: 目标广播设备（如信标）标识符，字符串
* SPACEID：所在空间（如商铺）的ID，大于等于0的整数
* DISTANCERANGE：扫描设备与目标广播设备之间的相对距离范围，整数（0：近（1米内），1：较近（1-3米），2：远（大于3米））
* TIMESTAMP：时间戳，取UNIX时间格式，整数
* MAC001：周边广播设备（如Wi-Fi AP）MAC001的信号强度，取值范围-104 到 0。如果未检测到设备MAC001，则信号强度为+100. 
* ...
* MAC020：周边广播设备（如Wi-Fi AP）MAC020的信号强度，取值范围-104 到 0。如果未检测到设备MAC020，则信号强度为+100. 
* ...

> 注意：模板列出20个MAC，其中，MAC001 - MAC020根据广播设备标识符（MAC地址）进行排序得到。实际的MAC总数应该等于针对某个目标广播设备周边的所有其它广播设备数目的和，这个值可能远大于20。

## 跨平台模型部署

PMML是数据挖掘的一种通用的规范，它用统一的XML格式来描述我们生成的机器学习模型。这样无论你的模型是sklearn,R还是Spark MLlib生成的，我们都可以将其转化为标准的XML格式来存储。当我们需要将这个PMML的模型用于部署的时候，可以使用目标环境的解析PMML模型的库来加载模型，并做预测。可以看出，要使用PMML，需要两步的工作，第一块是将离线训练得到的模型转化为PMML模型文件，第二块是将PMML模型文件载入在线预测环境，进行预测。

以下是将pmml序列化成ser，并在安卓中部署ser的过程：
> 以下1-4步可以通过脚本自动化：进入目录\FingerprintPositioningML\utils\jpmml-android-master\pmml-android-example\src\main\pmml，依次执行0.GetPmmlFile.bat、1.ConvertPmml2Ser.bat、2.GetSerFile.bat即可。注意相对路径不要改变，且mvn等相关环境配置正确。
1. sklearn模型转换为pmml格式，生成FiNDTreeClassifier.pmml文件
> 需要安装sklearn2pmml模块，例如：在Anaconda Powershell Prompt 中管理员权限运行 pip install sklearn2pmml pypmml。同时安装必要的python组件：pip install matplotlib等

> 以下说明仅用于解释原理，运行时无需额外操作：pmml文件第二行中版本号必须为4.2（否则会出错），本项目中已自动处理
2. sklearn模型转换pmml格式后，不能在android环境下运行，因此需要先进行序列化操作
   ```
   git clone https://github.com/jpmml/jpmml-android.git 
   ```
3. 将FiNDTreeClassifier.pmml文件放入pmml-android-example/src/main/pmml/目录
   ```
   cd jpmml-android-master
   mvn clean install
   ```
> 需要安装maven工具。可参考：https://www.runoob.com/maven/maven-setup.html
4. 执行完毕后，在目录pmml-android-example/target/generated-sources/combined-assets/下能够找到生成的FiNDTreeClassifier.pmml.ser文件
> 以下说明仅用于解释原理，运行时无需额外操作：由于安卓无法直接采用java中的Evaluator，因此需要使用jpmml-android工程生成的pmml-android-1.0-SNAPSHOT.jar来创建自定义的Evaluator对象。具体使用方法为，将目录pmml-android\target中的pmml-android-1.0-SNAPSHOT.jar放入FiND demo工程的libs目录下，并调用以下方法：org.jpmml.android.EvaluatorUtil.createEvaluator(InputStream is)
5. 通过USB连接手机，通过adb工具将FiNDTreeClassifier.pmml.ser文件拷贝到手机/sdcard/Downloads/目录下
   ```
    adb push FiNDTreeClassifier.pmml.ser /sdcard/Download/
   ```
6. 模型部署完成






