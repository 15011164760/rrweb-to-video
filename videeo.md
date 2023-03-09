# rrweb实现录像功能
## 安装ffmpeg 
```
如何安装ffmpeg（windows）
进入官网页面：https://ffmpeg.org/download.html
找到git下面的第一个链接进行下载

下载完毕，解压到需要的路径，并复制bin路径。如：[C://ffmpeg/bin](file:///C://ffmpeg/bin)，这是程序所在的路径。可以从cmd里面粘取路径（一定得是自己解压以后的路径）

我的电脑 -> 高级系统设置 -> 环境变量 -> Path -> 新建 -> 输入咱们的FFmpeg路径 -> 一路确定再依次关闭

测试一下是否配置成功：cmd窗口输入ffmpeg命令，如有信息则说明成功
具体不知道看这个文章：https://www.jianshu.com/p/9b5a5085e781
```

## 安装rrvideo 
```
npm i -g rrvideo 以安装 rrvideo CLI
```
```
我们使用rrweb录屏后是将数据传送给服务端，然后在服务端生成一个json文件，下载这个json文件，运行如下命令：

rrvideo --input PATH_TO_YOUR_RRWEB_EVENTS_FILE --output OUTPUT_PATH

PATH_TO_YOUR_RRWEB_EVENTS_FILE: 是要转码的json文件的路径
OUTPUT_PATH：是转码后生成mp4文件的路径

注意：OUTPUT_PATH路径后要加 \xxx.mp4 才能生成mp4格式的视频

```
1. 例子代码
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>rrweb demo web site</title>
    <script crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/rrweb@latest/dist/rrweb.min.js"></script>
    <script crossorigin="anonymous"
        src="https://cdn.jsdelivr.net/npm/rrweb@latest/dist/record/rrweb-record.min.js"></script>
    <link rel="stylesheet" crossorigin="anonymous"
        href="https://cdn.jsdelivr.net/npm/rrweb-player@latest/dist/style.css" />
    <script crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/rrweb-player@latest/dist/index.js"></script>
</head>

<body>
    <h1 class="some-title">this is some title for test</h1>
    <input type="button" value="开始录制" onclick="record()" />
    <br />
    <input type="button" value="结束录制" onclick="replay()" />
    <img src="http://localhost:8082/gtm/web/guarantee/img/header_bg.bf74a3a9.png" alt="">
    <img src="https://tenfei01.cfp.cn/creative/vcg/800/version23/VCG41175510742.jpg" alt="">
    <div id="replaycontent">
    </div>
    <script>
        if(localStorage.getItem('videoData')){
            replay();
        }
        window.events = [];

        function record() {
            rrweb.record({
                emit(event) {
                    // 将 event 存入 events 数组中
                    events.push(event);
                    console.log(event);
                },
            });
        }

        function replay() {
            if(localStorage.getItem('videoData')){
                new rrwebPlayer({
                    target: document.getElementById("replaycontent"), // 可以自定义 DOM 元素
                    data: {
                        events:JSON.parse(localStorage.getItem('videoData')),
                    },
                });
            }else{
                localStorage.setItem('videoData',JSON.stringify(window.events))
                new rrwebPlayer({
                    target: document.getElementById("replaycontent"), // 可以自定义 DOM 元素
                    data: {
                        events,
                    },
                });
            }
        }
        // save 函数用于将 events 发送至后端存入，并重置 events 数组
        function save() {
            const body = JSON.stringify(window.events);
            console.log(body)
            events = [];
            fetch("http://localhost:8080/api", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body,
            });
        }
        // 每 10 秒调用一次 save 方法，避免请求过多
        // setInterval(save, 10 * 1000);
    </script>
</body>

</html>
```
## 生成文件转换成视频
对于上文中的数据写入json文件 转换成视频
rrvideo --input PATH_TO_YOUR_RRWEB_EVENTS_FILE --output OUTPUT_PATH 
视频可能是快速的需要设置一下 用下面的方法 设置自己想要的速度
ffmpeg官网对播放速度的说明：
http://trac.ffmpeg.org/wiki/How%20to%20speed%20up%20/%20slow%20down%20a%20video
## 视频加速减速 
```
ffmpeg -i input.mp4 -filter:v "setpts=4*PTS" output.mp4
```