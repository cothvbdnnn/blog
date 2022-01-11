# Tìm hiểu về toán tử mới optional chaining ?. trong javascript và ứng dụng tuyệt vời của nó trong Vuejs

# Cách truyền thống để render một nested object
Như chúng ta đã biết, để hiển thị giá trị của một object trong template của Vuejs thì chúng ta thường sử dụng text interpolation `{{ }}` như sau:
``` html
<template>`
  <p>{{ data.user.name }}</p>
</template>
```

Trong ví dụ trên, chúng ta muốn hiển thị giá trị `name` của object `data`, nhưng nếu thuộc tính `user` đứng trước nó không tồn tại thì trình duyệt sẽ báo lỗi `Error in render: "TypeError: Cannot read property 'name' of undefined"`.

Giải pháp thường hay được sử dụng để hiển thị nested object trong template mà không gây lỗi đó là sử dụng directive `v-if` của Vuejs để kiểm tra sự tồn tại của các thuộc tính trong nested object.

Chúng ta sẽ đặt `v-if` trên template để kiểm tra object `data`, nếu object `data` có thuộc tính `user` và `user` có thuộc tính `name` thì giá trị `name` mới được hiển thị trên trình duyệt.
``` html
<template v-if=”data.user && data.user.name”>
  <p>{{ data.user.name }}</p>
</template>
```
Chúng ta thấy rằng, giải pháp trên có thể rất tiện dụng trong trường hợp nested object của chúng ta chỉ có độ sâu khoảng một đến hai cấp.

Trong trường hợp nested object được lồng nhau nhiều cấp thì chúng ta sẽ phải viết điều kiện `v-if` cho nhiều cấp, chẳng hạn như trường hợp dưới đây.
``` html
<template v-if=”data.obj1
  && data.obj1.obj2
  && data.obj1.obj2.obj3
  && data.obj1.obj2.obj3.obj4”
>
  <p>{{ data.obj1.obj2.obj3.obj4}}</p>
</template>
```
Như vậy có thể thấy code của chúng ta sẽ trở nên dài dòng, khó theo dõi và có thể lỗi khi chúng ta viết không đúng thứ tự các thuộc tính bên trong object.

Để khắc phục được những bất cập ở trên, chúng ta có thể sử dụng một toán tử khác mới của javascript là toán tử `?.` (Optional Chaining)
## Giới thiệu về toán tử optional chaining ?.
![image.png](https://images.viblo.asia/c167425c-e29d-4286-9729-ebab65916f41.png)

Được giới thiệu trong ES2020, toán tử `?.` giúp giải quyết vấn đề truy cập đến các thuộc tính trong object ngay cả khi object hoặc thuộc tính bên trong nó không tồn tại. Thông thường, có 3 kiểu cú pháp để sử dụng toán tử `?.` như sau

Sử dụng để truy cập thuộc tính của object:
``` javascript
let abc = data?.obj.abc
```
Sử dụng để truy cập thuộc tính theo kiểu dấu ngoặc vuông:
``` javascript
let abc = data?.[obj].abc
```
Sử dụng để gọi hàm của một object ngay cả khi không chắc chắn hàm đó có tồn tại hay không
``` javascript
data.method?.()
```

Áp dụng toán tử `?.` vào trường hợp hiển thị nested object trong template, chúng ta sẽ có thể viết ngắn gọn và an toàn như sau:
``` html
<template>
  <p>{{ data?.user?.name }}</p>
</template>
```
Trình duyệt sẽ không còn báo lỗi `Error in render: "TypeError: Cannot read property 'name' of undefined"` nữa.

Tuy nhiên, trong trường hợp mà thuộc tính `data` hoặc `user` không tồn tại, chúng ta thấy giá trị `undefined` sẽ được hiển thị ra trên trình duyệt, như vậy thì sẽ không tốt cho trải nghiệm của người dùng.

Để tránh vấn đề này, chúng ta có thể set giá trị rỗng `“”` hoặc một giá trị mặc định nào đó sẽ thay thế `undefined` như sau:

Set giá trị rỗng “”
``` html
<template>
  <p>{{ data?.user?.name ?? ”” }}</p>
