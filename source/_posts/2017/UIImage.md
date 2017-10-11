---
title: UIImage
date: 2014-10-16 10:47:43
tags:
- iOS
---

### UIImage占用内存大小
```
UIImage *image = [UIImage imageNamed:@"aa"];
NSUInteger size  = CGImageGetHeight(image.CGImage) * CGImageGetBytesPerRow(image.CGImage);
```

### 获取图片某一点的颜色
```
- (UIColor*) getPixelColorAtLocation:(CGPoint)point inImage:(UIImage *)image
{
UIColor* color = nil;
CGImageRef inImage = image.CGImage;
CGContextRef cgctx = [self createARGBBitmapContextFromImage:inImage];
if (cgctx == NULL) {
return nil; /* error */
}
size_t w = CGImageGetWidth(inImage);
size_t h = CGImageGetHeight(inImage);
CGRect rect = {{0,0},{w,h}};
CGContextDrawImage(cgctx, rect, inImage);
unsigned char* data = CGBitmapContextGetData (cgctx);
if (data != NULL) {
int offset = 4*((w*round(point.y))+round(point.x));
int alpha =  data[offset];
int red = data[offset+1];
int green = data[offset+2];
int blue = data[offset+3];
color = [UIColor colorWithRed:(red/255.0f) green:(green/255.0f) blue:
(blue/255.0f) alpha:(alpha/255.0f)];
}
CGContextRelease(cgctx);
if (data) {
free(data);
}
return color;
}
```
