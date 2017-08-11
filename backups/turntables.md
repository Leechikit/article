# HTML5打碟器—"摩擦摩擦"

项目源码在[]，欢迎大家 **start** 支持。

这个项目使用到了HTML5的音频API和拖放API，在我以前的文章有介绍。

* [传送门]
* [传送门]

[戳我看栗子](https://codepen.io/leechikit/full/ZJyzZY)

## 使用HTML5音频API处理音乐

### 创建音频列表

*soundList.js*
```
export default {
	'Good Goodbye': {
		cover: 'https://y.gtimg.cn/music/photo_new/T002R300x300M000004XEwud03XB6I.jpg',
		link: 'https://leechikit.github.io/resources/article/turntables/song/GoodGoodBye.m4a'
	},
	'Heavy': {
		cover: 'https://y.gtimg.cn/music/photo_new/T002R300x300M000001TZBRx2mZw8A.jpg',
		link: 'https://leechikit.github.io/resources/article/turntables/song/Heavy.m4a'
	},
	'Numb': {
		cover: 'https://y.gtimg.cn/music/photo_new/T002R300x300M000002vmOxc3x7FWa.jpg',
		link: 'https://leechikit.github.io/resources/article/turntables/song/Numb.m4a'
	},
	'In The End':{
		cover: 'https://y.gtimg.cn/music/photo_new/T002R300x300M000004ImTxE1OkGqR.jpg',
		link: 'https://leechikit.github.io/resources/article/turntables/song/InTheEnd.m4a'
	}
}
```

### 解码音频文件

*decodeAudioData.js*
```
function decodeAudioData(audioContext, url) {
	return new Promise((resolve) => {
		let request = new XMLHttpRequest();
		request.open('GET', url, true);
		request.responseType = 'arraybuffer';
		request.onload = () => {
			audioContext.decodeAudioData(request.response, (buffer) => {
				if (!buffer) {
					alert('error decoding file data: ' + url);
					return;
				} else {
					resolve(buffer);
				}
			})
		}
		request.onerror = function() {
			alert('BufferLoader: XHR error');
		}
		request.send();
	})
}
```

### 转换成buffer音频源

*bufferList.js*
```
// 创建promise
let promise = () => {
	return Promise.resolve();
}
let bufferList = {};

/**
 * 创建buffer列表
 *
 */
function createBufferList() {
	let promiseArrs = [];
	// 遍历存储音频文件地址
	for (let key in soundList) {
		let bufferPromise = promise().then(() => {
			return decodeAudioData(audioContext, soundList[key].link)
		}).then((buffer) => {
			bufferList[key] = buffer;
			return promise();
		});
		promiseArrs.push(bufferPromise);
	}
	Promise.all(promiseArrs).then((values) => {
		downloadCallback();
	});
}
```

### 创建AudioContext对象

*audioContext.js*
```
let audioContext;

try {
	audioContext = new(window.AudioContext || window.webkitAudioContext)();
} catch (e) {
	alert("Web Audio API is not supported in this browser");
}
```

### 创建音频构造函数

*createSource.js*

### 创建音调构造函数

*createOsillator.js*

## 使用HTML5拖放API选择音乐

### 拖动音乐列表选项

*createSongList.js*
```
/**
 * dragstart
 *
 */
function dragstartHandle() {
	let img = document.createElement('img');
	img.src = "../image/dragdefault.png";
	songListEl.addEventListener('dragstart', (event) => {
		/*setDragImage start*/
		event.dataTransfer.setDragImage(img, 100, 100);
		/*setDragImage end*/

		let dataList = event.dataTransfer.items;
		dataList.add(event.target.getAttribute('data-song'), "text/plain");
		console.log("dragstart");
	})
}

/**
 * dragend
 *
 */
function dragendHandle() {
	songListEl.addEventListener("dragend", (event) => {
		let dataList = event.dataTransfer.items;
		dataList.clear();
		console.log("dragend");
	});
}
```

### 放置音乐选项

*createDisk.js*
```
/**
 * dragover
 *
 */
Disk.prototype.dragoverDiskHandle = function() {
	this.diskEl.addEventListener("dragover", (event) => {
		event.preventDefault();
	});
}

/**
 * drop
 *
 */
Disk.prototype.dropDiskHandle = function() {
	this.diskEl.addEventListener("drop", (event) => {
		let dataList = event.dataTransfer.items;
		for (let i = 0, len = dataList.length; i < len; i++) {
			if (dataList[i].kind == "string" && dataList[i].type.match("^text/plain")) {
				dataList[i].getAsString((name) => {
					this.stopSound();
					this.sound = new Source({
						soundName: name,
						loop: this.loop || true
					});
					this.setCover(name);
				});
			}
		}
	});
}
```

## 使用鼠标坐标转动磁碟

