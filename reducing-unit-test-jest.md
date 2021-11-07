# Viết test dễ dàng hơn

## Nhìn qua về unit test
Bài viết này phù hợp với những người đã biết về unit test, nên mình sẽ không giải thích các khái niệm căn bản nữa và sẽ đi thẳng vào những phần chính

Ví dụ có một component có sử dụng store và router
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
Thì lúc test chúng ta sẽ cần setup router và store để có thể test được component này
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
// Tạo store 

let wrapper
beforeEach(() => {
  wrapper = shallowMount(CreateTodo, {
    localVue,
    router,
    store
  })
  // Dùng shallowMount tạo một wrapper với router và store để sử dụng
})

afterEach(() => {
  wrapper.destroy()
})

describe('Test CreateTodo', () => {
  ...
})
```
Như vậy, khi mỗi lần test các component đều phải setup lại những phần này, nên bài viết này mình sẽ cấu trúc lại để có thể viết test một cách ngắn ngọn và đơn giản hơn

## Tổ chức thư mục
```
test
└── unit               
    ├── components
    |   └── CreateTodo.spec.js
    └── methods.js                 
```
``` javascript
// methods.js
import Vuex from 'vuex';
import VueRouter from 'vue-router';
import { shallowMount, mount, createLocalVue } from '@vue/test-utils';
import { routes } from '@/router/index';
import user from '@/store/modules/user';

export const localVue = createLocalVue();
localVue.use(VueRouter);
localVue.use(Vuex);

export const createStore = (customModule = {}) => new Vuex.Store({
  modules: {
    user,
    ...customModule,
  },
});

export const createRouter = (customRoute = []) => new VueRouter({
  routes: [
    ...routes,
    ...customRoute
  ],
});

export const createMock = ((mock = {}) => ({
  // thêm các mock mặc định ở đây
  $t: () => '', // ví dụ với i18n
  ...mock,
}));

export const createStub = ((customStub = {}) => ({
  // thêm các stub mặc định ở đây
  'ButtonComponent': { template: '<div></div>' },
  ...customStub,
}));

export const createDeep = (component, options = {}) => mount(component, {
  stubs: createStub(),
  store: createStore(),
  router: createRouter(),
  mocks: createMock(),
  localVue,
  ...options,
});

export const createShallow = (component, options = {}) => shallowMount(component, {
  stubs: createStub(),
  store: createStore(),
  router: createRouter(),
  mocks: createMock(),
  localVue,
  ...options,
});

```