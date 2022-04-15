# Một số tip với Vuejs

## Thêm xóa với $set và $delete
Một số phương pháp để có thể thêm và xóa các phần tử trong array và object
### Array
```javascript
add (newItem, index) {
  this.array.splice(index, 1, newItem)
}

// this.$set(array, indexOfItem, newValue)
add (newItem, index) {
  this.$set(this.array, index, newItem)
}

remove (index) {
 this.array.splice(index, 1)
}

// this.$delete(array, index)
remove (index) {
 this.$delete(this.todos, index)
}
```
### Object
```javascript
add (key, value) {
  this.object = { ...this.object, { key: value }}
}

// this.$set(object, propertyName, value)
add (key, value) {
  this.$set(this.object, key, value)
}

remove (key) {
  delete this.object[key]
} 

// this.$delete(object, propertyName)
remove (key) {
 this.$delete(this.object, key)
}
```
## Cập nhật đồng bộ bằng $emit update
Cập nhật các prop của component cha một cách nhanh chóng từ component con
```html
<dialog-component :visible.sync="isVisible" />
```

```javascript
// DialogComponent
handleClose() {
  this.$emit('update:visible', false)
}
```
Được sử dụng cho các trường hợp đóng mở dialog nhanh chóng mà không cần truyền event lên component cha

Lưu ý khi dùng ở những chỗ khác cũng phải thêm `.sync` lúc truyền props để có thể sử dụng 
## Truy cập data của component cha bằng $parent
Phương pháp này giúp cho chúng ta có thể nhận được các data của component cha mà không cần truyền xuống prop

```html
<p> {{ $parent.name }} </p>
```
Chỉ nên dùng khi component này không tái sử dụng
## Xem console.log của event với $log
Tạo một vue prototype đặt tên là $log sau đó bind với console.log
```javascript
Vue.prototype.$log = console.log.bind(console)
```

Và rồi chúng ta có thể sử dụng $log để kiểm tra console.log của các event trực tiếp trên template một cách nhanh chóng mà không cần tạo function
```html
<input @keydown="$log" @keyup="$log" @keypress="$log" />

```

## Rerender lại component với key
Phương pháp này được sử dụng đối với nhưng trường hợp component không tự render lại khi dư liệu thay đổi

Hoặc với trường hợp dưới đây là 2 path dùng chung một component với dữ liệu khác nhau, khi chúng ta thay đổi path thì component không rerender lại

```javascript
const routes = [
  {
    path: "/path1",
    component: ViewComponent,
  },
  {
    path: "/path2",
    component: ViewComponent,
  },
];
```
Cách giải quyết là sử dụng `key` gán bằng `$route.path`, mỗi khi key là path thay đổi thì component sẽ rerender lại

```html
<router-view :key="$route.path" />
```

```javascript
export default {
  mounted() {
    window.addEventListener('resize', this.resizeHandler);
    this.$on("hook:beforeDestroy", () => {
      window.removeEventListener('resize', this.resizeHandler);
    })
  },
};
```
```html 
<child-comp @hook:mounted="someFunction" />

```

```javascript
export default {
  watch: {
    title: {
      immediate: true,
      handler(newTitle, oldTitle) {
        console.log("Title changed from " + oldTitle + " to " + newTitle);
      },
    },
  },
};
```

```javascript
<child-comp v-bind="$props" v-on="$listeners" />

```

```javascript
export default {
  model: {
    event: 'change',
    prop: 'checked'
  }
}
```

```javascript
<script>
  props: {
    status: {
      type: String,
      required: true,
      default: 'syncing',
      validator: function (value) {
        return [
          'syncing',
          'synced',
          'version-conflict',
          'error'
        ].includes(value)
      }
    }
  }
</script>
```