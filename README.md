# turbojpeg-ios
libturbojpeg libjpeg iOS 静态库

turbojpeg压缩
```
+(NSData *)turboJpegcompress:(NSData *)data withWidth:(int)width withHeight:(int)height {
    tjhandle handle = tjInitCompress();
    unsigned long size = TJBUFSIZE(width, height);
    unsigned char *dstBuf = malloc(size);
    tjCompress(handle, (unsigned char *)data.bytes, width, width * 4, height, 4, dstBuf, &size, 0, 100, 0);
    NSData *dstData = [[NSData alloc]initWithBytes:dstBuf length:size];
    free(dstBuf);
    return dstData;
}
```

turbojpeg解压
```
+(NSData *)turboJpegDecompress:(NSData *)data withWidth:(int)width withHeight:(int)height {
    tjhandle deHandle = tjInitDecompress();
    long size = width * height * 4;
    unsigned char *dstBuf = malloc(size);
    int result = tjDecompress(deHandle, (unsigned char *)data.bytes, data.length, dstBuf, width, width * 4, height, 4, TJFLAG_FORCESSE3);
    if (result == -1) {
        NSLog(@"turbojpeg failed");
    }
    NSData *dstData = [[NSData alloc]initWithBytes:dstBuf length:size];
    free(dstBuf);
    return dstData;
}
```
