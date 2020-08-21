# [RN]Flex布局管理

详细的问答资料, 请参考[StackOverflow](https://stackoverflow.com/questions/31250174/css-flexbox-difference-between-align-items-and-align-content)上的详细说明.



Flex默认的主轴方向为row, 与主轴垂直的方向为交叉轴. 如主轴方向设置为column, 交叉轴方向随之调整.

Flex可以是**单行**, 也可以是**多行**, 取决于如何设置**flexWrap**.

## 0.1 flexWrap属性

flex wrap定义子项是单行或是多行. [参考W3C文档](https://www.w3.org/TR/css-flexbox-1/#propdef-flex-wrap)

* `noWrap`: 单行
* `wrap`: 多行
* `wrap-reverse`: 同`wrap`.

如果设置的值是`noWrap`或`wrap`, 回卷之后交叉轴的起始方向与原交叉轴方向一致;

如果设置的值是`wrap-reverse`, 回卷之后的交叉轴起始方向与原交叉轴方向相反.

## 0.2 alignSelf属性

`alignSelf`将改变子项在**交叉轴**上的对齐方式, 将覆盖`alignItems`的设置.

* `auto` 默认值, 继承于`alignItems`
* `flex-start`
* `flex-end`
* `center`
* `stretch`
* `baseline`

## 0.3 position属性

`position`属性用来设置子项相对于父容器的位置, 有`relative`和`absolute`两个选项, 默认为`relative`

* `relative` 使用这个属性的时候, 可以使用`marginXXXX`相关的设置, 如`marginLeft`, `marginRight`, `marginBottom`, `marginTop`
* `absolute` 子项基于父容器的绝对位置布局, 这时候可以使用`left`, `right`, `bottom`, `top`来设置相对于父容器的绝对位置.



## 1.1 justifyContent

`justifyContent`属性控制所有flex容器沿主轴上的布局设置, 它的各属性效果如下:

![justify-content](/Users/liangqin/Public/oldmanfan.github.io/images/react-native-images/flex-justify-content.png)

## 2.1 alignItems

`alignItems`属性控制flex容器中子项沿交叉轴的布局设置, 它的各属性效果如下:

![align-items](/Users/liangqin/Public/oldmanfan.github.io/images/react-native-images/flex-align-items.png)

## 3.1 alignContent

`alignContent`属性只有当flex容器设置为多行时, 才起效. 它的效果如下:

![align-content](/Users/liangqin/Public/oldmanfan.github.io/images/react-native-images/flex-align-content.png)