</template>
```
Set giá trị mặc định “abc”
``` html
<template>
  <p>{{ data?.user?.name ?? ”abc” }}</p>
</template>
```
Khi sử dụng các cách ở trên, thẻ `p` vẫn được hiển thị ra trình duyệt ngay cả khi chúng ta set giá trị mặc định cho nó là rỗng.

Có một cách hay hơn đó là sử dụng kết hợp toán tử `?.` và directive `v-text` của Vuejs, trong ví dụ ở trên, thẻ `p` sẽ chỉ được hiển thị ở trình duyệt khi giá trị của thuộc tính `name` không phải là `nullish`

Có thể thấy `v-text` là một directive rất hay của Vuejs nhưng lại bị rất nhiều người bỏ qua. Ví dụ trên của chúng ta sẽ trở thành.
``` html
<template>
  <p v-text="data?.user?.name"></p>
</template>
```
Kiểm tra trình duyệt, chúng ta thấy trong trường hợp object `data` hoặc thuộc tính `user` không tồn tại thì vẫn không báo lỗi, thẻ `p` sẽ chỉ render trong trường hợp thuộc tính `name` có giá trị cụ thể.

Code trở nên ngắn gọn, dễ theo dõi và an toàn.

Như vậy rõ ràng, chúng ta thấy sử dụng toán tử `?.` đi kèm với directive `v-text` để hiển thị giá trị của một nested object chính là một sự kết hợp quá hoàn hảo.

Do `?.` là một toán tử khá mới, được giới thiệu trong ES2020 nên có thể một số phiên bản trình duyệt cũ từ trước năm 2019 không hỗ trợ toán tử này.

Ngoài ra, hiện chỉ có phiên bản Vue 3 hỗ trợ toán tử này, do vậy, để có thể sử dụng được toán tử này trên phiên bản Vue 2 và an toàn trên các trình duyệt cũ, chúng ta cần cài đật đối với Vuejs 2

## Cài đặt và cấu hình cho Vuejs 2

Trước hết, cài đặt thư viện `vue-template-babel-compiler` để biên dịch toán tử `?.` trên template. Chạy lệnh để cài đặt
```
yarn add -D vue-template-babel-compiler
```

Chỉnh sửa lại file `vue.config.js`, sử dụng `vue-template-babel-compiler` để giúp webpack biên dịch
``` javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap(options => {
        options.compiler = require('vue-template-babel-compiler')
        return options
      })
  }
}
```
Hoặc có thể cấu hình tại file webpack.config.js
``` javascript
// webpack.config.js
module: {
  rules: [
    {
      test: /\.vue$/,
      loader: "vue-loader",
      options: {
        compiler: require("vue-template-babel-compiler"),
      },
    },
  ],
}
```
Với cấu hình như trên thì chúng ta đã có thể sử dụng được toán tử `?.` trong Vuejs 2. Trong trường hợp cần viết unit test bằng Jest, chúng ta cần cấu hình thêm `jest.config.js` để có thể biên dịch được trên môi trường test.
``` javascript
// jest.config.js
module.exports = {
  transform: {
    '.*\\.(vue)$': 'vue-jest',
  },
  globals: {
    'vue-jest': {
      templateCompiler: {
        compiler: require('vue-template-babel-compiler'),
        transformAssetUrls: true,
      },
    },
  },
}
```
Lưu ý: phiên bản của `vue-jest` phải lớn hơn hoặc bằng 4.0.0 và `jest` nhỏ hơn hoặc bằng 26.6.3.

## Kết luận

Mặc dù có một chút khó khăn khi config để có thể sử dụng được toán tử `?.` trên Vuejs 2, nhưng chúng ta có thừa lý do để thực hiện vì nhiều lợi ích và ưu điểm vượt trội mà toán tử `?.` mang lại cho chúng ta khi được sử dụng trong Vuejs như đã phân tích ở trên.