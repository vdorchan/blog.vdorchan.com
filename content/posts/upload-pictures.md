---
title: "实现上传图片以及相关概念"
date: 2021-07-07T14:32:53+08:00
---

## 效果

{{< codepen "GRmZmzP" >}}

## `<input type="file">`

我们使用 `<input type="file">` 来允许用户选择文件。

```html
<!-- `accept="image/*"` 表示只允许上传图片格式的文件。 -->
<Input id="addPic" type="file" accept="image/*" />
```

监听 onchange 获取已选择图片的信息，`files` 包含了一列 `File` 对象的 `FileList` `对象。FileList` 的行为像一个数组，所以你可以检查 length 属性来获得已选择文件的数量。

每个 File 包含下列信息：

`name`：文件名。

`lastModified`：一个数字，指定文件最后一次修改的日期和时间，以 UNIX 新纪元（1970年1月1日午夜）以来的毫秒数表示。

`lastModifiedDate` ：一个 Date 对象，表示文件最后一次修改的日期和时间。这被弃用，并且不应使用。使用 lastModified 作为替代。

`size`：以字节数为单位的文件大小。

`type`：文件的 MIME 类型。

`webkitRelativePath`：一个字符串，指定了相对于在目录选择器中选择的基本目录的文件路径（即，一个设置了 webkitdirectory 属性的 file 选择器）。 这是非标准的，应该谨慎使用。

```js
document.getElement('addPic').onchange = function () {
  // 
  console.log(this.files)
}
```

## 预览

可以将 File 对象转换为 Blob 或 base 64 显示在页面上。

添加对应的 dom

```html
<div id="preview1">
  <h2>Blob File List</h2>
</div>

<div id="preview2">
  <h2>Base64 File List</h2>
</div>
```

js部分

```js
function addImg(src, id) {
  const img = document.createElement('img')
  img.src = src
  document.getElementById(id).appendChild(img)
}

function file2Blob(file) {
  const blob = window.URL.createObjectURL(file)
  addImg(blob, 'preview1')
}

function file2Base64(file) {
  const reader = new FileReader()
  reader.readAsDataURL(file)
  reader.onload = function () {
    addImg(this.result, 'preview2')
  }
}

```

## 了解 Blob、File、FileReader 等概念

* Blob 和 File 都是数据
* FileReader 是工具

### Blob

`Blob` 全称 `binary large object`，表示二进制大对象。我们可以将它看作是**只读**的二进制文件。

通过构造函数，我们可以将任意内容转换为 Blob 对象

```js
const blob = new Blob(array, options)
// array(可选)：是一个由ArrayBuffer, ArrayBufferView, Blob, DOMString 等对象构成的 Array 
// 或者其他类似对象的混合体。（暂时可以不用理解，就可以看作是一堆数据）
// options（可选）：一个对象，用来设置Blob的一些属性。
// 主要的是一个type属性，表示Blob的类型（其他暂时也不用管）。
```

blob 拥有以下实例属性：

* blob.size: 数据大小（字节）
* blob.type: 表明该Blob对象所包含数据的MIME类型

将 `Hello World` 作为纯文本下载：

```js
const content = 'Hello World'
const blob = new Blob([content], { type: 'text/plain' })
document.getElementById('info').innerText = `size: ${blob.size}字节; type: ${blob.type}`
const link = document.createElement('a')
link.download = 'blob.txt'
link.href = window.URL.createObjectURL(blob)
link.click()
```

### File

`File` 对象也是二进制对象，基于 `Blob`，所以 `Blob` 可以使用的方法，`File` 也可以使用。

### FileReader

`Blob` 是二进制对象，如果直接读取只能拿到一堆 0 1 数据，因此需要借助专门的工具来读取，这个工具就是 `FileReader`。
