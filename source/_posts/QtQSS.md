---
title: Qt QSS用法
---

# 设置控件背景

- **background**

  属性类型：`Background`

  属性类型语法为：`{Brush | Url | Repeat | Alignment}`

  比如 `QWidget {background: red url(:/img/40x40.png) no-repeat bottom}`

  ![background](background.png)

  `background-color`、`background-image`、`background-repeat`和 `background-position`。

  ---
- **background-color**

  属性类型：`Brush`

  属性类型语法为：`Color | Gradient | PaletteRole`

  - *Color*：

    `QLable { background-color: red}`

    `QLable { background-color: #FF0000}`

    `QLable { background-color: rgb(255, 0, 0)}`

  ![background-color25%red](background-color-red.png)

  `QLable { background-color: rgba(255, 0, 0, 25%)}`

  ![background-color-25red.png](background-color-25red.png)

  - *Gradient渐变填充*

  1. 线性渐变

  ```
  QTextEdit {
  background-color: qlineargradient(x1:0, y1:0, x2:1, y2:1,
              stop:0 white, stop: 0.4 gray, stop:1 green)
  }
  ```

  ![lineargradient](lineargradient.png)

  渐变位置：在这个例子中，(x1,y1)是起点，在左上角，(x2,y2)是终点在右下角。
  渐变点设置：在0处设白色，在0.4处设灰色，在1处设绿色。

  2. 辐射渐变

  ```
  QTextEdit {
  background-color: qradialgradient(cx:0, cy:0, radius: 1,
              fx:0.5, fy:0.5, stop:0 white, stop:1 green)
  }
  ```

  ![radialgradient](radialgradient.png)

  渐变位置：(cx,cy)是圆的中心，radius圆的半径，(fx,fy)是渐变的起始点。
  渐变点设置：在0处设白色，在1处设green

  3. 圆锥渐变

  ```
  QTextEdit {
  background-color: qconicalgradient(cx:0.5, cy:0.5, angle:30,
              stop:0 white, stop:1 #00FF00)
  }
  ```

  ![conicalgradient](conicalgradient.png)

  渐变位置：(cx,cy)是圆锥的中心，angle是渐变的起始角度。
  渐变点设置：在0处设白色，在1处设#00FF00。

  - *PaletteRole*

    颜色设置为调色板对应角色的值

    `QPushButton { color: palette(dark)}`

> https://blog.csdn.net/YinShiJiaW/article/details/104632028
> https://blog.csdn.net/ycyzrenlifei/article/details/126442250

---

- **background-image**

  属性类型：`Url`

  属性类型语法为：`url(filename)`

  例子：

  `QWidget { background-image: url(:/img/40x40.png) }`

  ![background-image](background-image.png)

  40x40.png大小为40x40像素，背景widget大小为110x110，从效果可以看出默认情况下，背景图片
  在x和y方向上做重复。

---

- **background-repeat**

  属性类型：`Repeat`

  属性类型语法为：`repeat-x | repeat-y | no-repeat | repeat-xy`

  `QWidget { background-image: url(:/img/40x40.png); background-repeat: repeat-x }`

  ![repeat-x](repeat-x.png)

  `QWidget { background-image: url(:/img/40x40.png); background-repeat: repeat-y }`

  ![repeat-y](repeat-y.png)

  `QWidget { background-image: url(:/img/40x40.png); background-repeat: no-repeat }`

  ![no-repeat](no-repeat.png)

  `QWidget { background-image: url(:/img/40x40.png); background-repeat: repeat-xy }`

  ![background-image](background-image.png)

---

- **background-position**

  属性类型：`Alignment`

  属性类型语法为：`{top | bottom | left | right | center}`

  `QWidget { background-image: url(:/img/40x40.png); background-repeat: no-repeat;background-position: bottom center }`

  ![bottom-center](bottom-center.png)

---

- **background-attachment**

  决定QAbstractScrollArea中的背景图像相对于窗口是滚动还是固定的。

  属性类型：`Attachment`

  属性类型语法为：`scroll | fixed`

  默认是滚动的。

  ```
  QTextEdit {
      background-image: url("leaves.png");
      background-attachment: fixed;
  }
  ```

---

