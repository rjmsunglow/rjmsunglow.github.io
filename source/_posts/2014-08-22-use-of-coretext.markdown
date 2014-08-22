---
layout: post
title: "coreText使用总结"
date: 2014-08-22 13:08
comments: true
categories: 
---
####NSAttributedString 可以将一段文字中的部分文字设置单独的字体和颜色。与UITouch结合可以实现点击不同文字触发不同事件的交互功能。

- 可以设置某段文字的字体名称，颜色，下滑线等信息。

```Objective-C
- (void)addAttribute:(NSString *)name value:(id)value range:(NSRange)range;
```

- 移除之前设置的字体属性值。

```Objective-C
- (void)addAttributes:(NSDictionary *)attrs range:(NSRange)range;
```

- 存储某段文字包含的信息（包括字体属性或其它，也可以存储一些自定义的信息）
 
```Objective-C
- (void)addAttributes:(NSDictionary *)attrs range:(NSRange)range;
```

- 通过location来获取某段文字中之前存储的信息NSDictionary

```
- (NSDictionary *)attributesAtIndex:(NSUInteger)location effectiveRange:(NSRangePointer)range;
```

```
//设置字体
   CTFontRef aFont = CTFontCreateWithName((CFStringRef)textFont.fontName, textFont.pointSize, NULL);
   if (!aFont) return;
   CTFontRef newFont = CTFontCreateCopyWithSymbolicTraits(aFont, 0.0, NULL, kCTFontItalicTrait, kCTFontBoldTrait);    //将默认黑体字设置为其它字体 
   [self removeAttribute:(NSString*)kCTFontAttributeName range:textRange];
   [self addAttribute:(NSString*)kCTFontAttributeName value:(id)newFont range:textRange];
   CFRelease(aFont);
   CFRelease(newFont);
```

#####实现点击文字触发事件
1. 根据touch事件获取点point
2. textFrame通过CTFrameGetLineOrigins获取所有line的原点
3. 比较point和line原点的y值获取点击处于哪个line
4. line、point通过CTFrameGetStringIndexForPosition获取到点击字符在整段文字中的index
5. NSAttributedString 通过index 用方法－(NSDictionary *)attributesAtIndex:(NSUInteger)location effectiveRange:(NSRangePointer)range  可以获取到点击到的NSAttributedString中存储的NSDictionary
6. 通过NSDictionary中存储的信息判断点击的哪种文字类型分别处理

---
* 设置字体属性

```
//设置字体颜色
   [self removeAttribute:(NSString*)kCTForegroundColorAttributeName range:textRange];
   [self addAttribute:(NSString*)kCTForegroundColorAttributeName value:(id)textColor.CGColor range:textRange];
   
   //设置对齐 换行
   CTTextAlignment coreTextAlign = kCTLeftTextAlignment;
   CTLineBreakMode coreTextLBMode = kCTLineBreakByCharWrapping;
   CTParagraphStyleSetting paraStyles[2] =
   {
       {.spec = kCTParagraphStyleSpecifierAlignment, .valueSize = sizeof(CTTextAlignment), .value = (const void*)&coreTextAlign},
       {.spec = kCTParagraphStyleSpecifierLineBreakMode, .valueSize = sizeof(CTLineBreakMode), .value = (const void*)&coreTextLBMode},
   };
   CTParagraphStyleRef aStyle = CTParagraphStyleCreate(paraStyles, 2);
   [self removeAttribute:(NSString*)kCTParagraphStyleAttributeName range:textRange];
   [self addAttribute:(NSString*)kCTParagraphStyleAttributeName value:(id)aStyle range:textRange];
   CFRelease(aStyle);
```

- 实现图文混排的方法

```
CTFrameRef  textFrame     // coreText 的 frame

     CTLineRef      line             //  coreText 的 line

     CTRunRef      run             //  line  中的部分文字

     相关方法：

   
   CFArrayRef CTFrameGetLines    (CTFrameRef frame )      //获取包含CTLineRef的数组

   void CTFrameGetLineOrigins(
   CTFrameRef frame,
   CFRange range,
   CGPoint origins[] )  //获取所有CTLineRef的原点

 CFRange CTLineGetStringRange  (CTLineRef line )    //获取line中文字在整段文字中的Range

 CFArrayRef CTLineGetGlyphRuns  (CTLineRef line )    //获取line中包含所有run的数组

 CFRange CTRunGetStringRange  (CTRunRef run )     //获取run在整段文字中的Range

 CFIndex CTLineGetStringIndexForPosition(
   CTLineRef line,
   CGPoint position )   //获取点击处position文字在整段文字中的index

   CGFloat CTLineGetOffsetForStringIndex(
   CTLineRef line,
   CFIndex charIndex,
   CGFloat* secondaryOffset ) //获取整段文字中charIndex位置的字符相对line的原点的x值

  主要步骤：

       1）计算并存储文字中保含的所有表情文字及其Range

       2）替换表情文字为指定宽度的NSAttributedString

           CTRunDelegateCallbacks callbacks;
   callbacks.version = kCTRunDelegateVersion1;
   callbacks.getAscent = ascentCallback;
   callbacks.getDescent = descentCallback;
   callbacks.getWidth = widthCallback;
   callbacks.dealloc = deallocCallback;
   
   CTRunDelegateRef runDelegate = CTRunDelegateCreate(&callbacks, NULL);
   NSDictionary *attrDictionaryDelegate = [NSDictionary dictionaryWithObjectsAndKeys:
                                           (id)runDelegate, (NSString*)kCTRunDelegateAttributeName,
                                           [UIColor clearColor].CGColor,(NSString*)kCTForegroundColorAttributeName,
                                           nil];
   
   NSAttributedString *faceAttributedString = [[NSAttributedString alloc] initWithString:@"*" attributes:attrDictionaryDelegate];
   
   [weiBoText replaceCharactersInRange:faceRange withAttributedString:faceAttributedString];
   [faceAttributedString release]; 
```
- 根据保存的表情文字的Range计算表情图片的Frame

textFrame 通过CTFrameGetLines 获取所有line的数组 lineArray
遍历lineArray中的line通过CTLineGetGlyphRuns获取line中包含run的数组 runArray
遍历runArray中的run 通过CTRunGetStringRange获取run的Range
判断表情文字的location是否在run的Range
通过CTLineGetOffsetForStringIndex获取x的值 y的值为line原点的值













