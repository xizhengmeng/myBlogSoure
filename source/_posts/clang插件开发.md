---
title: clang插件开发
date: 2017-09-05 10:41:44
tags:
- clang插件
---

## 资源准备&&编译源文件
我们需要llvm和clang的源文件，而且这些文件都存在于github上，我们如果直接从github下载会很慢，所以第一步是制作国内源。

<!--more-->

- 制作国内源`http://jingyan.baidu.com/article/f79b7cb367e72e9145023e40.html`

还有就是不要把这些源文件放到一些需要权限执行的文件里，这样在编译过程中你会发现各种问题。

- 下载源码的时候注意点就是要对照你的xcode的版本:`https://trac.macports.org/wiki/XcodeVersionInfo`
- 然后找到对应的clang版本`https://opensource.apple.com/source/clang/clang-800.0.42.1/src/configure.auto.html`

经过对比我们发现我们需要的是39版本

比如现在我们在一个叫做GitHub的文件夹下遍建立了一个叫做llvm的文件
按照顺序执行以后代码：

```
cd GitHub
sudo mkdir llvm
sudo chown `whoami` llvm
cd llvm
export LLVM_HOME=`pwd`

git clone -b release_39 https://git.coding.net/hanshenghui/llvm.git llvm
git clone -b release_39 https://git.coding.net/hanshenghui/clangnew.git llvm/tools/clang
git clone -b release_39 https://git.coding.net/hanshenghui/clang-tools-extra.git llvm/tools/clang/tools/extra
git clone -b release_39 https://git.coding.net/hanshenghui/compiler-rt.git llvm/projects/compiler-rt

mkdir llvm_build
cd llvm_build
cmake -G Xcode ../llvm -DCMAKE_BUILD_TYPE:STRING=MinSizeRel

```
最后一句`cmake -G Xcode`是关键，用这种方式cmake的话，我们就可以用Xcode来编译后边的工程
执行完毕之后你会发现，在llvm_build这个文件夹下边你会发现有一个LLVM.xcodeproj的文件，有了这个我们可以像iOS开发样去编译任何一个库了

## 编写插件代码
- 进入文件`/llvm/tools/clang/examples`在里面新建一个目录如MyPlugin
- 然后修改example目录的CMakeLists.txt文件，添加一项：
```
add_subdirectory(MyPlugin)
```
- 然后进入创建的MyPlugin目录，生成三个文件，分别是：
```
CodingStyleUtil.hpp
MyPlugin.cpp
CMakeLists.txt
```

CMakeLists.txt中的内容：
```
add_llvm_loadable_module(MyPlugin MyPlugin.cpp PLUGIN_TOOL clang)

if(LLVM_ENABLE_PLUGINS AND (WIN32 OR CYGWIN))
  target_link_libraries(MyPlugin ${cmake_2_8_12_PRIVATE}
    clangAST
    clangBasic
    clangFrontend
    LLVMSupport
    )
endif()
```
CodingStyleUtil.hpp
主要是一些处理字符串的函数

MyPlugin.cpp
这个文件是关键所在，当我们编译插件的时候，主要就是这里的代码在起作用，先来个简化版的

```

#include "clang/Frontend/FrontendPluginRegistry.h"
#include "clang/AST/AST.h"
#include "clang/AST/ASTConsumer.h"
#include "clang/Frontend/CompilerInstance.h"
#include "clang/AST/RecursiveASTVisitor.h"
#include "CodingStyleUtil.hpp"
#include <fstream>

using namespace clang;
using namespace std;
using namespace llvm;

string gSrcRootPath;
static string kClassInterfPrefix = "JR";
static int kMethodParamMaxLen = 15;
//static int kMethodParamMaxParamsSingleLine = 3;
static int kMethodBodyMaxLines = 500;

namespace MyPlugin
 {

class MyPluginVisitor : public RecursiveASTVisitor<MyPluginVisitor>//这里我们要声明一个class，这个class是继承自RecursiveASTVisitor的，可以随便取名字，尖括号里边就是这个visitor的名字
    {
    private:
        CompilerInstance &Instance;
        ASTContext *Context;
        string objcClsImpl;
        bool objcIsInstanceMethod;
        string objcSelector;
        string objcMethodSrcCode;
        
    public:
        
        void setASTContext (ASTContext &context)//这里定义了一个能方法，用来便捷的对context进行赋值操作
        {
            this -> Context = &context;
        }
        
        MyPluginVisitor (CompilerInstance &Instance)
        :Instance(Instance)
        {
            
        }
        
        bool VisitDecl(Decl *decl) {//所有的声明分析都需要重载这个方法
        }
        
    };

    class MyPluginConsumer : public ASTConsumer
    {
        CompilerInstance &Instance;
        std::set<std::string> ParsedTemplates;
    public:
        MyPluginConsumer(CompilerInstance &Instance,
                         std::set<std::string> ParsedTemplates)
        : Instance(Instance), ParsedTemplates(ParsedTemplates), visitor(Instance) {}
        
        bool HandleTopLevelDecl(DeclGroupRef DG) override
        {
            return true;
        }
        
        void HandleTranslationUnit(ASTContext& context) override
        {
            visitor.setASTContext(context);
            visitor.TraverseDecl(context.getTranslationUnitDecl());
        }
    private:
        MyPluginVisitor visitor;
    };
    //这里是所有处理逻辑的入口，在这里调用了consumer
    class MyPluginASTAction : public PluginASTAction
    {
        std::set<std::string> ParsedTemplates;
    protected:
        std::unique_ptr<ASTConsumer> CreateASTConsumer(CompilerInstance &CI,
                                                       llvm::StringRef) override
        {
            return llvm::make_unique<MyPluginConsumer>(CI, ParsedTemplates);
        }
        
        bool ParseArgs(const CompilerInstance &CI,
                       const std::vector<std::string> &args) override {
            return true;
        }
    };

```
现在先回到源码根目录，使用同样的cmake语句来更新Xcode项目，更新完成后原来的项目会多出一个叫MyPlugin的插件项目，然后对这个插件项目进行编译。编译成功后会在Debug/lib目录中多出一个名字叫做MyPlugin.dylib文件

只有这个plugin文件是不够的，我们还需要一个对应的clang和clang++，那么这个文件是哪里来的呢，答案就是我们自己编译的，这个插件和clang版本必须是对应的，否则在运行工程的时候就会说symbol不存在等错误

## 安装调试插件
打开要使用插件的Xcode项目，在build settings一栏中对Other C Flags一项进行编辑，调整为：
```
-Xclang -load -Xclang /Users/han/GitHub/llvm/llvm_build/Debug/lib/MyPlugin.dylib -Xclang -add-plugin -Xclang MyPlugin
```
>注：最后一项-Xclang MyPlugin中的MyPlugin为插件名字，一定要是自己设置的插件名称，否则无法调用插件

这个时候运行你会发现报错，error:unable to load plugin

为了解决这个问题需要调整Xcode中使用的Clang编译器，将默认的编译器改为我们自己编译出来的编译器。具体的方法是在build settings中再添加两项自定义项：
```
CC = /Users/han/GitHub/llvm/llvm_build/Debug/bin/clang
CXX = "/Users/han/GitHub/llvm/llvm_build/Debug/bin/clang+
```

## clang插件作用范围
