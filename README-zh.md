# 虚幻引擎UE5 像素流 依赖

和官方臃肿不堪的像素流SDK相比，我们开发出了轻量、零依赖、开箱即用的软件套装，前端的peer-stream.js基于WebComponentsAPI，后端signal.js基于NodeJS和npm/ws。
 
 | 文件名         | 大小 | 作用                         |
 | -------------- | ---- | ---------------------------- |
 | peer-stream.js | 18KB | 浏览器SDK，一键开启。        |
 | signal.js      | 5KB  | 信令服务器、负载均衡、认证。 |
 | .signal.js     | <1KB | 通过环境变量调节signal.js。  |
 | test.html      | 3KB  | 测试网页。                   |

## Demo

```s
# 安装 WebSocket
npm install ws@8.5.0

# 启动信令服务器
PORT=88 node signal.js

# 启动 UE5
start path/to/UE5.exe -PixelStreamingURL="ws://localhost:88"

# 打开测试网页
start http://localhost:88/test.html
```

## signal.js 信令服务器

signal.js在官方库的基础上做了大量优化

- 提供http文件服务，和WebSocket共享端口号。
- 面向前端和面向UE5的端口号绑定，通过WebSocket子协议区分。
- 通过环境变量统一传参。
- 提供密码认证服务。
- 可以限制最大连接数。
- 支持多个UE5连接。
- 控制台实时打印UE5和前端的多对多映射关系。
- 对WebSocket连接做节流过滤，提高稳定性。
- 支持UE5和前端一一映射。
- 前端连入时，可以自动启动UE5进程。
- 多个UE5连入时，负载均衡。
- 支持stun公网穿透，在公网间互连。
- 控制台可输入调试代码，并打印计算结果。
- 定时发送心跳连接保活。
- 前端端口号作为前端ID。


### .signal.js 环境变量

| 环境变量 | 类型       | 默认值    | 功能                           |
| -------- | ---------- | --------- | ------------------------------ |
| PORT     | 正整数     | 88        | WebSocket/HTTP 全局统一端口号  |
| UE5_*    | 命令行列表 | []        | UE5自启动脚本池                |
| one2one  | 布尔       | false     | 限制UE5和前端一一映射          |
| token    | 字符串     | ''        | WebSocket 密码认证             |
| limit    | 正整数     | +Infinity | 限制前端最大连接数             |
| throttle | 布尔       | false     | WebSocket 节流, 避免频繁的重连 |

### 负载均衡

`signal.js` 既支持多个前端连接，也支持多个UE5连接，此时前端和UE5的多对多映射关系是均衡负载的：前端会被引向最空闲的UE5进程。若想要限制一一映射关系，开启`one2one` 环境变量。 Provide `UE5_*` to start UE5 automatically. More detailed example in `.signal.js`.

```mermaid
flowchart LR;
    121{一一映射?};
    match(匹配);
    finish(结束);
    UE5{存在UE5进程?};
    join(前端连入);
    start[启动UE5];
    idle{空闲UE5进程?};
    min[寻找最小负载];
 
    join --> UE5;
    UE5 -->|否| start;
    start -->|启动失败| finish;
    start -->|一段时间后| match;
    UE5 -->|是| 121;
    121 -->|开| idle; 
    idle -->|无| start;
    idle -->|有| match;
    121 -->|关| min; 
    min --> match;
```

## Unreal Engine

enable the plugin:

```s
Plugins > Built-In > Graphics > Pixel Streaming > Enabled
Editor Preferences > Level Editor > Play > Additional Launch Parameters
start path/to/UE5.exe -{key}={value}
```

common startup options:

```s
 -PixelStreamingURL="ws://localhost:88"
 -RenderOffScreen
 -Unattended
 -GraphicsAdapter=0
 -ForceRes
 -Windowed
 -ResX=1280
 -ResY=720
 -AudioMixer
 -AllowPixelStreamingCommands
 -PixelStreamingEncoderRateControl=VBR
```

## peer-stream.js 前端开发包

HTML:

```html
<script src="peer-stream.js"></script>
<video is="peer-stream" id="ws://127.0.0.1:88/"></video>
```

or JavaScript:

```html
<script type="module">
import "peer-stream.js";
const ps = document.createElement("video", { is: "peer-stream" });
ps.id = "ws://127.0.0.1:88/";
document.body.append(ps);
</script>
```

### Messages

sending messages:

```js
// object will be JSON.stringify()
ps.emitMessage(msg: string | object);
```

receiving messages:

```js
ps.addEventListener("message", e => {
    // JSON.parse(e.detail)
});
```

## Requirement

- Google Chrome 90+
- Unreal Engine 5.0.3
- NodeJS 14+
- npm/ws 8.0+

 