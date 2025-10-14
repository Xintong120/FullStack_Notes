# svg+react

### SVG元素表

| 排名 | SVG元素      | 用途     | 使用频率 | 示例                                       |
| :--- | :----------- | :------- | :------- | :----------------------------------------- |
| 1    | `<svg>`      | 根容器   | ⭐⭐⭐⭐⭐    | `<svg viewBox="0 0 24 24">`                |
| 2    | `<path>`     | 复杂图形 | ⭐⭐⭐⭐⭐    | `<path d="M10 10 L 50 50" />`              |
| 3    | `<circle>`   | 圆形     | ⭐⭐⭐⭐     | `<circle cx="50" cy="50" r="40" />`        |
| 4    | `<rect>`     | 矩形     | ⭐⭐⭐⭐     | `<rect width="100" height="50" />`         |
| 5    | `<g>`        | 分组     | ⭐⭐⭐      | `<g fill="red">...</g>`                    |
| 6    | `<ellipse>`  | 椭圆     | ⭐⭐       | `<ellipse rx="40" ry="20" />`              |
| 7    | `<line>`     | 直线     | ⭐⭐       | `<line x1="0" y1="0" x2="100" y2="100" />` |
| 8    | `<polygon>`  | 多边形   | ⭐⭐       | `<polygon points="0,0 50,0 25,50" />`      |
| 9    | `<polyline>` | 折线     | ⭐        | `<polyline points="0,0 20,20 40,10" />`    |
| 10   | `<text>`     | 文字     | ⭐        | `<text x="10" y="20">Logo</text>`          |

### 样式属性

所有SVG元素都可以设置样式属性

| HTML属性          | React JSX属性    | 值类型        | 示例                                  |
| :---------------- | ---------------- | ------------- | ------------------------------------- |
| `class`           | `className`      | string        | `className="w-8 h-8"`                 |
| `fill`            | `fill`           | color         | `fill="red"` / `fill="currentColor"`  |
| `stroke`          | `stroke`         | color         | `stroke="blue"`                       |
| `stroke-width`    | `strokeWidth`    | number/string | `strokeWidth="2"` / `strokeWidth={2}` |
| `stroke-linecap`  | `strokeLinecap`  | enum          | `strokeLinecap="round"`               |
| `stroke-linejoin` | `strokeLinejoin` | enum          | `strokeLinejoin="round"`              |
| `fill-opacity`    | `fillOpacity`    | 0-1           | `fillOpacity="0.5"`                   |
| `stroke-opacity`  | `strokeOpacity`  | 0-1           | `strokeOpacity="0.8"`                 |
| `opacity`         | `opacity`        | 0-1           | `opacity="0.7"`                       |

### 样式继承规则

| 属性          | 继承性 | 说明                       |
| ------------- | :----- | :------------------------- |
| `fill`        | 可继承 | 父元素的填充色会传给子元素 |
| `stroke`      | 可继承 | 父元素的描边色会传给子元素 |
| `strokeWidth` | 可继承 | 描边宽度会继承             |
| `opacity`     | 可继承 | 透明度会继承               |
| `fontFamily`  | 可继承 | 字体会继承                 |
| `fontSize`    | 可继承 | 字号会继承                 |
| `viewBox`     | 不继承 | 只对 `<svg>` 有效          |
| `width`       | 不继承 | 每个元素独立               |
| `height`      | 不继承 | 每个元素独立               |
| `x`, `y`      | 不继承 | 位置属性不继承             |
| `d` (path)    | 不继承 | 路径数据不继承             |

### 样式优先级

| 优先级                    | 位置                | 示例                                  |
| :------------------------ | :------------------ | :------------------------------------ |
| **1. 行内样式（最高）**   | 直接在元素上        | `<circle fill="red" />`               |
| **2. className**          | CSS类               | `<circle className="fill-red-500" />` |
| **3. 父元素继承（最低）** | 从 `<svg>` 或 `<g>` | `<svg fill="red"><circle /></svg>`    |