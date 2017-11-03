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
				.message{
					.title{}
					.description{}
				}
			}
		}
	}
}
```

编译出来后的 css 中选择器会变得很长。
```
.module .header .mark(){}
.module .content .list .item .message .title(){}
.module .content .list .item .message .description(){}
```

尤其当你使用类 **BEM** 命名的方式的时候，选择器就会变得更长
```
.search-form .search-form__content .search-form__content__button
```

当你试图改变嵌套关系时，很容易改变了选择器的权重。
```
.module{
	.header{
		.mark{}
	}
	.content{
		.list{
			
		}
		.item{

		}
		.message{
			.title{}
			.description{}
		}
	}
}
```