- **background-clip**

  background绘制的背景矩形，此属性指定背景颜色和背景图像被裁剪到的矩形，默认是border。

  属性类型：`Origin`

  属性类型语法为：`margin | border | padding | content`

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 10px solid green;
      padding: 10px 10px 10px 10px;

      background-color: red;
      background-clip: content;
  }
  ```

  ![clip_color_content](clip_color_content.png)

  背景颜色裁剪到content。

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 10px solid green;
      padding: 10px 10px 10px 10px;

      background-color: red;
      background-clip: padding;
  }
  ```

  ![clip_color_content](clip_color_padding.png)

  背景颜色裁剪到padding。

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 10px solid green;
      padding: 10px 10px 10px 10px;

      background-color: red;
      background-clip: margin;
  }
  ```

  ![clip_color_content](clip_color_margin.png)

  背景颜色裁剪到margin。

---

- **background-origin**

  widget的背景矩形，与背景位置和背景图像一起使用。

  属性类型：`Origin`

  属性类型语法为：`margin | border | padding | content`

  此属性和box模型息息相关，box模型如下：

  ![box-model](box-model.png)

  默认情况下，margin、border-width、padding大小都是0，在这种情况下，四个长方形完全重叠。
  为了展示background-origin我们需要先设置margin、padding的大小

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 1px solid red;
      padding: 10px 10px 10px 10px;

      background-image: url(:/img/40x40.png);
      background-position: top left;
      background-repeat: no-repeat;
      background-origin: content;
  }
  ```

  ![origin-content](origin-content.png)

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 1px solid red;
      padding: 10px 10px 10px 10px;

      background-image: url(:/img/40x40.png);
      background-position: top left;
      background-repeat: no-repeat;
  }
  ```

  ![origin-padding](origin-padding.png)

  不指定background-origin时，默认值是padding。

# 设置控件边框

- **border**

  属性类型：`Border`

  属性类型语法为：`{Border Style | Length | Brush}`

  设置边框的简化形式，相当于设置 `border-width, border-style, border-color`

  `QLineEdit { border: 1px solid green }`

  ![border](border.png)

  下面四个属性单独设置某一个边框，值同 `border`:

  1. border-top
  2. border-right
  3. border-bottom
  4. border-left

---

- **border-width**

  属性类型：`Box Lengths`

  属性类型语法为：`Length{1,4}`

  设置四个边框的颜色，相当于设置 `border-top-width, border-right-width, border-bottom-width, border-left-width`。

  `QLabel { border-style: solid; border-color: red; border-width:2px }`

  ![border-width-1](border-width-1.png)

  `QLabel { border-style: solid; border-color: red; border-width:2px 4px}`

  ![border-width-1](border-width-2.png)

  `QLabel { border-style: solid; border-color: red; border-width:2px 4px 6px}`

  ![border-width-1](border-width-3.png)

  `QLabel { border-style: solid; border-color: red; border-width:2px 4px 6px 8px}`

  ![border-width-1](border-width-4.png)

  下面四个属性单独设置某一个边框的宽度:

  1. border-top-width
  2. border-right-width
  3. border-bottom-width
  4. border-left-width

     属性类型：`Length`

     属性类型语法为：`Number (px | pt | em | ex)?`

---

- **border-style**

  属性类型：`Border Style`

  属性类型语法为：

  ```
  dashed 
  | dot-dash 
  | dot-dot-dash 
  | dotted 
  | double 
  | groove 
  | inset 
  | outset 
  | ridge 
  | solid 
  | none
  ```

  默认值是none.

  ![border-style](border-style.png)

  `QLabel { border-style: double; border-color: red; border-width:8px}`

  ![border-style-double](border-style-double.png)

  下面四个属性单独设置某一个边框的样式:

  1. border-top-style
  2. border-right-style
  3. border-bottom-style
  4. border-left-style

     属性类型：`Border Style`

     属性类型语法为：同 `border-style`

---

- **border-color**

  属性类型：`Box Colors`

  属性类型语法为：`Brush{1,4}`

  设置四个边框的颜色，相当于设置 `border-top-color, border-right-color, border-bottom-color, border-left-color`。

  这个属性默认值是控件的前景色。

  `QLabel { border-width: 4px; border-style: solid; border-color: red }`

  ![border-color-1](border-color-1.png)

  `QLabel { border-width: 4px; border-style: solid; border-color: red blue}`

  ![border-color-1](border-color-2.png)

  `QLabel { border-width: 4px; border-style: solid; border-color: red blue green}`

  ![border-color-1](border-color-3.png)

  `QLabel { border-width: 4px; border-style: solid; border-color: red blue green yellow}`

  ![border-color-1](border-color-4.png)

  下面四个属性单独设置某一个边框的颜色:

  1. border-top-color
  2. border-right-color
  3. border-bottom-color
  4. border-left-color

     属性类型：`Brush`

     属性类型语法为：同 `Color | Gradient | PaletteRole`

---

- **border-image**

  属性类型：`Border Image`

  属性类型语法为：`none | Url Number{4} (stretch | repeat){0, 2}`

  在背景图和控件大小不匹配时，可以使用此属性来设置背景图。

  背景图如下:

  ![btn](btn.png)

  通过下面qss代码来设置不同大小的按钮:

  ```
  QPushButton{
      border-image:url(:/img/btn.png) 1 1 4 1;
      border-top: 1px transparent;
      border-bottom: 4px transparent;
      border-right: 1px transparent;
      border-left: 1px transparent;
  }
  ```

  ![border-image](border-image.png)

> https://blog.csdn.net/mycoolx/article/details/52318556

---

- **border-radius**

  属性类型：`Radius`

  属性类型语法为：`Length{1, 2}`

  设置控件的圆角。

  `QPushButton {border: 10px solid green; border-radius: 20px}`

  ![border-radius](border-radius.png)

  下面四个属性单独设置某一个方向的圆角:

  1. border-top-left-radius
  2. border-top-right-radius
  3. border-bottom-right-radius
  4. border-bottom-left-radius

  属性类型：`Radius`

  属性类型语法为：`Length{1, 2}`

# 设置控件外边距和内边距

- **margin**

  属性类型：`Box Lengths`

  属性类型语法为：`Length{1,4}`

  设置控件外边距大小，此属性相当于设置 `margin-top, margin-right, margin-bottom, margin-left`.
- **padding**

  属性类型：`Box Lengths`

  属性类型语法为：`Length{1,4}`

  设置控件内边距大小，此属性相当于设置 `padding-top, padding-right, padding-bottom, padding-left`.

  ```
  QWidget {
      margin:10px 10px 10px 10px;
      border: 10px solid green;
      padding: 10px 10px 10px 10px;

      background-color: red;
      background-clip: content;
  }
  ```

  ![clip_color_content](clip_color_content.png)

# 设置子控件内容背景图片

- **image**

  属性类型：`Url+`

  属性类型语法为：`url(filename)`

  设置子控件**内容部分**的背景图片，属性值可以是图片路径列表或者一个svg文件。

  属性值是图片时，会根据图片大小设置子控件的宽和高。

  属性值是svg时，svg会缩放到子控件大小。

  ```
  QCheckBox::indicator:unchecked {
      image: url(:/img/CheckBoxUncheck.png);
  }

  QCheckBox::indicator:checked {
      image: url(:/img/Checkbox.png);
  }

  QCheckBox::indicator:indeterminate {
      image: url(:/img/Checked_Part.png);
  }
  ```

  ![sub-control-image](sub-control-image.png)

# 设置字体

- **font**

  设置文本字体的速记符号，相当于设置了 `font-family, font-size, font-style, font-weight`.

  属性类型：`Font`

  属性类型语法为：`(Style | Weight){0,2} Size Family`

  1. Style和Weight的位置可以交换，并且可以省略
  2. Size不能被省略
  3. Size和Family必须写在其他两个属性的后面，并且位置不能交换
  4. Family可以省略，省略后使用默认字体

  `QLabel {font: italic bold 14px "Fira code"}`

  ![font-1](font-1.png)

  `QLabel {font: bold italic 14px "Fira code"}`

  ![font-1](font-1.png)

  `QLabel {font: bold 14px "Fira code"}`

  ![font-2](font-2.png)

  `QLabel {font: 16px }`

  ![font-3](font-3.png)

---

- **font-family**
  设置文本的字体系列

  属性类型：`String`

  `QLabel {font-family: "Fira Code"}`

  ![font-family-1](font-family-1.png)

---

- **font-size**

  设置文本的字体大小

  属性类型：`Font Size`

  属性类型语法为：`Number (px | pt)`

  `QLabel {font-size: 12px}`

  ![font-size-1](font-size-1.png)

  `QLabel {font-size: 16px}`

  ![font-size-1](font-size-2.png)

---

- **font-style**

  设置文本的字体样式

  属性类型：`Font Style`

  属性类型语法为：`normal | italic | oblique`

  `QLabel {font-style: noraml}`

  ![font-style-normal](font-style-normal.png)

  `QLabel {font-style: italic}`

  ![font-style-normal](font-style-italic.png)

  `QLabel {font-style: oblique}`

  ![font-style-normal](font-style-oblique.png)

---

- **font-weight**
  设置文本字体的粗细

  属性类型：`Font Weight`

  属性类型语法为：`normal | bold | 100 | 200 ... | 900`

  `QLabel {font-weight: bold}`

  ![font-weight-bold](font-weight-bold.png)

# 设置子控件

- **subcontrol-origin**

  设置子控件相对父控件的那个矩形（margin、border、padding、content）进行绘制。

  属性类型：`Origin`

  属性类型语法为：`margin | border | padding | content`

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: margin;
      subcontrol-position: left top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-origin-margin.png)

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: border;
      subcontrol-position: left top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-origin-border.png)

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: padding;
      subcontrol-position: left top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-origin-padding.png)

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: content;
      subcontrol-position: left top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-origin-content.png)

---

- **subcontrol-position**

  子控件绘制的位置。

  属性类型：`Alignment`

  属性类型语法为：`top | bottom | left | right | center`

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: padding;
      subcontrol-position: left top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-position-left-top.png)

  ```
  QSpinBox {
  margin: 20px;
  border: 20px solid green;
  padding:20px;

  background-color: pink;
  background-clip: content;
  }

  QSpinBox::up-button {
      subcontrol-origin: padding;
      subcontrol-position: center top;
  }


  QSpinBox::down-button {
      subcontrol-origin: content;
      subcontrol-position: right bottom;
  }
  ```

  ![subcontrol-origin-margin](subcontrol-position-center-top.png)

- **position**
  使用left、right、top和bottom指定的偏移量是相对坐标还是绝对坐标。

  默认是相对坐标。

  属性类型语法为：`relative | absolute`

---

- **left**

  如果position是relative(默认值)，将子控件向*右*移动一定的偏移量。

  如果position是absolute，则left属性指定子控件的左边缘相对于父控件的左边缘距离。

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)`

  `QSpinBox::down-button { left: 0px }`

  ![left-0](left-0.png)

  `QSpinBox::down-button { left: 6px }`

  ![left-0](left-6.png)

