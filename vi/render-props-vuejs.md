# Render Props với Vuejs

Như chúng ta đã biết, props trong Vuejs là phương pháp truyền dữ liệu trực tiếp từ component cha xuống component con một cách nhanh chóng và hiệu quả.

Thông thường, chúng ta hay sử dụng props để truyền dữ liệu dạng string, object, array, number, boolean… mà ít khi để ý rằng props cũng có thể truyền một hàm logic.

Sử dụng props truyền logic có nhiều lợi ích trong việc phát triển code như việc tăng khả năng tái sử dụng các đoạn mã logic, tăng performance… và một trong số ứng dụng của nó là chúng ta có thể render một template thông qua props hay còn gọi là Render Props.
## Render Props là gì?

Render props là một cách giúp chúng ta sử dụng lại logic của một component con cho một component cha để render một template theo ý muốn trong Vuejs bằng cách truyền trực tiếp một hàm từ component cha xuống component con thông qua props.
## Tại sao phải sử dụng Render Props?

Ví dụ khi chúng ta muốn tạo một component Counter với chức năng đơn giản là sẽ render ra một thẻ `p` với giá trị `count` tăng dần theo mỗi giây, chúng ta có đoạn mã code như sau: 

``` javascript
// Counter.vue

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
Với cách thông thường, chúng ta có thể tái sử dụng được component Counter này ở nhiều nơi khác bằng cách import nó và khai báo vào components cần sử dụng

Tuy nhiên, chúng ta không thể thay đổi được template (ở đây là thẻ p), khi muốn có một component Counter mới với template là một thẻ `h1` chẳng hạn thì chúng ta sẽ phải viết lại component Counter hoặc sử dụng `v-if`, `v-else` trong template. 

Rõ ràng là với cách làm này sẽ không hiệu quả khi chúng ta muốn thay đổi nhiều kiểu template, chẳng hạn như lúc chúng ta cần thẻ `p`, lúc cần thẻ `h1`, lúc cần thẻ `span` … thì chúng ta sẽ phải viết khối code vô cùng dài dòng.

Trong trường hợp này, Render Props chính là giải pháp, chúng ta sẽ sử dụng props để truyền xuống cho component Counter một hàm logic có tên là `render` như dưới đây: 


``` javascript
// Counter.vue

<template>
  <div v-html="render(count)" />
</template>

<script>
export default {
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

Các bạn có thể thấy, hàm logic render sẽ được định nghĩa trong component cha, khi được truyền xuống cho component Counter, nó lại có khả năng sử dụng được các biến bên trong component con (biến `count`) để render các giá trị template mong muốn vào trong template của component Counter thông qua directive `v-html` của Vuejs.

Chúng ta sẽ sử dụng component Counter.vue này để render ra nhiều template counter khác nhau thông qua Render Props như sau: 

Tạo component cha là DemoRenderProps.vue, import component Counter, truyền xuống các component Counter các hàm logic theo từng template mong muốn như `h1`, `p`…

``` javascript
// DemoRenderProps.vue

<template>
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

Và kết quả chúng ta đã có thể hiển thị được 2 components Counter dùng thẻ `h1` và thẻ `p` bằng cách render template thông qua việc truyền hàm logic qua prop.

![image.png](https://images.viblo.asia/552cd970-83a5-4bcc-bc13-413c16eba9f4.png)


## Kết luận

Như vậy qua ví dụ trên, có thể thấy rằng, với Render Props trong Vuejs, chúng ta có thêm một lựa chọn rất hữu ích để tạo ra được các dynamic template từ một component có trước, giúp chúng ta tái sử dụng triệt để được các mã logic, code base ngắn gọn, tăng performance, thậm chí là chúng ta có thể chạy được cả các script tính toán ngay cả khi dùng directive `v-html`, điều mà chúng ta không làm được khi sử dụng thuộc tính `innerHtml` thuần của Javascript.
