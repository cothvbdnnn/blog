# Sử dụng optional chaning trong template của Vue?

## Tại sao cần sử dụng
Khi chúng ta muốn hiển thị giá trị trong object ra template
``` html
<template>
  <p>{{ data.user.name }}</p>
</template>
```
Trong trường hợp này nếu thuộc tính ```user``` không tồn tại, trình duyệt sẽ báo lỗi 
```
Error in render: "TypeError: Cannot read property 'name' of undefined"
```
Và nó có thể khiến cho một một số component của thư viện UI không hiển thị, vậy nên có một cách đó là check if trước khi hiển thị
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
Rất dài dòng và mệt mỏi
## Optional chaining là gì
Được giới thiệu trong ES2020 của javascript, optional chaining ```?.``` giúp cho việc truy cập đến các phần tử trong object ngay cả khi object không tồn tại

Có 3 kiểu cú pháp trong đấy 2 kiểu gọi đến phần tử và 1 kiểu gọi đến phương thức trong object

``` javascript
data?.obj
data?.[obj]
data.method?.()
```

Hiện tại optional chaining mới chỉ được hỗ trợ trên template của Vue 3, còn Vue 2 khi sử dụng sẽ báo lỗi
```
SyntaxError: Unexpected token 
```
Nên mình sẽ hướng dẫn cách cài đặt để có thể sử dụng optional chaining trên template của Vue 2

## Cài đặt biên dịch

Sử dụng thư viện ```vue-template-babel-compiler``` để biên dịch

Chạy lệnh để cài đặt
```
yarn add -D vue-template-babel-compiler
```
Cấu hình tại vue.config.js
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
Hoặc nếu sử dụng webpack để biên dịch vue
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
Với trường hợp sử dụng Unit test với Jest cũng cần phải cấu hình để có thể biên dịch được trên môi trường test
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

## Sử dụng với template

Và bây giờ chúng ta chỉ cần viết thế này là trình duyệt không báo lỗi ```Error in render: "TypeError: Cannot read property 'name' of undefined"``` nữa
``` html
<template>
  <p>{{ data?.user?.name }}</p>
</template>
```

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
Sự kết hợp hoàn hảo