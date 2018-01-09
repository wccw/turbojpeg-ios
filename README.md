# turbojpeg-ios
libturbojpeg libjpeg iOS 静态库

turbojpeg压缩
```
+(NSData *)turboJpegCopress:(NSData *)data withWidth:(int)width withHeight:(int)height {
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

## 其它压缩算法
jpeg压缩属于图片压缩算法，还有对文件而言的通用压缩算法，通用压缩算法也能够对图片进行压缩，例如 [zstd](https://github.com/facebook/zstd) [snappy](https://github.com/google/snappy) [lz4](https://github.com/lz4/lz4) 

zstd压缩
```
+(NSData *)zstdCompress:(NSData *)data {
    size_t const cBuffSize = ZSTD_compressBound(data.length);
    void* const cBuff = malloc(cBuffSize);
    size_t const cSize = ZSTD_compress(cBuff, cBuffSize, data.bytes, data.length, 1);
    if (ZSTD_isError(cSize)) {
        printf("error compressing : %s \n", ZSTD_getErrorName(cSize));
        exit(8);
    }
    NSData *comData = [[NSData alloc]initWithBytes:cBuff length:cSize];
    free(cBuff);
    return comData;
}
```

zstd解压
```
+(NSData *)zstdDecompress:(NSData *)data {
    long rSize = ZSTD_getDecompressedSize(data.bytes, data.length);
    if (rSize == ZSTD_CONTENTSIZE_ERROR) {
        printf("it was not compressed by zstd.\n");
    } else if (rSize==ZSTD_CONTENTSIZE_UNKNOWN) {
        printf("original size unknown. Use streaming decompression instead.\n");
    }
    void* const rBuff = malloc(rSize);
    size_t const dSize = ZSTD_decompress(rBuff, rSize, data.bytes, data.length);
    if (dSize != rSize) {
        printf("error decoding : %s \n", ZSTD_getErrorName(dSize));
    }
    NSData *content = [[NSData alloc]initWithBytes:rBuff length:dSize];
    free(rBuff);
    return content;
}
```

snappy压缩
```
+(NSData *)snappyCompress:(NSData *)data {
    size_t outSize = snappy_max_compressed_length(data.length);
    char* outBuff = malloc(outSize);
    snappy_compress(data.bytes, data.length, outBuff, &outSize);
    NSData *snappyData = [[NSData alloc]initWithBytes:outBuff length:outSize];
    free(outBuff);
    return snappyData;
}
```

snappy解压
```
+(NSData *)snappyDecompress:(NSData *)data {
    size_t outSize;
    snappy_status status = snappy_uncompressed_length(data.bytes, data.length, &outSize);
    if (status != 0) {
        printf("snappy status is zero");
    }
    char* outBuff = malloc(outSize);
    snappy_uncompress(data.bytes, data.length, outBuff, &outSize);
    NSData *snappyData = [[NSData alloc]initWithBytes:outBuff length:outSize];
    free(outBuff);
    return snappyData;
}
```

lz4压缩
```
+(NSData *)lz4Compress:(NSData *)data {
    int dstSize = LZ4_compressBound((int)data.length);
    char *outBuff = malloc(dstSize);
    //dstSize = LZ4_compress_default(data.bytes, outBuff, (int)data.length, dstSize);
    dstSize = LZ4_compress_fast(data.bytes, outBuff, (int)data.length, dstSize, 1);
    NSData *dstData = [[NSData alloc]initWithBytes:outBuff length:dstSize];
    free(outBuff);
    return dstData;
}
```

lz4解压
```
+(NSData *)lz4Decompress:(NSData *)data {
    int dstSize = data.length;
    char *outBuff = malloc(dstSize);
    dstSize = LZ4_decompress_safe(data.bytes, outBuff, (int)conData.length, dstSize);
    //dstSize = LZ4_decompress_fast(data.bytes, outBuff, dstSize);
    NSData *dstData = [[NSData alloc]initWithBytes:outBuff length:dstSize];
    free(outBuff);
    return dstData;
}
```
