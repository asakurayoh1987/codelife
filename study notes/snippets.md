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

