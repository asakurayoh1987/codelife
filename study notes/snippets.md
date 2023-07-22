# Snippets

## bash

```bash
# 通过chrome的headless模式截屏或查看dom
./Google\ Chrome --headless --disable-gpu --screenshot https://v.douyin.com/dJ9DhCy/
./Google\ Chrome --headless --disable-gpu --dump-dom https://www.douyin.com/video/6992482384893496576\?previous_page\=app_code_link
```

```bash
# 当前文件夹下所有png图片转换成jpg格式
for image in *.png;
do
convert -quality 85 "$image" "${image%.*}.jpg";
done
```

```bash
# ffmpeg获取音频时长
ffmpeg -i 曾经的你.mp3 2>&1 | grep -Eo 'Duration: [^\s,]+' | cut -d ' ' -f2
```

```bash
# 查询当前目录下的mp4文件，每行传入一个给mv命令，将其移动到指定目录下
# -n指定每次输出条目数，这里传1，每次传入一个地址
# -I 用户指定一个占位符，在后续命令中可以使用这个占位符
find . -name "*.mp4" | xargs -n1 -I{} mv {} ~/Downloads/sketch教程
```

```bash
# grep -v 过滤指定的字符串
cat vring.kuyin123.com.access.log.bak20220816 | grep 'GET /friend/296cc1ba42e3fb42' | grep '16/Aug' | grep -v 'Apache-HttpClient/UNAVAILABLE' | wc -l

# awk 打印指定字段，然后排序去重统计行数
cat vring.kuyin123.com.access.log.bak20220816 | grep 'GET /friend/296cc1ba42e3fb42' | grep '16/Aug' | grep -v 'Apache-HttpClient/UNAVAILABLE' | awk '{print $6}' | sort | uniq | wc -l
```

```bash
# 当前文件下的png图片转webp
for file in $(ls ./*.png); do
cwebp $file -o $(echo $file | sed 's/.png/.webp/')
done
```

```bash
# 批量删除分支
git branch | grep feature_2021 | xargs -n1 -I{} git branch -d {}
```

```bash
# 根据用户ip查看用户所在地及运营商类型
for ip in $(cat ip.txt); do
  curl "http://whois.pconline.com.cn/ipJson.jsp?ip=$ip&json=true" >> r.txt
done;
```

```bash
# git clone时指定--single-branch表示仅下载单个分支，--depth=1表示仅下载单个commit，这样速度会快几十倍，算是一个加速小技巧
git clone --depth=1 --single-branch git@github.com:ant-design/ant-design.git
```

```bash
# 替换当前目录下每个子git项目的远程地址
for d in $(ls); do
cd $d;
# git remote set-url origin $(git remote -v | grep '(fetch)' | awk '{print $2}' | sed 's/git@git.iflytek.com:/ssh:\/\/git@code.iflytek.com:30004\//');
# git remote -v
git pull
cd ..;
done;
```

```bash
oldIFS=${IFS};
IFS=$'\n';

for f in $(ls *.mp4);
do
  ffmpeg -i "$f" -f srt -i "`echo $f | sed s/.mp4/.srt/`" -c:v copy -c:a copy -c:s mov_text "`echo $f | sed s/.mp4/_merged.mp4/`";
done;

IFS=${oldIFS}
```

```bash
# 打印powerlevel10k的颜色
for i in {0..255}; do print -Pn "%K{$i}  %k%F{$i}${(l:3::0:)i}%f " ${${(M)$((i%6)):#3}:+$'\n'}; done
```

```bash
# 获取本机公网ip地址
curl 'https://open.kuyin123.com/q_ip'
```

```bash
# 统计接口请求耗时
grep q_mrc unionorder.log_20230103 | grep 'call service' | grep '"0000"' |  jq .service.res.cost | awk 'BEGIN {max=0;min=65536} {if ($1+0<min+0) min=$1 fi;if ($1+0>max+0) max=$1 fi;sum+=$1} END{print "max=",max,"min=",min,"times=",NR,"avg=",sum/NR}'
```

```bash
# 统计接口请求耗时在指定值范围的调用次数
grep '"http://172.22.145.102/h5/q_biz"' unionorder.log_20230103 | grep 'call service' | grep '"0000"' | jq -c '{"tc": .service.req.id, "phone":.service.req.params.phone,"cost":.service.res.cost} | select(.cost <= 3000)' | wc -l
```