---

- **right**

  如果position是relative(默认值)，将子控件向*左*移动一定的偏移量。

  如果position是absolute，则right属性指定子控件的右边缘相对于父控件的右边缘距离。

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)`

  `QSpinBox::down-button { right: 6px }`

  ![right-6](right-6.png)

---

- **top**

  如果position是relative(默认值)，将子控件向*下*移动一定的偏移量。

  如果position是absolute，则top属性指定子控件的上边缘相对于父控件的上边缘距离。

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)`

  `QSpinBox::up-button { top: 6px }`

  ![right-6](top-6.png)

---

- **bottom**

  如果position是relative(默认值)，将子控件向*上*移动一定的偏移量。

  如果position是absolute，则top属性指定子控件的下边缘相对于父控件的下边缘距离。

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)`

  `QSpinBox::down-button { bottom: 6px }`

  ![right-6](bottom-6.png)

---

- **spacing**

  控件内部间距。QCheckBox、checkable QGroupBoxes、QMenuBar和QRadioButton都支持这个属性。

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)` 

  `QCheckBox {spacing: 0px}`

  ![spacing-0](spacing-0.png)

  `QCheckBox {spacing: 10px}`

  ![spacing-0](spacing-10.png)

---

- **width**

  设置子控件的宽度

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)` 

  `QSpinBox::up-button { width: 10px }`

  ![width-10](width-10.png)

  `QSpinBox::up-button { width: 20px }`

  ![width-10](width-20.png)

---

- **height**

  设置子控件的高度

  属性类型：`Length`

  属性类型语法为：`Number (px | pt | em | ex)` 

  `QSpinBox::down-button { height: 10px }`

  ![height-10](height-10.png)

  `QSpinBox::down-button { height: 20px }`

  ![height-10](height-20.png)

>https://blog.csdn.net/kanchuan1905/article/details/53714928

# 设置选择区域颜色

- **selection-background-color**

  选定文本或项目的背景颜色

  属性类型：`Brush`

  属性类型语法为：`Color | Gradient | PaletteRole` 

  `QTextEdit { selection-background-color: darkblue }`

  ![selection-background-color](selection-background-color.png)

---

- **selection-color**

  选定文本或项目的文本颜色

  属性类型：`Brush`

  属性类型语法为：`Color | Gradient | PaletteRole` 

  `QTextEdit { selection-color: red }`

  ![selection-color](selection-color.png)

# 选择器类型

| 选择器 | 例子 | 说明 |
|:-------:|:------:|:-----|
| 通配选择器 | * | 匹配所有控件 |
| 类型选择器 | QPushButton| 匹配所有QPushButton和其子类的实例 |
| 属性选择器 | QPushButton[flat="false"] | 匹配所有flat属性是false的QPushButton实例，注意该属性可以是自定义的属性，不一定非要是类本身具有的属性 |
| 类选择器 | .QPushButton | 匹配所有QPushButton的实例，但是并不匹配其子类 |
| ID选择器 | #myButton | 匹配所有id为myButton的控件实例，这里的id实际上就是objectName指定的值 |
| 包含选择器 | QDialog QPushButton | 所有QDialog容器中包含的QPushButton，不管是直接的还是间接的 |
| 子选择器 | QDialog > QPushButton | 所有QDialog容器下面的QPushButton，其中要求QPushButton的直接父容器是QDialog |

  上面所有的这些选择器可以联合使用，并且支持一次设置多个选择器类型，用逗号隔开，同CSS一样。