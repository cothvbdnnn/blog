# Tìm hiểu về toán tử mới optional chaining ?. trong javascript và ứng dụng tuyệt vời của nó trong Vuejs

## Cách truyền thống để render template từ một nested object trong Vuejs

Khi chúng ta muốn hiển thị giá trị từ một nested object ra template
``` html
<template>
  <p>{{ data.user.name }}</p>
</template>
```
Trong trường hợp này nếu thuộc tính ```user``` không tồn tại, trình duyệt sẽ báo lỗi 
```
Error in render: "TypeError: Cannot read property 'name' of undefined"
```
Và nó có thể khiến cho một một số component của thư viện UI không hiển thị, thậm chí là ứng dụng chết ngay lập tức

Vậy nên có một giải pháp đó là check if
``` html
<template>
  <p>{{ data.user && data.user.name }}</p>
</template>
```
Nhưng mỗi lần phải check như vậy đối với các thuộc tính được lồng trong nhiều object thì nó sẽ trở thành như này
``` html
<template>
  <p>
    {{ 
      data.obj1
      && data.obj1.obj2
      && data.obj1.obj2.obj3
      && data.obj1.obj2.obj3.obj4
      ...
    }}
  </p>
</template>
```
Rất dài dòng và mệt mỏi khi mỗi object đều phải kiểm tra như thế này, vậy nên toán tử Optional Chaining đã được sinh ra và mình sẽ giới thiệu kỹ ở phần dưới
## Giới thiệu về toán tử optional chaining ?.

Được giới thiệu trong ES2020 của javascript, optional chaining ```?.``` giúp giải quyết vấn đề  truy cập đến các thuộc tính trong object ngay cả khi object không tồn tại

Có 3 kiểu cú pháp đó là

``` javascript
// sử dụng khi truy cập thuộc tính của object
data?.obj

// sử dụng với biểu thức là dấu ngoặc vuông
data?.[obj]

// sử dụng để gọi hàm khi chưa chắc chắn hàm đó có tồn tại hay không
data.method?.()
```
Nó sẽ không báo lỗi trên log và sẽ trả về  ```undefined``` nếu các phần tử bên trái ```?.``` là nullish

## Cách sử dụng và cấu hình toán tử ?. trong ứng dụng Vuejs

### Cách dùng
Và bây giờ chúng ta chỉ cần viết ngắn gọn thế này
``` html
<template>
  <p>{{ data?.user?.name }}</p>
</template>
```
Trình duyệt đã không còn báo lỗi ```Error in render: "TypeError: Cannot read property 'name' of undefined"``` nữa

Nhưng trên màn hình lúc này thẻ p sẽ render ra ```undefined```

Có một cách mọi người thường dùng là kết hợp với ```nullish coalescing operator``` để hiển thị ra giá trị rỗng thay vì ```undefined```

``` html
<template>
  <p>{{ data?.user?.name ?? '' }}</p>
</template>
```
Cách hay hơn là sử dụng ```v-text```, nó sẽ kiểm tra giá thị rồi mới render ra thẻ p còn không thì sẽ không render, một directive rất hay nhưng lại bị rất nhiều người bỏ qua

``` html
<template>
  <p v-text="data?.user?.name" />
</template>
```
Như vậy khi thuộc tính không tồn tại, ứng dụng cũng không báo lỗi và cũng sẽ không render ra các thẻ ```undefined``` nữa, một sự kết hợp hoàn hảo

### Cài đặt sử dụng đối với Vue 2
Hiện tại optional chaining mới chỉ được hỗ trợ trên template của Vue 3, còn Vue 2 khi sử dụng sẽ báo lỗi
```
SyntaxError: Unexpected token 
```
Nên mình sẽ hướng dẫn cách cài đặt để có thể sử dụng optional chaining trên template của Vue 2

Sử dụng thư viện ```vue-template-babel-compiler``` để biên dịch

Chạy lệnh để cài đặt
```
yarn add -D vue-template-babel-compiler
```
Cấu hình tại vue.config.js, sử dụng vue-template-babel-compiler để giúp webpack biên dịch
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
Hoặc nếu cấu hình tại webpack.config.js
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
},
```
Với trường hợp có sử dụng Unit test với Jest, lúc chạy test sẽ báo lỗi ```Unexpected token```, nên cũng cần phải cấu hình jest.config để có thể biên dịch được trên môi trường test
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
};
```
Lưu ý: phiên bản của vue-jest phải lớn hơn hoặc bằng 4.0.0 và jest nhỏ hơn hoặc bằng 26.6.3.

## Kết luận

Chúng ta đã thấy được các ưu điểm được kể trên, phương thức sử dụng rất dễ dàng và ngắn gọn, giúp cho ứng dụng hạn chế tối đa các lỗi vặt không đáng có

Tuy nhiên nó cũng có một số hạn chế vì là ra mắt trong ES2020 nên vẫn hỗ trợ đối với các các trình duyệt cũ, cần cân nhắc trước khi sử dụng