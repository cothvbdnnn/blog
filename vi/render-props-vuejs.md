# Render Props với Vuejs

## Render Props là gì?

Render props là một cách giúp chúng ta có thể sử dụng lại logic của một component cho một component khác. Sử dụng bằng cách truyền props với giá trị là một function.

## Tại sao phải sử dụng Render Props?

Ví dụ khi chúng ta muốn tạo một component `Counter` với chức năng đơn giản là sẽ render ra một thẻ p với giá trị `count` tăng dần theo mỗi giây

``` javascript
<template>
  <p v-text="count">
</template>

<script>
export default {
  data() {
    return {
      count: 0,
    }
  },
  beforeMount() {
    // Tạo một thuộc tính counter từ object this.$options nhận giá trị setInterval
    this.$options.counter = setInterval(() => {
      this.count += 1;
    }, 1000); 
    // Và clearInterval khi beforeDestroy để tránh leak memory
    this.$on('hook:beforeDestroy', () => {
      clearInterval(this.$options.counter)
    })
  },
```
Chúng ta có thể tái sử dụng được component này ở nhiều nơi, nhưng khi chúng ta muốn thay vì hiển thị thẻ p mà là một thẻ khác thì phải làm thế nào?

Giải pháp chính là sử dụng render props như sau


``` javascript
// Counter.vue

<template>
  // Truyền biến count vào hàm render, nó sẽ callback và trả về giá trị
  // Directive v-html để xuất ra html
  <div v-html="render(count)" />
</template>

<script>
export default {
  // Tạo một prop render với kiểu Function
  props: {
    render: {
      type: Function,
    }
  },
  data() {
    return {
      count: 0,
    }
  },
  beforeMount() {
    this.$options.counter = setInterval(() => {
      this.count += 1;
    }, 1000); 
    this.$on('hook:beforeDestroy', () => {
      clearInterval(this.$options.counter)
    })
  },
}
</script>
```

Vậy là đã xong component Counter.vue, bây giờ sẽ đến cách dùng nó như thế nào

``` javascript
<template>
  // Truyền prop render là một function trả về html
  <div>
    <Counter :render="renderH1" />
    <Counter :render="renderP" />
  </div>
</template>

<script>
import Counter from '../RenderProps/Counter.vue';

export default {
  components: {
    Counter,
  },
  methods: {  
    // Các function này sẽ nhận biến count từ component Counter và render ra các html tùy ý
    renderH1(count) {
      return (
        `<h1>h1 Counter: ${ count }</h1>`
      )
    },
    renderP(count) {
      return (
        `<p>p Counter: ${ count }</p>`
      )
    }
  }
}
</script>
```

Và kết quả chúng ta đã có thể hiển thị được 2 components Counter dùng thẻ `h1` và thẻ `p`

![image.png](https://images.viblo.asia/552cd970-83a5-4bcc-bc13-413c16eba9f4.png)


## Kết luận

Ví dụ ở trên chỉ là một trường hợp đơn giản về render props, thực tế sẽ có rất nhiều cách để chúng ta có thể sử dụng render props để giúp quá trình phát triển ứng dụng dễ dàng và tối ưu hơn