```bash
# jq多条件搜索
cat result.log | jq '. | select((.phone=="13843140031") and .opdesc=="sendOrderResult")'
```

```bash
# 获取某个包
npm info --loglevel=silent @ring-order/sdk | grep -oP '(?<=latest: )((\d+\.){2}\d+)'
```

```bash
# 这里用到的jq在取字段时，如果字段有特殊字符在里面时的用法
grep x-requested-with unionorder.log_20230611 | jq '.req.headers."x-requested-with"' | sort | uniq -c | sort -nr | head -n 100
```

```bash
# curl请求的各阶段耗时统计
curl -o /dev/null -sS -w "DNS Lookup: %{time_namelookup}s\nConnect: %{time_connect}s\nApp Connect: %{time_appconnect}s\nPre-transfer: %{time_pretransfer}s\nRedirect: %{time_redirect}s\nStart Transfer: %{time_starttransfer}s\nTotal time: %{time_total}s\n" 'https://kuyin.iflysec.com/union-ycyu/api/v1/q_base?btp=1&cid=975d3f8c4f82012d&from=ycyu'
```

## css

```css
/* 修改输入框光标颜色 */
.caret-color {
  caret-color: #ffd476;
}
```

```css
/* input为number时去除尾部小箭头 */
.no-arrow::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
```

```css
/* 隐藏滚动条 */
.box-hide-scrollbar::-webkit-scrollbar {
  display: none; /* Chrome Safari */
}
```

```css
/*自定义scrollbar样式*/
html::-webkit-scrollbar {
  width: 10px;
}

html::-webkit-scrollbar-thumb {
  background: -webkit-gradient(linear, left top, left bottom, from(#ff8a00), to(#da1b60));
  background: linear-gradient(to bottom, #ff8a00, #da1b60);
  border-radius: 5px;
  height: 30px;
  -webkit-box-shadow: inset 2px 2px 2px rgba(255, 255, 255, .25), inset -2px -2px 2px rgba(0, 0, 0, .25);
  box-shadow: inset 2px 2px 2px rgba(255, 255, 255, .25), inset -2px -2px 2px rgba(0, 0, 0, .25)
}

html::-webkit-scrollbar-track {
  background: linear-gradient(to right, #201c29, #201c29 1px, #100e17 1px, #100e17)
}
```

```javascript
// 将如“my-component”转成“MyComponent”
const classifyRE = /(?:^|[-_])(\w)/g
const classify = str => str
  .replace(classifyRE, c => c.toUpperCase())
  .replace(/[-_]/g, '')
```



## node

```bash
# nvm安装node版本
nvm install v14.19.0 --reinstall-packages-from=v14.17.0
```



## javascript

```javascript
// 调起OPPO主题商店
function launchApp(e, i) {
    if (isInWX)
        return void $(".wx-tip").show();
    resourceType = resourceType ? resourceType : typeMap[utilTool.getUrlQuery("type")];
    var o = document.createElement("iframe");
    o.style.display = "none",
    "home" == e ? o.src = "oaps://theme/home?from=h5" : o.src = "oaps://theme/detail?from=h5&rtp=" + resourceType + "&id=" + e,
    i !== !1 && loadingMask.show(),
    document.body.appendChild(o);
    var a = Date.now();
    setTimeout(function() {
        Date.now() - a < 4010 && toastInfo.show(),
        loadingMask.hide(),
        setTimeout(function() {
            toastInfo.hide()
        }, 3e3)
    }, 4e3),
    setTimeout(function() {
        document.body.removeChild(o)
    }, 1e3)
}

```

```javascript
// fetch示例（获取讯飞统一技术门户上的文章）
fetch('http://tech.iflytek.com/utp/forum/post/detail/12087', {
  headers: {
    accept: 'application/json, text/plain, */*',
    'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8',
    authorization: '0v_340aaJBIiTfCW7_S2ta2bKZ7dlOC3WU__',
    'cache-control': 'no-cache',
    pragma: 'no-cache',
  },
  referrer: 'http://tech.iflytek.com/app/forum/article/12087',
  referrerPolicy: 'strict-origin-when-cross-origin',
  body: null,
  method: 'GET',
  mode: 'cors',
  credentials: 'include',
})
  .then(resp => resp.json())
  .then(resp => console.log(resp.data.content));

```

