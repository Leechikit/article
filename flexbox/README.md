# Flexbox布局的正确使用姿势

## Flexbox兼容性

![Alt text](./flexbox-pc.png)
![Alt text](./flexbox-wap.png)

## Flexbox新旧属性

Flexbox的新属性提供了很多旧版本没有的功能，但是目前Android4.x和UC仍有一定市场占有率需要兼容，因此目前只使用新旧属性都有的功能。
能实现相同功能的Flexbox新旧属性如下表：
![Alt text](./flexbox-attribute.png)

但想象是美好的，现实是残酷的，新旧属性里有那么几个顽固分子并不能乖乖的表现的一样，总有那么一点不同。
下面我们来看看是哪些新旧属性有不同：
1. flex-direction:row-reverse vs box-orient:horizontal;box-direction:reverse
2. flex-direction:column-reverse vs box-orient:vertical;box-direction:reverse
3. oreder:integer vs box-ordinal-group:integer
4. flex-grow:number vs box-flex:number
5. flex-shrink:number vs box-flex:number

## Flexbox分配空间原理

## Flexbox属性缩写陷阱

## 需要注意的Flexbox特性

## 旧版Flexbox的BUG