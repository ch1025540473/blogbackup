layout: pages
title: 安卓安全-apk完整性校验
date: 2016-11-23 15:16:51
tags: android安全
---


## crc32
全称是“Cyclic Redundancy Check”，中文名是“循环冗余码”。
> 它的计算是非常非常非常严格的。严格到什么程度呢？你的程序只要被改动了一个字节（甚至只是大小写的改动），它的值就会跟原来的不同

至于crc32的值是如何计算的以及实现原理，本文不做讲解，有兴趣的可以google。
<!--more-->
在apk中，反编译后恶意的篡改代码重新打包主要集中在dex文件中，所以可以通过获取dex文件的crc32值来观察dex文件是否被篡改过了。看代码：
```
/**
 * 获取当前apk的crc32值
 * @return
 */
public static Long getCrc32(Context context){
    String apkPath = ApkPathUtils.getApkPath(context);
    ZipFile zipfile = null;
    ZipEntry dexentry = null;
    try {
        zipfile = new ZipFile(apkPath);
        dexentry = zipfile.getEntry("classes.dex");
    } catch (IOException e) {
        e.printStackTrace();
    }
    return dexentry.getCrc();
}
```
上文代码中的工具类ApkPathUtils,
```
public class ApkPathUtils {

    public static String getApkPath(Context context){
        try {
            PackageInfo packageInfo = context.getPackageManager().getPackageInfo(context.getPackageName(),PackageManager.GET_META_DATA);
            ApplicationInfo applicationInfo = packageInfo.applicationInfo;
            return applicationInfo.publicSourceDir; // 获取当前apk包的绝对路径
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        return "";
    }
}
```
看getCrc32(Context context) 这个方法，只是给ZipFile这类塞了一个当前apk包的绝对路径就可以了，而这个当前apk的绝对路径可以通过PackageInfo拿到，记得加上读外部存储卡的权限。当然，5.0以后的权限另做处理
获取到crc32值之后一般有两种处理方式：
1. 把crc32值放在本地的string.xml文件中，在运行时获取比对，如果与.xml获取到的crc32值不同，则说明代码有变动，则apk已经被修改。注意：不要放在raw，asset中的文件。只有不牵扯javad代码的修改，放在本地的任意位置都行。
2. 将获取到的crc32值加密送到后台，解密与后台保存的crc32值进行比对。通过接口获取比对结果，判断apk是否被修改过。

> 注意：校验的代码最好加上版本的判断，只有在release版本的时候才会去校验，debug模式的时候不做判断，因为在debug模式的时候也判断的话，你就没法调试代码了，永远在修改，获取到的值永远不同

## apk Hash值判断

> MD5Hash算法的”数字指纹”特性，使它成为目前应用最广泛的一种文件完整性校验

先看校验方法
```
/**
 * 获取当前apk包的hash值
 * @return
 */
public String getHash(Context context){

    MessageDigest msgDigest = null;

    String apkPath = ApkPathUtils.getApkPath(context);
    FileInputStream fis = null;
    try {
        msgDigest = MessageDigest.getInstance("SHA-1");
        byte[] bytes = new byte[1024];
        int byteCount;
        fis= new FileInputStream(new File(apkPath));
        while ((byteCount = fis.read(bytes)) > 0)
        {
            msgDigest.update(bytes, 0, byteCount);
        }
        BigInteger bi = new BigInteger(1, msgDigest.digest());
        return bi.toString(16);
    } catch (Exception e) {
        e.printStackTrace();
    }finally {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return "";
}
```
从上面代码可以看出，就是对apk文件的流进行了sha-1，并转成16进制，获取到的hash值。因此
> 与 DEX 校验不同 APK 检验必须把把计算好的 Hash 值放在网络服务端，因为对 APK 的任何改动都会影响到最后的 Hash 值。

当你获取到hash值后放在本地，这时候apk的hash值已经改变，所以你永远获取不到一个准确版本的hash值，所以获取后只能放在服务端进行校验，校验的方式与crc32在服务端的校验是相同的，不再赘述。

crc32,apk-hash值都可以通过dos命令行获取。

当然上述的保护方式容易被暴力破解, 完整性检查最终还是通过返回 true/false 来控制后续代码逻辑的走向,如果攻击者直接修改代码逻辑,完整性检查始终返回 true,那这种方法就无效了当然你可以把校验逻辑放进.so文件，加大破解的难度。但还是不能完全保证安全，而且遇到apk动态加载，会自动创建dex文件的情况，或者应用加固，不知道这种方式还能不能奏效，有做过的朋友可以给点意见
