# Viết test dễ dàng hơn với thứ này

## Nhìn qua về unit test

Bài viết này phù hợp với những người đã biết về unit test, nên mình sẽ không giải thích các khái niệm căn bản nữa và sẽ đi thẳng vào những phần chính

Unit test giống như chúng ta viết lại một lần component đấy, nhưng khi ứng dụng càng lớn, nhiều thư viện được sử dụng như Vuex, Vue Router... nó sẽ khiên cho các file test trở nên rất dài, đôi khi còn dài hơn cả component cần test

Ví dụ về một component có sử dụng store và router như thế này
``` html
<!-- CreateTodo.vue -->
<script>
export default {
  name: 'CreateTodo',
  computed: {
    user() {
      return this.$store.state.user
    }
  },
  methods: {
    submit() {
      this.$router.push({ name: 'List' })
    }
  }
}
</script>
```
Khi tiến hành test chúng ta sẽ cần setup router và store để có thể test được component này
``` javascript
// CreateTodo.spec.js
import { shallowMount, createLocalVue } from '@vue/test-utils'
import CreateTodo from '@/components/CreateTodo'
import userModule from '@/store/modules/user'
import VueRouter from 'vue-router'
import Vuex from 'vuex'

const localVue = createLocalVue()
localVue.use(VueRouter)
localVue.use(Vuex)
// Tạo localVue sử dụng VueRouter và Vuex

const router = new VueRouter({
  routes: [
    {
      path: '/list',
      name: 'List',
      component: () => import('@/components/List.vue')
    }
  ]
})
// Tạo router liên kết với component List.vue

const store = new Vuex.Store({
  modules: {
    user: userModule
  }
})
// Tạo store với user module

let wrapper
beforeEach(() => {
  wrapper = shallowMount(CreateTodo, {
    localVue,
    router,
    store
  })
  // Tạo một wrapper từ shallowMount với router và store để sử dụng
})

afterEach(() => {
  wrapper.destroy()
})

describe('Test CreateTodo', () => {
  ...
})
```
Như vậy, khi có nhiều component sử dụng store và router, đều phải setup lại những phần router và store lặp đi lặp lại thế này, nên bài viết này mình sẽ cấu trúc lại để có thể viết test một cách ngắn ngọn và đơn giản hơn
## Tổ chức thư mục
```
test
└── unit               
    ├── components
    |   └── CreateTodo.spec.js
    └── methods.js                 
```
## Tạo các function dùng lại
``` javascript
// methods.js
import { shallowMount, mount, createLocalVue } from '@vue/test-utils';
// có ba thứ cần dùng là shallowMout, mount và createLocalVue
import Vuex from 'vuex';
import VueRouter from 'vue-router';
import { routes } from '@/router/index';
import user from '@/store/modules/user';
// import routes và các module từ router và store của bạn

export const localVue = createLocalVue();
localVue.use(VueRouter);
localVue.use(Vuex);
// Tạo localVue sử dụng VueRouter và Vuex

export const createStore = (customModule = {}) => new Vuex.Store({
  modules: {
    user,
    ...customModule,
  },
});
// createStore đã được cấu hình sẵn với các module được import từ store của bạn 
// Có thể custom với giá trị đầu vào nhận một custom module có kiểu dữ liệu object, sẽ được merge với các module mặc định

export const createRouter = (customRoute = []) => new VueRouter({
  routes: [
    ...routes,
    ...customRoute
  ],
});
// createRouter đã được cấu hình sẵn với các routes được import từ router của bạn
// Có thể custom với giá trị đầu vào nhận vào một custom route có kiểu dữ liệu array chứ các object routes, sẽ được merge với các routes mặc định


export const createMock = ((customMock = {}) => ({
  // thêm các mock mặc định ở đây
  $t: () => '', // ví dụ với i18n
  ...customMock,
}));
// createMock với các mock mặc định và customMock dùng để custom

export const createStub = ((customStub = {}) => ({
  // thêm các stub mặc định ở đây
  'ButtonComponent': { template: '<div></div>' },
  ...customStub,
}));
// createStub với các stub mặc định và customStub dùng để custom

export const createShallow = (component, options = {}) => shallowMount(component, {
  stubs: createStub(),
  store: createStore(),
  router: createRouter(),
  mocks: createMock(),
  localVue,
  ...options,
});
// Khởi tạo hàm createShallow, hàm này sẽ trả về hàm shallowMount của vue-test-utils
// Nó đã được nhận vào sẵn các stubs, store, router, mocks từ các hàm bên trên
// Và đầu vào sẽ nhận 2 giá trị là component và object chứa các options như shallowMount 

export const createDeep = (component, options = {}) => mount(component, {
  stubs: createStub(),
  store: createStore(),
  router: createRouter(),
  mocks: createMock(),
  localVue,
  ...options,
});
// Tương tự như createShallow chỉ khác là sử dụng mount thay vì shallow

```
## Hướng dẫn sử dụng
Và sau đó khi sử dụng chỉ cần dùng

