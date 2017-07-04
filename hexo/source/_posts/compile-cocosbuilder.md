---
title: 编译CocosBuilder 3.5 踩坑实录
date: 2017-07-04 17:13:18
tags: cocosbuilder
category: cocos2dx
---

因为cocos studio实在太过难用，简直不能愉快地码界面，所以不得已计划换到cocosbuilder下（虽然也很久没更新了，但是至少有源代码）。

一个很艰难的过程就这样开始了。

1.踩坑开始

按照readme里的内容clone下来，初始化子模块，然后编译。毫无疑问地失败了。所以踩着前人的足迹开始踩坑。

2.所以开始追随[前人的足迹](https://github.com/cocos2d/CocosBuilder/issues/434)

```
diff --git a/.gitmodules b/.gitmodules
index 70efbcf..e61c729 100644
--- a/.gitmodules
+++ b/.gitmodules
@@ -1,12 +1,12 @@
 [submodule "CocosBuilder/libs/cocos2d-iphone"]
        path = CocosBuilder/libs/cocos2d-iphone
-       url = https://github.com/cocos2d/cocos2d-iphone.git
+       url = git://github.com/cocos2d/cocos2d-iphone-classic.git
 [submodule "CocosBuilder/libs/ssziparchive"]
        path = CocosBuilder/libs/ssziparchive
-       url = https://github.com/vlidholt/ssziparchive
+       url = git://github.com/vlidholt/ssziparchive
 [submodule "CocosBuilder/libs/Fragaria"]
        path = CocosBuilder/libs/Fragaria
-       url = https://github.com/vlidholt/Fragaria.git
+       url = git://github.com/vlidholt/Fragaria.git
 [submodule "Examples/CocosBuilderExample/libs/CCBReader"]
        path = Examples/CocosBuilderExample/libs/CCBReader
        url = git://github.com/cocos2d/CCBReader.git
```

改好之后，再编译，再次失败。

于是，继续尝试。

```
Okay just worked through this.

Make sure you pull branch v3.5.0.

Step 1: Follow patrickhoey's Step 1 and modify .gitmodules

Step 2: Make sure you're in the root directory of the CocosBuilder project and run:
git submodule update --init --recursive

This will error on cocos-iphone-classic.

Step 3: cd CocosBuilder/libs/cocos2d-iphone

Step 4: git checkout 5e8fdee5cdc4450ad32f5a8579ee6a0fdcbb1cfa

Step 5: Get a copy of this project in an entirely separate directory. There have been files removed at some point that need to be re-added.
https://github.com/cocos2d/cocos2d-iphone-classic

Step 6: Copy the following files from the cocos2d-iphone-classic project:
cocos-iphone-classic/cocos2d/ccFPSImages.m --> CocosBuilder/libs/cocos2d-iphone/cocos2d/ccFPSImages.m
cocos-iphone-classic/cocos2d/Support/uthash.h --> CocosBuilder/libs/cocos2d-iphone/cocos2d/Support/uthash.h
cocos-iphone-classic/cocos2d/Support/utlist.h --> CocosBuilder/libs/cocos2d-iphone/cocos2d/Support/utlist.h

This worked for me.
```
按照这个描述，尝试，发现也不行。
不过提供了一个很好的思路。

于是经过多方尝试，终于找到的办法。


1. 修改`.gitmodule`把cocos2d-iphone改成cocos2d-iphone-classic

2. 进入CocosBuilder/libs/cocos2d-iphone目录，执行`git checkout 5e8fdee5cdc4450ad32f5a8579ee6a0fdcbb1cfa`
3. 随后，按照

```
Step 6: Copy the following files from the cocos2d-iphone-classic project:
cocos-iphone-classic/cocos2d/ccFPSImages.m --> CocosBuilder/libs/cocos2d-iphone/cocos2d/ccFPSImages.m
cocos-iphone-classic/cocos2d/Support/uthash.h --> CocosBuilder/libs/cocos2d-iphone/cocos2d/Support/uthash.h
cocos-iphone-classic/cocos2d/Support/utlist.h --> CocosBuilder/libs/cocos2d-iphone/cocos2d/Support/utlist.h
```
凑齐这三个丢失的文件。

###编译，通过！！！！！！！！！！！！！！


### 运行！！






### 崩溃！！

继续寻找[前辈的足迹](https://stackoverflow.com/questions/30864055/coco2d-2-1-and-xcode-7-ios-9-crash-ccshader)。

修改CCGLProgram.m，

```
#define EXTENSION_STRING "#extension GL_OES_standard_derivatives : enable"
static NSString * g_extensionStr = @EXTENSION_STRING;

- (BOOL)compileShader:(GLuint *)shader type:(GLenum)type byteArray:(const GLchar *)source
{
    GLint status;

    if (!source)
        return NO;

    // BEGIN workaround for Xcode 7 bug
    BOOL hasExtension = NO;
    NSString *sourceStr = [NSString stringWithUTF8String:source];
    if([sourceStr rangeOfString:g_extensionStr].location != NSNotFound) {
        hasExtension = YES;
        NSArray *strs = [sourceStr componentsSeparatedByString:g_extensionStr];
        assert(strs.count == 2);
        sourceStr = [strs componentsJoinedByString:@"\n"];
        source = (GLchar *)[sourceStr UTF8String];
    }

    const GLchar *sources[] = {
        (hasExtension ? EXTENSION_STRING "\n" : ""),
    #ifdef __CC_PLATFORM_IOS
        (type == GL_VERTEX_SHADER ? "precision highp float;\n" : "precision mediump float;\n"),
    #endif
        "uniform mat4 CC_PMatrix;\n"
        "uniform mat4 CC_MVMatrix;\n"
        "uniform mat4 CC_MVPMatrix;\n"
        "uniform vec4 CC_Time;\n"
        "uniform vec4 CC_SinTime;\n"
        "uniform vec4 CC_CosTime;\n"
        "uniform vec4 CC_Random01;\n"
        "//CC INCLUDES END\n\n",
        source,
    };
    // END workaround for Xcode 7 bug
```

### 运行，成功！！！！！！！