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

* * *
### 7.9
#### (5).toString和5.toString的区别
5.toString中5.会被认为是一个浮点数，因此会报错，可写成5..toString()，(5).toString中括号标识一个表达式，因此可调用方法

* * *
### 7.22

#### 精读Vue3.0 Function API

1. Vue通过value API创建响应式数据，返回的是一个Wrapper对象，因为只能对对象进行数据劫持
2. 由于返回的是Wrapper对象，则其与React中的UseRef类似，不具备[capture value](https://github.com/dt-fe/weekly/blob/v2/095.%E7%B2%BE%E8%AF%BB%E3%80%8AFunction%20VS%20Class%20%E7%BB%84%E4%BB%B6%E3%80%8B.md#capture-props)的特性
3. setup函数只会执行一次，因为内部是利用了Vue的响应式原理；而React Hooks每次都会执行，并且需要UseCallback、UseMemo进行优化，在Vue Hooks不需要考虑这些

参考：[https://juejin.im/post/5d1955e3e51d4556d86c7b09?utm_source=gold_browser_extension](https://juejin.im/post/5d1955e3e51d4556d86c7b09?utm_source=gold_browser_extension)

* * *

### 7.23

#### 微前端的相关实践

1. 微服务化：将模块作为单独的应用部署服务，主应用通过iframe引用；缺点：iframe与spa相比性能差
2. 微应用化：

    * npm组件库方式：组件库开发时是一个单独应用，拥有独立的开发环境；构建和运行时是一个个独立的模块，版本管理遵循semver较为容易
    * 每个模块都是独立的应用，拥有独立的仓库。开发时也是单独的应用，构建时通过持续集成将bundle复制到主应用

应提供模块注册、销毁等功能，在注册模块时需要与主应用的router和store进行合并，静态资源建议上传cdn，否则需要在应用间复制才可以使用。

* * *

### 7.25

#### V8相关优化

* V8一开始是将JS源代码转换成AST后直接编译到机器码执行，执行性能较高，但是占用内存过大，导致浏览器缓存出问题。并且生成机器码需要考虑硬件环境等，使V8的代码变得异常复杂
* v8.5.9中发布了Ignition字节码解释器，将AST转换成编码更加精简的字节码，字节码再通过如Turbofan的JIT生成高效的机器码。如此一来，浏览器只需缓存内存更加小的字节码即可，配合工程化的按需加载，可以减轻内存压力。不过，由于运行时多了生成字节码的环节，导致性能没有直接转换成机器码好
* JIT在解释执行字节码的过程中，会观察其执行情况，若发现热点代码，即经常执行的代码，JIT会将其编译成高效代码
* 开发者在编写JS代码时，要避免没必要的类型转换，因为JS引擎是用C++实现的，其内部考虑了JS复杂的类型转换，影响效率，JIT发现热点代码后会延用之前的类型信息，一旦类型转换，优化会失效

