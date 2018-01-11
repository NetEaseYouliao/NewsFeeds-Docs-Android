# Android设备唯一ID推荐方案

```java
/**
* 情形一：6.0以下系统（SDK版本低于23）
* <p>
* 1、取设备IMEI作为DeviceId；
* 2、取不到IMEI，次取mac地址作为DeviceId；
* 3、取不到mac地址，非模拟器，再取AndroidId作为DeviceId；
* 4、取不到AndroidId，用默认值“0000000000000000”（16位），一般指模拟器及部分山寨机等。
* <p>
* 情形二：6.0以上系统（SDK版本高于或等于23）
* <p>
* 由于6.0以上系统，Android对系统权限做了统一管理，获取IMEI需要用户授权“android.permission.READ_PHONE_STATE”，导致在用户授权/不授权的情况下取到的设备标识不唯一的问题。针对该情形，采用新的生成DeviceId的方法，规则如下：
* <p>
* 1、序列号规则serial：如果SDK版本>=8，则serial = Build.SERIAL，否则serial=“”;如果serial长度>20，则截取前20个字符；
* 2、生成DeviceId，由AndroidId、serial及“\t”拼接：DeviceId = AndroidId + "\t" + serial；
* 3、加密SHA
*
* @param context
* @return
*/
public static String getUniqueIDImp(Context context) {
    String deviceId;
    if (Build.VERSION.SDK_INT < 23) {
        deviceId = getUniqueID23Below(context);
    } else {
        deviceId = getUniqueID23EqualOrUp(context);
    }
    deviceId = SHA(deviceId);
    return deviceId;
}

public static String getUniqueID23Below(Context context) {
    String deviceId;
    deviceId = getIMEI(context);
    if ("".equals(deviceId)) {
        deviceId = getMacAddr(context);
    }
    if ("".equals(deviceId)) {
        deviceId = getAndroidId(context);
    }
    if ("".equals(deviceId)) {
        deviceId = DEFAULT_ANDROID_ID;
    }
    return deviceId;
}

public static String getUniqueID23EqualOrUp(Context context) {
    String deviceId;
    String serial = "";
    String androidId = getAndroidId(context);
    if (Build.VERSION.SDK_INT >= 8) {
        serial = getSerial();
        if (serial.length() > 20) {
            serial = serial.substring(0, 20);
        }
    }
    deviceId = androidId + "\t" + serial;
    return deviceId;
}

public static String SHA(final String toConvert) {
    if (null == toConvert)
        return "";
    try {
        final MessageDigest md = MessageDigest.getInstance("SHA");
        final byte[] digest = md.digest(toConvert.getBytes("UTF-8"));
        final BigInteger hashedNumber = new BigInteger(1, digest);
        return hashedNumber.toString(16);
    } catch (final NoSuchAlgorithmException e) {
        throw new RuntimeException(e);
    } catch (final UnsupportedEncodingException e) {
        throw new RuntimeException(e);
    }
}
```