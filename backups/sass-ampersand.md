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

可以发现因为类名要表示层级关系，因此很多类名都会有重复的部分，例如`blockName`和`elementName`，我们可以使用 **SASS** 的嵌套语法来提高我们的编写效率。

```
.blockName{
	&-elementName{
		&-modifierName{

		}
	}
}
```

这样的编写风格更清楚地反应了层级关系，也减少了重复编写的代码。

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