```javascript
// canvas绘制文本自然换行
// 在canvas的原型上添加一个方法
CanvasRenderingContext2D.prototype.wrapText = function (
  canvas,
  text,
  x,
  y,
  maxWidth,
  lineHeight,
) {
  // 对入参的类型进行检测
  if (typeof text != 'string' || typeof x != 'number' || typeof y != 'number') {
    return;
  }
  //如果最大宽度未定义 默认为300px
  if (typeof maxWidth == 'undefined') {
    maxWidth = (canvas && canvas.width) || 300;
  }
  //如果行高未定义 则定义为检测画布文本的行高或html页面的默认行高
  //window.getComptedStyle(Eelement) 传入节点返回节点对象
  if (typeof lineHeight == 'undefined') {
    lineHeight =
      (canvas.canvas &&
        parseInt(window.getComputedStyle(canvas.canvas).lineHeight)) ||
      parseInt(window.getComputedStyle(document.body).lineHeight);
  }
  var arrText = text.split('');
  var line = '';
  for (var n = 0; n < arrText.length; n++) {
    //每个循环累加字符
    var testLine = line + arrText[n];
    //检测累加字符 获取累加字符的高度和宽度
    var metrics = canvas.measureText(testLine);
    var testWidth = metrics.width;
    //如果累加字符的宽度大于定义的绘制文本最大宽度 则绘制累加字符的文本 并且设置换行间距再次进行绘制
    if (testWidth > maxWidth && n > 0) {
      canvas.fillText(line, x, y);
      line = arrText[n];
      y += lineHeight;
    } else {
      line = testLine;
    }
  }
  canvas.fillText(line, x, y);
};
```

```javascript
// joi检验两个字段不能同时为0
Joi.object().keys({
  rtype: Joi.any().valid('0', '2'),
  btype: Joi.any().when('rtype', {
    is: '0',
    then: Joi.required().valid('1'),
    otherwise: Joi.required().valid('0', '1'),
  })
})
```

```javascript
// 数字格式化
"1234567890".replace(/\B(?=(?:\d{3})+(?!\d))/g,',')
```

```javascript
let offset = 12345;
// 将最后一位置为 0
offset &= ~1;
```

```javascript
const isSupportWebp = (nature = 'lossy') => {
  const strategies = Object.assign(Object.create(null), {
    'lossy': 'UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA',  //有损
    'lossless': 'UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==',  // 无损
    'alpha': 'UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==',  // 透明
    'animation': 'UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA',  // 动图
  })
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => (img.width > 0 && img.height > 0) && resolve(true)
    img.onerror = () => resolve(false)
    img.src = `data:image/webp;base64,${strategies[nature]}`
  })
}

isSupportWebp().then(bool => {
  // true 表示支持，false 表示不支持
})
```

```javascript
// 获取Canvas指纹
function getCanvasFp() {
  var result = [];
  var canvas = document.createElement('canvas');
  canvas.width = 50;
  canvas.height = 30;
  canvas.style.display = 'inline';
  var ctx = canvas.getContext('2d');
  ctx.rect(0, 0, 10, 10);
  ctx.rect(2, 2, 6, 6);
  result.push(
    'canvas winding:' +
      (ctx.isPointInPath(5, 5, 'evenodd') === false ? 'yes' : 'no')
  );
  ctx.textBaseline = 'alphabetic';
  ctx.fillStyle = '#f60';
  ctx.fillRect(1, 1, 10, 10);
  ctx.fillStyle = '#069';
  ctx.globalCompositeOperation = 'multiply';
  ctx.fillStyle = 'rgb(255,0,255)';
  ctx.beginPath();
  ctx.arc(0, 0, 10, 0, Math.PI * 2, true);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = 'rgb(0,255,255)';
  ctx.beginPath();
  ctx.arc(10, 10, 10, 0, Math.PI * 2, true);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = 'rgb(255,255,0)';
  ctx.beginPath();
  ctx.arc(20, 20, 20, 0, Math.PI * 2, true);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = 'rgb(255,0,255)';
  if (canvas.toDataURL) {
    var e = canvas.toDataURL().replace('data:image/png;base64,', '');
    e = window.atob(e);
    result.push('canvas fp:' + e);
  }
  return result.join('~');
}

```

