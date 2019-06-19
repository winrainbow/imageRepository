### HTML5

#### 引入多媒体支持

#### 引入可编程内容

#### 引入语义web

* overflow： 定义 当一个元素的内容太大而无法适应块级格式化上下文时 该怎么做

  1. visible:默认值，内容不会被修剪，会呈现在元素框之外
  2. hidden: 内容被修剪，并且其余内容不可见
  3. scroll:内容被修剪，浏览器会显示滚动条以便查看其余内容
  4. auto:由浏览器定夺，如果内容被修剪，就会显示滚动条
  5. inherit:规定从父元素继承overflow属性值
* display: 指定了元素应该生成的框的类型

  1. inline(默认值):此元素会被显示为内联元素，元素前后没有换行符
  2. none:此元素不会被显示
  3. block:此元素将显示为块级元素，此元素前后会有换行符
  4. inline-block:行内块元素
  5. list-item:列表显示
  6. run-in:根据上下文作为块级元素或内联元素显示
  7. table:表格显示，前后有换行符
  8. inherit:从父元素继承
* onMouseLeave 和 onMouseOut 区别：
  1. onMouseOut: 不论鼠标指针离开被选中的元素还是任何子元素，都会触发mouseOut
  2. onMouseLeave:只有鼠标离开被选元素时，才会触发mouseLeave事件