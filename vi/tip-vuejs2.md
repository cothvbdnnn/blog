## Tối ưu với v-once
Sử dụng đối với các thẻ không có dữ liệu hoặc có dữ liệu nhưng chúng ta chỉ muốn hiển thị nó 1 lần, giúp tối ưu hóa dữ liệu

Nó sẽ không render lại dữ liệu trong thẻ đấy kể cả chúng ta có thay đổi
```html
<p v-once>{{ text }}</p>
```