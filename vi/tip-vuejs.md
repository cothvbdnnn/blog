# Một số tip với Vuejs

![image.png](https://images.viblo.asia/7e018804-8ff5-4cb0-9af5-9c84538377ef.png)
# Thêm xóa với `$set` và `$delete`
Một số phương pháp để có thể thêm và xóa các phần tử trong array và object
## Array
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
## Object
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
# Cập nhật đồng bộ bằng `$emit` update
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
# Truy cập data của component cha bằng `$parent`
Phương pháp này giúp cho chúng ta có thể nhận được các data của component cha mà không cần truyền xuống prop

```html
<p> {{ $parent.name }} </p>
```
Chỉ nên dùng khi component này không tái sử dụng
# Xem console.log của event với $log
Tạo một vue prototype đặt tên là $log sau đó bind với console.log
```javascript
Vue.prototype.$log = console.log.bind(console)
```

Và rồi chúng ta có thể sử dụng $log để kiểm tra console.log của các event trực tiếp trên template một cách nhanh chóng mà không cần tạo function
```html
<input @keydown="$log" @keyup="$log" @keypress="$log" />
```

# Rerender lại component với key
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
Cách giải quyết là sử dụng `key` gán bằng `$route.path`, mỗi khi key là path thay đổi thì component sẽ rerender lại component

```html
<router-view :key="$route.path" />
```
# Tùy biến `v-model`

```html
<input v-model="name" />
// v-model ở phiên bản đầy đủ sẽ là như thế này
<input :value="name" @input="name = $event.target.value" />
```
Nó có dạng
```
{ prop?: string, event?: string }
```
Với giá trị mặc định `prop` là `value` còn `event` là `input`

Nếu chúng ta có thể không muốn dùng các event, prop mặc định, thì hãy thêm một property cho component tên model
```javascript
export default {
  model: {
    prop: 'name'
    event: 'change',
  }
}
```
Khi dùng nó sẽ có dạng thế này
```html
<input v-model="name" />
// v-model ở phiên bản đầy đủ sẽ là như thế này
<input :name="name" @change="name = $event.target.value" />
```
# Sử dụng `$on(‘hook:’)` và `@hook`
Khi component trở nên lớn dần thì cách viết này có thể giúp chúng ta quan sát các sự kiện dễ dàng hơn

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
`@hook` có thể bắt được các lifecycle từ các component con
```html 
<child-comp @hook:mounted="someFunction" />
```
# Dùng `immediate` để  kích hoạt watch ngay khi khởi tạo
Nếu cần kích hoạt watch ngay khi component bắt đầu khởi tạo, sử dụng handler và set `immediate` bằng `true`
```javascript
export default {
  watch: {
    title: {
      immediate: true,
      handler(newValue, oldValue) {
        console.log(newValue);
      },
    },
  },
};
```
# Nhận tất các props và events với $props và $listeners
Khi chúng ta cần phải chỉnh sửa một thư viện hoặc một component nhiều tầng mà có nhiều `props` và sự kiện `emit`, chúng ta sẽ phải viết rất tốn công
```html
<child-comp :placeholder="placeholder" :required="required" :value="value" @change="change" @click="click" />
```
```javascript
export default {
  props: {
    placeholder: {
      type: String,
      default: '',
    },
    required: {
      type: Boolean,
      default: false,
    },
    value: {
      type: Number,
    },
  methods: {
    change(data){
      this.$emit('change', data);
    },
    click(data){
      this.$emit('click', data);
    }
  }
}
```
Thay vào đấy chúng ta có thể sử dụng cách bên dưới
```html
<child-comp v-bind="$props" v-on="$listeners" />
```
Lưu ý là vẫn phải định nghĩa các props, còn các listeners sẽ tự động lắng nghe và emit ra ngoài

Khi chúng ta muốn thay thế một prop hoặc event
```html
<child-comp v-bind="{ ...$props, value: 2 }" v-on="{ ...$listeners, click: handleClick }" />
```
Hoặc khi muốn bỏ một prop hoặc event
```html
<child-comp v-bind="myProps" v-on="myListeners" />
```
```javascript
export default {
  computed: {
    myListeners() {
      const { click, ...listeners } = this.$listeners;
      return listeners
    },
    myProps() {
      const { click, ...props } = this.$props;
      return props
    },
  },
}
```
# Dynamic component và event
## Component
Một cách đơn giản và giảm thiếu số lượng code mà không cần phải dùng `v-if` `v-else` quá nhiều
```html
<component :is="currentComponent" />
```
```javascript
import ComponentA from './ComponentA.vue/';
import ComponentB from './ComponentB.vue/';

export default {
  data() {
    return {
      currentComponent: ComponentA,
    };
  },
  methods: {
    changeComponent(){
      this.currentComponent = ComponentB;
    }
  }
}
```
Nó sẽ rerender lại component mỗi khi `currentComponent` thay đổi
Nếu muốn giữ nguyên dữ liệu khi chuyển component chúng ta có thể bọc `keep-alive` ở ngoài
```html
<keep-alive>
      <component :is="currentComponent" />
</keep-alive>
```
## Event
Sử dụng khi muốn thay đổi sự kiện theo điều kiện, ví dụ như muốn thay đổi sự kiện `click` sang `dbclick` thì đây là 1 cách hiệu quả
```html
<button @[event]="test">Test</button>
```
```javascript
export default {
  data() {
    return {
      event: 'click',
    };
  },
  methods: {
    changeEvent(){
      this.event = 'dbclick';
    }
  }
}
```