# SASS中更好的嵌套写法

在使用 **SASS** 编写样式时，会经常使用到嵌套的功能。

```
.module{
	.header{
		.mark{}
	}
	.content{
		.list{}
	}
}
```

当我们使用类似 **BEM** 这种层级命名方式时，貌似用不上 **SASS** 的嵌套语法，每个类名都能表示一个元素或其状态，如下。

```
.blockName {}
.blockName-elementName{}
.blockName-elementName--modifierName {}
```


但当我们的HTML结构越来越多并且越来越深的时候，**SASS** 中的嵌套也会越来越深。
```
.module{
	.header{
		.mark{}
	}
	.content{
		.list{
			.item{
				.pic{}
				.message{
					.title{}
					.description{}
				}
			}
		}
	}
}
```
