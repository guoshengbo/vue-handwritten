# vue-hand-written

基于 Vue 2.x 的手写组件

<!-- ![screenshot](https://github.com/guoshengbo/vue-handwritten/blob/master/png.jpg?raw=true) -->

## 安装

```
npm i vue-hand-written --save
```

## 使用

```vue
<template>
  <div>
    <handwritten :width='400' :height="250" @getimageu8="getu8" @getimage="get"></handwritten>
  </div>
</template>

<script>
  import handwritten from 'vue-hand-written'

  export default {
      components: { 
        handwritten
      },
      methods:{
        getu8 (img) {
          console.log(img)
        },
        get (img) {
          console.log(img)
        }
      }
  }
</script>
```
width（number）: 画板宽度，默认400        

height（number）：画板宽度，默认250
