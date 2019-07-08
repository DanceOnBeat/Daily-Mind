### 7.5
#### 使用Blob URL作为视频链接

* Blob URL指向一段类似于文件的二进制数据，通过URL.createObjectUrl创建。相同的二进制数据每一次调用API创建都会得到不同的URL，改URL只在当前页面有效，因此可以用作防盗链
* 正常情况下，只能等到一段视频全部下载完成，转成Blob URL才可播放，这样做既对服务器内存是一种消耗，同时会造成播放延迟
* 借助流媒体的技术可解决上述问题，其会将视频内容切成一段段很小的切片，每个切片上有不同的码率供选择，这样就可实现边下载边播放，或是类似视频网站上无缝切换码率的功能，流媒体协议包括：

    1. HLS（HTTP Live Streaming），以m3u8作为扩展名
    2. DASH（Dynamic Adaptive Streaming over HTTP），集成多种流媒体协议的标准协议，扩展名为.m4s或.mp4

* 分段传输后，客户端需要实现多段视频文件无缝连续播放，可以将MediaSource通过createObjectUrl创建一个指向数据容器的链接，不断地往容器中塞数据更新视频后续的内容，实现连续播放
* 一般的mp4文件都需要从头开始播放，不支持从中间某一段开始播放，因此视频分段需要使用特殊的mp4文件Fragmented MP4

参考：[https://juejin.im/post/5d1ea7a8e51d454fd8057bea?utm_source=gold_browser_extension](https://juejin.im/post/5d1ea7a8e51d454fd8057bea?utm_source=gold_browser_extension)

### 7.6
#### 归并排序的递归实现

```javascript
Array.prototype.mergeSort = function() {
  const arr = [];
  function sort(arr, tmpArr, left, right) {
    if (left >= right) {
      return;
    }
    const center = Math.floor((left + right) / 2);
    sort(arr, tmpArr, left, center);
    sort(arr, tmpArr, center + 1, right);
    merge(arr, tmpArr, left, center + 1, right);
  }
  function merge(arr, tmpArr, leftPos, rightPos, rightEnd) {
    const leftEnd = rightPos - 1;
    const nums = rightEnd - leftPos + 1;
    const index = leftPos;
    for (let i = 0; i < nums; i++) {
      if (leftPos > leftEnd || arr[leftPos] > arr[rightPos]) {
        tmpArr[i + index] = arr[rightPos];
        rightPos++;
      } else if (rightPos > rightEnd || arr[leftPos] <= arr[rightPos]) {
        tmpArr[i + index] = arr[leftPos];
        leftPos++; 
      }
    }
    for (let j = 0; j < nums; j++) { // 将当前merge用到的tmpArr部分copy到arr，保证arr有序
      arr[j + index] = tmpArr[j + index];
    }
  }
  sort(this, arr, 0, this.length - 1);
}
```

实现过程使用tmpArr来临时存储部分数据，再通过遍历copy到原数组arr中，空间复杂度为N，且copy过程效率较低

#### 快速排序实现

```javascript
Array.prototype.quickSort = function() {
  function sort(arr, left, right) {
    if (left >= right) {
      return;
    }
    let i = left;
    let j = right + 1;
    let tmp;
    const pivot = arr[left]
    while (i < j) {
      while (arr[++i] < pivot) {}
      while (arr[--j] > pivot) {}
      if (i < j) {
        tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
      }
    }
    tmp = arr[j];
    arr[j] = pivot;
    arr[left] = tmp;
    sort(arr, left, j - 1);
    sort(arr, j + 1, right);
  }
  sort(this, 0, this.length - 1);
}
```

过程中直接交换元素位置，空间复杂度较低。该实现以第一个元素为枢纽元，若考虑数据是预排序的情况，可使用三数中值分割法，但是额外计算枢纽元也增加了开销。

* * *
### 7.8
#### Object中preventExtensions/seal/freeze

* 默认情况下所有对象都是可扩展的，Object.preventExtensions可以使对象不可扩展，即不能给对象添加属性和方法，但是仍然可以修改和删除，使用Object.isExtensible来判断对象是否可扩展
* 第二个保护级别是密封对象，使用Object.seal方法密封对象。其不可扩展，且[[configurable]]被置为false，即不能删除属性和方法，但属性值可以修改。使用Object.isSealed判断对象是否密封
* 最严格的防篡改级别是冻结对象，使用Object.freeze冻结对象。冻结对象既是密封也是不可扩展的，数据属性不可修改，访问器属性仍然可写。使用Object.isFrozen判断对象是否冻结，一般用于第三方库，防止核心对象被修改

