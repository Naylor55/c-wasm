# c-wasm
将C代码编译为Webassembly，提供给js调用



# WebAssembly简介

* 一种运行在浏览器中的新型编程语言
* 设计目的：为低级/底层编程语言（c、c++、rust等）提供一个高效的编译目标
* 意义：让客户端app可以在浏览器中运行，但是性能和速度肯定没有原生的好
* 互操作性：通过WebAssemblyJavascriptAPI，可以让多种编程语言协同工作，wasm屏蔽了多语言交互的复杂性
* coding：不为手写代码而生，WebAssembly和其它编程语言不同，通过基本数据类型+多态等特性提供计算和存储服务，它的诞生目的就是为了解决c代码无法在浏览器中运行的问题。当然作为编程语言它本身可以通过AssemblyScript编写代码并编译为二进制的wasm文件
* 同为浏览器运行的语言，不为替代Javascript，而是要与其协同工作

# WebAssembly工作流程概述


* C 代码使用 Emssripten 工具编译为 wasm 后缀的二进制文件，同时可以生成访问wasm的js胶水代码和html代码
* wasm后缀的二进制格式文件的文本表示方式为后缀为wat格式的文本文件，方便在编辑器和浏览器开发者工具中查看
* 可以使用wabt工具将wat格式的文本文件直接打包成wasm的二进制文件
* 使用WebAssemblyJavascriptAPI发起对wasm的调用
    * 编写胶水代码
    * 使用fetch/xhr获取wasm
    * 借助胶水代码访问wasm中的函数



# 安装 Emscripten

Emscripten 是一个打包工具，用来将c代码编译成wasm，官方提供的安装方式是使用一个叫做emsdk的工具，此工具负责下载相关依赖并执行安装，需要科学上网。

git clone https://github.com/juj/emsdk.git 

cd emsdk

~~~

//下载最新版本

PS D:\Application\emsdk> .\emsdk.bat install latest
Resolving SDK alias 'latest' to '3.1.54'
Resolving SDK version '3.1.54' to 'sdk-releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'
Installing SDK 'sdk-releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'..
Skipped installing node-16.20.0-64bit, already installed.
Skipped installing python-3.9.2-nuget-64bit, already installed.
Skipped installing java-8.152-64bit, already installed.
Installing tool 'releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'..
Downloading: D:/Application/emsdk/downloads/aa1588cd28c250a60457b5ed342557c762f416e3-wasm-binaries.zip from https://storage.googleapis.com/webassembly/emscripten-releases-builds/win/aa1588cd28c250a60457b5ed342557c762f416e3/wasm-binaries.zip, 459538273 Bytes
Unpacking 'D:/Application/emsdk/downloads/aa1588cd28c250a60457b5ed342557c762f416e3-wasm-binaries.zip' to 'D:/Application/emsdk/upstream'
Done installing tool 'releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'.
Done installing SDK 'sdk-releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'.

//安装并激活最新版本

PS D:\Application\emsdk> .\emsdk activate --system latest
Resolving SDK alias 'latest' to '3.1.54'
Resolving SDK version '3.1.54' to 'sdk-releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit'
Registering active Emscripten environment permanently


Setting the following tools as active:
   node-16.20.0-64bit
   python-3.9.2-nuget-64bit
   java-8.152-64bit
   releases-aa1588cd28c250a60457b5ed342557c762f416e3-64bit


Adding directories to PATH:
PATH += D:\Application\emsdk
PATH += D:\Application\emsdk\upstream\emscripten


Setting environment variables:
PATH = D:\Application\emsdk;D:\Application\emsdk\upstream\emscripten;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;D:\Application\openjdk-11+28_windows-x64_bin\jdk-11\bin;C:\Program Files\TortoiseSVN\bin;%JAVA_HOME\lib%;C:\Program Files\Microsoft SQL Server\140\Tools\Binn\;D:\Application\nvm;C:\Program Files\nodejs;D:\Application\nvm-noinstall\;C:\Users\cml\AppData\Local\Programs\Python\Python39\;D:\Application\gradle-6.9.1\bin\;D:\IntellijIdeal_maven_repository;D:\SoftData\nodejs\npm\npm_modules;C:\Program Files\Microsoft SQL Server\150\Tools\Binn\;D:\Application\go1.17.6\bin;C:\Program Files\nodejs\;D:\Application\Git\Git\cmd;%ERLANG_HOME%\bin;%RABBITMQ_SERVER%\sbin;C:\Program Files\dotnet\;D:\Application\VS2022\VC\Tools\MSVC\14.34.31933\bin\Hostx86\x64\;C:\Program Files\Docker\Docker\resources\bin;C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\;D:\Program Files\PuTTY\;D:\Program Files\Microsoft Visual Studio\2022\Professional\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin;C:\ProgramData\chocolatey\bin;
EMSDK = D:/Application/emsdk
EMSDK_NODE = D:\Application\emsdk\node\16.20.0_64bit\bin\node.exe
EMSDK_PYTHON = D:\Application\emsdk\python\3.9.2-nuget_64bit\python.exe
JAVA_HOME = D:\Application\emsdk\java\8.152_64bit
error: failed to set the environment variable 'PATH'! Setting environment variables permanently requires administrator access. Please rerun this command with administrative privileges. This can be done for example by holding down the Ctrl and Shift keys while opening a command prompt in start menu.

