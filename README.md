# vue-handwritten

基于 Vue 2.x 的手写组件

<!-- ![screenshot](https://github.com/guoshengbo/vue-handwritten/blob/master/png.jpg?raw=true) -->

## 安装

```
npm i vue-handwritten --save
```

## 使用

```vue
<template>
  <div>
    <handwritten></handwritten>
  </div>
</template>

<script>
  import handwritten from 'vue-handwritten'

  export default {
      data () {
      }
      components: { handwritten }
  }
</script>
```
deadline（number）:需显示的数字        

length（number）：一共显示多少位（不能小于deadline长度，多出部分填充0显示）
