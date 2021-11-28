# How to use Optional Chaining in Vuejs

# Traditional way to render a nested object

We often use text interpolation `{{ }}` to display value of an object as follows:
``` html
<template>
  <p>{{ data.user.name }}</p>
</template>
```
In the example above, we want to display the `name` value of the `data` object, but if the `user` property does not exist, the browser will give an error `Error in render: "TypeError: Cannot read property 'name' of undefined"`.

The solution we often use to avoid errors is to use the `v-if` directive of Vuejs to check for the existence of attributes in nested objects.

We will use `v-if` in the template to check the `data` object if the `data` object has the `user` property and `user` has the `name` property, at that time the `name` will be displayed in the browser.

``` html
<template v-if=”data.user && data.user.name”>
  <p>{{ data.user.name }}</p>
</template>
```
We see that the solution above may be very handy in case our object is only about one or two levels deep.

In case of multiple levels nested object, we will have to use `v-if` multiple level such as the case below.
``` html
<template v-if=”data.obj1
  && data.obj1.obj2
  && data.obj1.obj2.obj3
  && data.obj1.obj2.obj3.obj4”
>
  <p>{{ data.obj1.obj2.obj3.obj4}}</p>
</template>
```
So it can be seen our code will become verbose, difficult to follow, and possibly error when we write the wrong order of the attributes inside the object.

To fix the problems above, we can use a new operator of javascript called Optional chaining `?.`

## Introduction to optional chaining operator
![image.png](https://images.viblo.asia/c167425c-e29d-4286-9729-ebab65916f41.png)

Optional chaining introduced in ES2020, `?.` operator help to solve the problems of accessing properties inside an object even if the object or the property inside it does not exist. Generally, there are 3 syntax styles for using the `?.` operator as follows

Use to access object properties:
``` javascript
let abc = data?.obj.abc
```
Use to access properties in square brackets:
``` javascript
let abc = data?.[obj].abc
```
Use to call a function of an object even when not sure function exists or not
``` javascript
data.method?.()
```
Applying the `?.` operator in case render of a nested object in the template, we should be able to succinctly and safely write as follow:
``` html
<template>
  <p>{{ data?.user?.name }}</p>
</template>
```
The browser will no longer give the error `Error in render: "TypeError: Cannot read property 'name' of undefined"`

However, in the case where the `data` or `user` property does not exist, we can see `undefined` displayed in the browser, which is not good for the user experience

To avoid this problem, we can set the empty value `""` or a default value that will replace `undefined` as follows:

Set empty value ""
``` html
<template>
  <p>{{ data?.user?.name ?? ”” }}</p>
</template>
```
Set default value "abc"
``` html
<template>
  <p>{{ data?.user?.name ?? ”abc” }}</p>
</template>
```
When using the methods above, the `p` tag is still displayed in the browser even if we set its default value is empty

A better way to use a combination of the`?.` operator and Vuejs directive `v-text`, in the example above, the `p` tag will only be rendered in the browser when the value of `name` property is not `nullish`

Can see `v-text` is a very good directive of Vuejs but is ignored by many people. Our example above will become
``` html
<template>
  <p v-text="data?.user?.name"></p>
</template>
```
Check the browser, we can see that in case the `data` object or the `user` property does not exist, the browser does not give the error, the `p` tag will only be rendered in case the `name` property has a specific value

Code becomes short, easy to follow, and secure.

Using the `?.` operator with the `v-text` directive to display the value of a nested object is a perfect match.

`?.` is a fairly new operator, it was introduced in ES2020 so some older browsers from before 2019 may not support this operator

Currently, only Vue 3 support this operator, so we need to install a library to be able to use it for Vue 2

## Install and configure for Vuejs 2

Firstly, install the `vue-template-babel-compiler` library to compile the `?.` operator in the template. Run the command to install as follows

```
yarn add -D vue-template-babel-compiler
```

Configure`vue.config.js` file, using `vue-template-babel-compiler` to help webpack compile
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
Or configure `webpack.config.js` file
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
With the above configuring, we can already the `?.` operator in Vue 2. In case we need to write the unit test in Jest, we need to configure the `jest.config.js` file to be able to compile on the test environment

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
Note: version of `vue-jest` must be greater than or equal to 4.0.0 and `jest` less than or equal to 26.6.3.

## Conclusion

Although it is a bit difficult when configure to be able to use the `?.` operator in Vue 2, we have more than enough reasons to do it because of many benefits and advantages as analyzed above.