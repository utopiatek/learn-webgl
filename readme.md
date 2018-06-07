# LEARN-WEBGL
WebAssembly、Emscripten、WebGL

## 2018-06-07

### WebAssembly
* [官网](https://webassembly.org/)
* [中文官网](https://wasm-cn.org/)
* [中文MDN](https://developer.mozilla.org/zh-CN/docs/WebAssembly)
* [在线开发工具](https://webassembly.studio/)

### Emscripten
* 下载安装GIT
> 下载，运行[GIT](https://git-scm.com/download/win)安装程序
* 下载安装Python
> 下载安装Python2.7

> 拷贝python\2.7.5.3_64bit文件夹到Emsdk文件夹下

> Emscripten SDK需要使用python2.7进行安装
* 下载安装Emscripten SDK
> 下载，解压[EMSDK](https://github.com/juj/emsdk)程序包到Emsdk文件夹下，当前版本1.38.5

> 以管理员身份运行控制台，切换到Emsdk文件夹

> 执行命令`emsdk install latest --vs2017`

> 执行命令`emsdk activate latest`

> 注意：在使用EMSDK前需要调用emsdk_env命令来设置环境变量

> EMSDK提供命令提示符“emcmdprompt.bat”，启动后自动设置环境变量
* 安装支持WASM的浏览器
> 下载安装[Chrome](https://www.google.cn/intl/zh-CN/chrome/browser/thankyou.html?standalone=1&statcb=1&installdataindex=defaultbrowser#)

### WASM样例程序
* 编辑math.c代码
```
int square (int x)
{
  return x * x;
}
```
* 编译math.c代码为math.wasm代码
> 启动“emcmdprompt.bat”

> 执行命令：`emcc math.c -Os -s BINARYEN_ASYNC_COMPILATION=0 -s WASM=1 -s SIDE_MODULE=1 -o math.wasm`
* 编辑loader.js代码
```
function Config() {
  var pImports = {
    env: {
      memoryBase: 0,
      tableBase: 0,
      memory: new WebAssembly.Memory({ initial: 256, maximum: 1024 }),
      table: new WebAssembly.Table({ initial: 2, maximum: 8, element: 'anyfunc' }),
      DYNAMICTOP_PTR: 0,
      tempDoublePtr: 0,
      ABORT: 0,
      abortStackOverflow: function (allocSize) { },
      nullFunc_X: function () { },
      abort: function () { }
    },
    global: {
      NaN: 0,
      Infinity: 0
    }
  };

  pImports.env.table.set(0, null);
  pImports.env.table.set(0, null);

  return pImports;
}

function LoadWASM(pFileName, pImports = {}) {
  return fetch(pFileName)
    .then(response => response.arrayBuffer())
    .then(buffer => WebAssembly.compile(buffer))
    .then(bytes => WebAssembly.instantiate(bytes, pImports))
    .catch(console.error);
}
```
* 在HTML在中载入WASM代码并调用函数
```
<script src="loader.js"></script>
<script>
    LoadWASM('math.wasm', Config())
      .then(instance => {
        const square = instance.exports._square;
        alert('2 * 2 = '+ square(2));
      })
</script>
```

### Makefile
* 编辑Makefile.txt文件
```
CC=emcc
CPPFLAGS=-g -std=gnu++11 -Wall -Wno-gnu-zero-variadic-macro-arguments -pedantic
EMFLAGS = -s BINARYEN_ASYNC_COMPILATION=0 -s WASM=1 -s SIDE_MODULE=1

DIR_INC = -I./
DIR_SRC = ./

sources=math.cpp
objects=$(sources:.cpp=.o)
dependence=$(sources:.cpp=.d)

all: $(objects)
	$(CC) $(CPPFLAGS) $(EMFLAGS) $(objects) -o math.wasm

#include $(dependence)

{$(DIR_SRC)}.cpp.o:
	$(CC) $(CPPFLAGS) $< $(DIR_INC) -o $@

%.d : %.cpp
	@set -e; \
	rm -f $@; \
	$(CC) -M $(CPPFLAGS) $(EMFLAGS) $< > $@.$$$$; \
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	rm -f $@.$$$$

.PHONY : clean

clean :
	-rm all $(dependence) $(objects)
```
* 编译生成
> 启动“emcmdprompt.bat”

> 执行命令：`emmake nmake -f Makefile.txt`

### WebGL
* [中文MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial)
* 依赖库：-lGLESv2 -lEGL