//设置环境变量

PS D:\Application\emsdk> .\emsdk_env.bat
Setting up EMSDK environment (suppress these messages with EMSDK_QUIET=1)
Setting environment variables:
Clearing existing environment variable: EMSDK_PY

~~~

* --system：全局安装
* 安装完毕需要看看环境变量是否已经设置
    * D:\xxx\emsdk
    * D:\xxx\emsdk\upstream\emscripten
* 如果编译安装总失败，可以尝试直接获取一个完整的编译好了的包：https://github.com/Naylor55/full-emsdk
    


# Hello示例

找一个文件夹，新建 hello.c 文件夹，填入如下代码：


~~~

#include <stdio.h>


int main(int argc, char ** argv) {
  printf("Hello World\n");
}

~~~

然后执行 emcc  命令进行打包


~~~

emcc hello.c -s WASM=1 -o hello.html

~~~

* -s WASM=1 ： 指定我们想要的 wasm 输出形式。如果我们不指定这个选项，Emscripten 默认将只会生成 asm.js。
* -o hello.html ： 指定这个选项将会生成 HTML 页面来运行我们的代码，并且会生成 wasm 模块，以及编译和实例化 wasm 模块所需要的“胶水”js 代码，这样我们就可以直接在 web 环境中使用了。


打包完毕之后，会生成两个新的文件：
* hello.js：胶水代码，封装了上述c代码的调用方法，其它html可以直接引用此js
* hello.html：示例代码，写了一个通过js调用c代码中方法的例子
* hello.wasm：二进制的 wasm 模块，可以直接被其它js代码装载后调用



运行上述hello.html之后如果在控制台看到了 Hello World  的输出，说明成功了，其背后工作流程：
* 首先hello.js装载hello.wasm，并封装为了一个名叫 Module 的对象
* 在hello.html 中引入了hello.js，这样就可以访问Module对象了


# 调用C代码中的方法

默认情况下Emscripten仅会生成c代码中的main方法，其其它代码将视为无用代码，这在实际应用中肯定是不被允许的。

在c代码中的函数名前面加 EMSCRIPTEN_KEEPALIVE 这个关键字可以解决这种问题，需要导入emscripten.h 头文件。


新建一个文件夹，创建 hello3.c 文件，并填入如下代码：

~~~

#include <stdio.h>
#include <emscripten/emscripten.h>


int main(int argc, char ** argv) {
    printf("Hello World\n");
}


#ifdef __cplusplus
extern "C" {
#endif


int EMSCRIPTEN_KEEPALIVE myFunction(int argc, char ** argv) {
  printf("我的函数已被调用\n");
  return 555;
}


#ifdef __cplusplus
}
#endif

~~~

在文件夹中新建一个文件夹 html_template 并在里面创建一个shell_minimal.html 文件，填入如下代码：

~~~

