# Một số tip với Vuejs

## Thêm xóa vói $set và $delete
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

// this.$delete(Array, index)
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

// this.$delete(Object, propertyName)
remove (key) {
 this.$delete(this.todos, key)
}
```

```javascript
this.$emit('update:user', user)
```
```javascript
Vue.prototype.$log = console.log.bind(console)

<input @keydown="$log" @keyup="$log" @keypress="$log" placeholder="type here">

```

```javascript
const routes = [
  {
    path: "/a",
    component: MyComponent,
  },
  {
    path: "/b",
    component: MyComponent,
  },
];

<router-view :key='$route.path' />

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
```javascript 
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