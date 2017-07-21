# HTML5 音频 API

如下图，在 **AudioContext**音频接口中，

[图]

## AudioContext
```
let audioContext = new(window.AudioContext || window.webkitAudioContext)();
```

## Buffer
使用`decodeAudioData()`方法把音频文件编译成buffer格式。
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

## AudioNode


## AudioParam

## AudioBufferSourceNode

## GainNode

## decodeAudioData();

## createBufferSource();
```
let bufferSource = audioContext.createBufferSource();
```

## createGain();
```
let gainNode = audioContext.createGain();
```

## createBiquadFilter();
```
let filter = audioContext.createBiquadFilter();
```

## createOscillator();