```javascript
// 数字转转成可用于作为变量名的字符串
const base54 = (function(){
    var DIGITS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ$_";
    return function(num) {
            var ret = "";
            do {
                    ret = DIGITS.charAt(num % 54) + ret;
                    num = Math.floor(num / 54);
            } while (num > 0);
            return ret;
    };
})();
```

```javascript
  function urlencode(str) {
    str = (str + '').toString();
    return encodeURIComponent(str).replace(/!/g, "%21").replace(/'/g, "%27").replace(/\(/g, "%28").replace(/\)/g, "%29").replace(/\*/g, "%2A").replace(/%20/g, "+");
  }
```

```javascript
// 判断是否是html开始标签的正则匹配
const ncname = '[a-zA-Z_][\\w\\-\\.]*'
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
const startTagOpen = new RegExp(`^<${qnameCapture}`)

// 以开始标签开始的模板
'<div></div>'.match(startTagOpen) // ["<div", "div", index: 0, input: "<div></div>"]

// 以结束标签开始的模板
'</div><div>我是Berwin</div>'.match(startTagOpen) // null

// 以文本开始的模板
'我是Berwin</p>'.match(startTagOpen) // null
```

```javascript
// 兼容OPPO R9s的vconsole版本
https://kuyin.iflysec.com/ycyu/debug/vconsole.min.js
```



## typescript

```typescript
type BuildArr<
  L extends number,
  Ele = unknown,
  Arr extends unknown[] = [],
  > = Arr['length'] extends L ? Arr : BuildArr<L, Ele, [...Arr, Ele]>

type BuildToMax<
  Max extends number,
  Ele = unknown,
  Arr extends unknown[] = [],
  > = Arr['length'] extends Max ? Arr : Arr | BuildToMax<Max, Ele, [...Arr, Ele]>

type ArrRange<Min extends number, Max extends number, Ele = unknown> = BuildToMax<
  Max,
  Ele,
  BuildArr<Min, Ele>
>

const test: ArrRange<3, 4, number> = [1, 2, 3]
// test
type cases = [
  Expect<Equal<ArrRange<0, 1>, [] | [unknown]>>,
  Expect<
    Equal<
      ArrRange<1, 3>,
      [unknown] | [unknown, unknown] | [unknown, unknown, unknown]
    >
  >,
]


type Expect<T extends true> = T
type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y
  ? 1
  : 2
  ? true
  : false

```

```typescript
type Binary = '0' | '1' | '2'

type BuildArr<T, L extends number, Arr extends unknown[] = []> = Arr['length'] extends L ? Arr : BuildArr<T, L, [...Arr, T]>

// type T1 = Binary extends string ? true : false

// type T2 = `${Binary}${Binary}`

// type T3 = BuildArr<Binary, 2>

// type T4 = [Binary, Binary]['length']

// type T5 = `${Binary}${Binary}`['length']

// type T6 = [Binary, Binary, Binary]

// type T7 = T6 extends string[] ? true : false

type BitStr<T, R extends string = ''> = T extends [infer E extends string, ...infer Tail]
    ? BitStr<Tail, `${R}${E}`>
    : R

type T9 = BitStr<BuildArr<Binary, 2>>


type BitBuild<T extends string, L extends number, Arr extends unknown[] = [], Result extends string = ''> = Arr['length'] extends L
    ? Result
    : BitBuild<T, L, [...Arr, T], `${T}${Result}`>

type T10 = BitBuild<Binary,3>
```


## linux

```bash
# 查看cpu数、内存情况， top命令后，再按数字1
top

# 查看硬盘大小
df -lh

# 查看可用内存, -m是以兆为单位，-g则是吉为单位
free -m
free -g

```

```bash
# 查看系统发行版本
cat /etc/redhat-release
```

## 微信相关

```javascript
// 关闭H5页面，退出到微信界面
function fnClose(){
  WeixinJSBridge.call('closeWindow');
}
```

## pinia

```typescript
// store/index.ts
import type { App } from 'vue';
import { createPinia } from 'pinia';

// 注意这里创新并导出了pinia的实例
export const store = createPinia();

export function setupStore(app: App) {
  app.use(store);
}

// store/modules/app/index.ts
import { defineStore } from 'pinia';
// 省略中间代码
// 导入上一个文件中导出的pinia实例
import { store } from '@/store';

export const useAppStore = defineStore('app-store', {
  // 省略中间代码
});

// 重点是这个，手动添加pinia实例
export function useAppStoreWithOut() {
	return useAppStore(store);
}
```

