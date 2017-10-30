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