<!doctype html>
<html lang="en-us">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Emscripten-Generated Code</title>
    <style>
      .emscripten { padding-right: 0; margin-left: auto; margin-right: auto; display: block; }
      textarea.emscripten { font-family: monospace; width: 80%; }
      div.emscripten { text-align: center; }
      div.emscripten_border { border: 1px solid black; }
      /* the canvas *must not* have any border or padding, or mouse coords will be wrong */
      canvas.emscripten { border: 0px none; background-color: black; }


      .spinner {
        height: 50px;
        width: 50px;
        margin: 0px auto;
        -webkit-animation: rotation .8s linear infinite;
        -moz-animation: rotation .8s linear infinite;
        -o-animation: rotation .8s linear infinite;
        animation: rotation 0.8s linear infinite;
        border-left: 10px solid rgb(0,150,240);
        border-right: 10px solid rgb(0,150,240);
        border-bottom: 10px solid rgb(0,150,240);
        border-top: 10px solid rgb(100,0,200);
        border-radius: 100%;
        background-color: rgb(200,100,250);
      }
      @-webkit-keyframes rotation {
        from {-webkit-transform: rotate(0deg);}
        to {-webkit-transform: rotate(360deg);}
      }
      @-moz-keyframes rotation {
        from {-moz-transform: rotate(0deg);}
        to {-moz-transform: rotate(360deg);}
      }
      @-o-keyframes rotation {
        from {-o-transform: rotate(0deg);}
        to {-o-transform: rotate(360deg);}
      }
      @keyframes rotation {
        from {transform: rotate(0deg);}
        to {transform: rotate(360deg);}
      }


    </style>
  </head>
  <body>
    <hr/>
    <figure style="overflow:visible;" id="spinner"><div class="spinner"></div><center style="margin-top:0.5em"><strong>emscripten</strong></center></figure>
    <div class="emscripten" id="status">Downloading...</div>
    <div class="emscripten">
      <progress value="0" max="100" id="progress" hidden=1></progress>  
    </div>
    <div class="emscripten_border">
      <canvas class="emscripten" id="canvas" oncontextmenu="event.preventDefault()" tabindex=-1></canvas>
    </div>
    <hr/>
    <div class="emscripten">
      <input type="checkbox" id="resize">Resize canvas
      <input type="checkbox" id="pointerLock" checked>Lock/hide mouse pointer
      &nbsp;&nbsp;&nbsp;
      <input type="button" value="Fullscreen" onclick="Module.requestFullscreen(document.getElementById('pointerLock').checked,
                                                                                document.getElementById('resize').checked)">
    </div>
   
    <hr/>
    <textarea class="emscripten" id="output" rows="8"></textarea>
    <hr>
    <script type='text/javascript'>
      var statusElement = document.getElementById('status');
      var progressElement = document.getElementById('progress');
      var spinnerElement = document.getElementById('spinner');


      var Module = {
        print: (function() {
          var element = document.getElementById('output');
          if (element) element.value = ''; // clear browser cache
          return (...args) => {
            var text = args.join(' ');
            // These replacements are necessary if you render to raw HTML
            //text = text.replace(/&/g, "&amp;");
            //text = text.replace(/</g, "&lt;");
            //text = text.replace(/>/g, "&gt;");
            //text = text.replace('\n', '
', 'g');
            console.log(text);
            if (element) {
              element.value += text + "\n";
              element.scrollTop = element.scrollHeight; // focus on bottom
            }
          };
        })(),
        canvas: (() => {
          var canvas = document.getElementById('canvas');


          // As a default initial behavior, pop up an alert when webgl context is lost. To make your
          // application robust, you may want to override this behavior before shipping!
          // See http://www.khronos.org/registry/webgl/specs/latest/1.0/#5.15.2
          canvas.addEventListener("webglcontextlost", (e) => { alert('WebGL context lost. You will need to reload the page.'); e.preventDefault(); }, false);


          return canvas;
        })(),
        setStatus: (text) => {
          if (!Module.setStatus.last) Module.setStatus.last = { time: Date.now(), text: '' };
          if (text === Module.setStatus.last.text) return;
          var m = text.match(/([^(]+)\((\d+(\.\d+)?)\/(\d+)\)/);
          var now = Date.now();
          if (m && now - Module.setStatus.last.time < 30) return; // if this is a progress update, skip it if too soon
          Module.setStatus.last.time = now;
          Module.setStatus.last.text = text;
          if (m) {
            text = m[1];
            progressElement.value = parseInt(m[2])*100;
            progressElement.max = parseInt(m[4])*100;
            progressElement.hidden = false;
            spinnerElement.hidden = false;
          } else {
            progressElement.value = null;
            progressElement.max = null;
            progressElement.hidden = true;
            if (!text) spinnerElement.hidden = true;
          }
          statusElement.innerHTML = text;
        },
        totalDependencies: 0,
        monitorRunDependencies: (left) => {
          this.totalDependencies = Math.max(this.totalDependencies, left);
          Module.setStatus(left ? 'Preparing... (' + (this.totalDependencies-left) + '/' + this.totalDependencies + ')' : 'All downloads complete.');
        }
      };
      Module.setStatus('Downloading...');
      window.onerror = () => {
        Module.setStatus('Exception thrown, see JavaScript console');
        spinnerElement.style.display = 'none';
        Module.setStatus = (text) => {
          if (text) console.error('[post-exception status] ' + text);
        };
      };
    </script>
    {{{ SCRIPT }}}
  </body>
</html>

~~~

然后执行emcc命令进行打包：


~~~

emcc hello3.c   -s WASM=1 -o hello3.html       --shell-file html_template/shell_minimal.html  -s   "EXPORTED_RUNTIME_METHODS=['ccall']"   

~~~

* --shell-file html_template/shell_minimal.html：使用html模板来生成hello3.html
* -s   "EXPORTED_RUNTIME_METHODS=['ccall']"：导出运行时方法 ccall，js代码通过ccall访问c代码中的函数


打包完毕之后，我们需要对hello3.html稍作修改才能演示调用c代码函数的功能：

在dom中添加一个按钮 

~~~

<button class="mybutton">运行我的函数</button>

~~~

编写按钮的点击事件

~~~

document.querySelector(".mybutton").addEventListener("click", function () {
  alert("检查控制台");
  var result = Module.ccall(
    "myFunction", // name of C function
    null, // return type
    null, // argument types
    null,
  ); // arguments
});

~~~

* Module.ccall：通过ccall 方法调用c代码中的函数


此时运行hello3.html，首先将在控制台看到 Hello,World ，而后点击"运行我的函数"按钮，将看到控制台打印了“我的函数已被调用”





# 代码

https://github.com/Naylor55/c-wasm

# 引用

https://developer.mozilla.org/zh-CN/docs/WebAssembly/C_to_wasm
https://developer.mozilla.org/zh-CN/docs/WebAssembly/Using_the_JavaScript_API
https://github.com/Naylor55/full-emsdk
https://github.com/emscripten-core/emsdk






---