[toc]

---

# 引入Maven


```xml
 <!--二维码生成和解析相关的jar包【生成】【解析】-->
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.4.1</version>
</dependency>
```

# google.zxing 解析二维码异常问题

今天发现两个内容一模一样的二维码，但是一个是使用苹果6的手机截图的，一个是苹果13截图的，但是苹果6的会抛出一个NoFoundException异常。

有问题的代码：

```java
private static String decodeQrcode(URL file) {
    BufferedImage image = ImgUtil.read(file);
    LuminanceSource source = new BufferedImageLuminanceSource(image);
    Binarizer binarizer = new HybridBinarizer(source);
    BinaryBitmap binaryBitmap = new BinaryBitmap(binarizer);
    Map<DecodeHintType, Object> hints = Maps.newEnumMap(DecodeHintType.class);
    hints.put(DecodeHintType.CHARACTER_SET, Charsets.UTF_8.name());
    try {
        com.google.zxing.Result result = new MultiFormatReader().decode(binaryBitmap, hints);
        return result.getText();
    } catch (NotFoundException e) {
        e.printStackTrace();
        System.out.println(e.toString());
        return "";
    }
}
```

这时就会出现二维码异常的问题，大部分的二维码可以正常解析，但是有些二维码会报异常。经过各种尝试，做了一下更改：

1. HybridBinarizer换成GlobalHistogramBinarizer
   - zxing官方默认的HybridBinarizer，两者区别HybridBinarizer算法执行效率上要慢一些，但是更有效，专门针对黑白相间的图像设计，也更适用于有阴影和渐变的二维码图像。
   - GlobalHistogramBinarizer算法适用于低端机，对手机要求不高，识别速度快，精度高，但是无法处理阴影和渐变两个情况。

2. MultiFormatReader换成QRCodeMultiReader
   - 这里指定了单个类型，即QRCode。但是尝试了QrCodeReader解析发现还是会出现异常。于是使用了批量解析的方式，然后默认选取第一张。

更改完后，代码如下：

```java
private static String decodeQrcode(URL file) {
    BufferedImage image = ImgUtil.read(file);
    LuminanceSource source = new BufferedImageLuminanceSource(image);
    Binarizer binarizer = new GlobalHistogramBinarizer(source);
    BinaryBitmap binaryBitmap = new BinaryBitmap(binarizer);
    Map<DecodeHintType, Object> hints = Maps.newEnumMap(DecodeHintType.class);
    hints.put(DecodeHintType.POSSIBLE_FORMATS, EnumSet.allOf(BarcodeFormat.class));
    hints.put(DecodeHintType.PURE_BARCODE, Boolean.TRUE);
    hints.put(DecodeHintType.TRY_HARDER, BarcodeFormat.QR_CODE);
    hints.put(DecodeHintType.CHARACTER_SET, Charsets.UTF_8.name());
    try {
        //            QRCodeReader qrCodeReader =new QRCodeReader();
        //            return qrCodeReader.decode(binaryBitmap,hints).getText();
        com.google.zxing.Result result[] = new QRCodeMultiReader().decodeMultiple(binaryBitmap, hints);
        return result[0].getText();
    } catch (NotFoundException e) {
        e.printStackTrace();
        System.out.println(e.toString());
        return "";
    }
}
```