``` javascript
import CreateTodo from '@/components/CreateTodo'
import { createShallow } from '../methods'

let wrapper
beforeEach(() => {
  wrapper = createShallow(CreateTodo)
})

afterEach(() => {
  wrapper.destroy()
})

describe('Test CreateTodo', () => {
  ...
})
```
So sánh với ví dụ ban đầu thì số lượng code đã được giảm thiểu rõ rệt

Từ giờ thay vì gọi hàm shallowMount từ vue-test-utils chúng ta chỉ cần sử dụng createShallow được viết sẵn từ file methods.js

wrapper được tạo từ createShallow đã có đầy đủ store, router, mock và stub...
## Tùy chỉnh 

Những trường hợp cần tùy chỉnh trong quá trình test sẽ xảy ra không nhiều. Mặc dù vậy, các function vẫn cho phép chúng ta sử dụng trong những trường hợp cần phải tùy chỉnh 
### Router
``` javascript
import { createShallow, createRouter } from '../methods'
import TodoComponent from '@/components/Todo'

const customRouter = [
  {
    path: '/custom-router',
    name: 'CustomRouter',
    component: () => import('@/components/CustomRouter.vue')
  }
]
// Tạo một custom router

const router = createRouter(customRouter)
// createRouter() sẽ merge customRouter với các router hiện tại

let wrapper
beforeEach(() => {
  wrapper = createShallow(TodoComponent, {
    router
  })
  // truyền options là router mới sẽ thay thế router cũ
})
```
### Store
``` javascript 
import { createShallow, createStore } from '../methods'
import TodoComponent from '@/components/Todo'
import todo from '@/store/modules/todo'

const state = {
  todos: [
    { title: "newTodo" }
  ]
}
// Tạo state với dữ liệu mẫu để test

const store = createStore({
  todo: { 
    ...todo, 
    state 
  }
})
// Replace state và giữ lại getters, mutations, actions
// createStore() sẽ merge các module vào vuex

let wrapper
beforeEach(() => {
  wrapper = createShallow(TodoComponent, {
    store
  })
  // truyền options là store mới sẽ thay thế store cũ
})
```
### Mock
``` javascript 
import { createShallow, createMock } from '../methods'
import TodoComponent from '@/components/Todo'

const mocks = createMock({
  $t: () => '',
})
// createMock() sẽ merge các mock mới với các mock mặc định

let wrapper
beforeEach(() => {
  wrapper = createShallow(TodoComponent, {
    mocks
  })
  // truyền options là mocks mới sẽ thay thế mocks cũ
})
```
### Stub
``` javascript 
import { createShallow, createStub } from '../methods'
import TodoComponent from '@/components/Todo'

const stubs = createStub({
  'CustomStub': { template: '<div></div>' },
})
// createStub() sẽ merge các stubs mới với các stubs mặc định

let wrapper
beforeEach(() => {
  wrapper = createShallow(TodoComponent, {
    stubs
  })
  // truyền options là stubs mới sẽ thay thế stubs cũ
})
```
## Kết luận

Đây là một phương pháp giúp chúng ta giảm thiểu số lượng code cần viết mỗi khi test một các đơn giản và ngắn gọn