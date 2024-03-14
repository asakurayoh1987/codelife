

# Houdini使用示例

## 一、Worklets——工作线程

worklets是javascript模块，用于操作浏览器的渲染引擎，从而允许开发者自定义绘制、布局以及动画相关的行为，并以此来实现扩展CSS能力的诉求

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Houdini Demo - Starry Night</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <section id="night-sky"></section>
    <script>
      CSS.paintWorklet.addModule('./app.js');
    </script>
  </body>
</html>
```

```css
body {
  margin: 0;
  background-color: #000;
}

#night-sky {
  width: 100vw;
  height: 100vh;
  background-image: paint(starrySky);
}
```

```javascript
class Star {
  /**
   * 
   * @param {*} ctx canvas context
   * @param {*} geom gives information about element's size
   * @param {*} properties accesses our custom CSS properties for star color and size
   */
  paint(ctx, geom, properties) {
    const numStarts = 100;
    const starColors = properties.get('--star-colors') || [
      'white',
      'grey',
      'darkorange',
    ];
    const sizeRange = properties.get('--star-size-range') || '2,3';

    for (let i = 0; i < numStarts; i++) {
      const randomColor =
        starColors[Math.floor(Math.random() * starColors.length)];
      const minSize = parseFloat(sizeRange.split(',')[0]);
      const maxSize = parseFloat(sizeRange.split(',')[1]);
      const starSize = Math.random() * (maxSize - minSize) + minSize;
      const x = Math.random() * geom.width;
      const y = Math.random() * geom.height;

      ctx.fillStyle = randomColor;
      ctx.beginPath();
      ctx.arc(x, y, starSize, 0, 2 * Math.PI);
      ctx.fill();
      ctx.closePath();
    }
  }
}

registerPaint('starrySky', Star);
```

## 二、Custom Properties——自定义属性

