# Vue-父子组件传值

>vue 中的父子组件传值，值得注意的是要遵守单向数据流原则。所谓单向数据流原则，简单的说就是父组件的数据可以传递给子组件，子组件也可以正常获取并使用由父组件传过来的数据；但是，子组件中不能直接修改父组件传过来的数据，必须要向父组件传递一个事件来父组件需要修改数据，即通过子组件的操作，在父组件中修改数据；这就是单项数据流。

## 一、普通方式

普通方式即为父组件使用自定义属性向子组件传值，通过自定义事件接收事件；子组件通过 props 接收数据，子组件通过 $emit 向父组件传递自定义事件。

实现代码如下：

子组件：

```vue
<template>
  <div>
    我是子组件：
    <input type="text" :value="msg" @input="changeValFn" />
  </div>
</template>

<script>
export default {
  name: 'child',
  props: ['msg'],
  methods: {
    changeValFn(e) {
      this.$emit('changeMsg', e.target.value)
    },
  },
}
</script>
```

父组件：

```vue
<template>
  <div class="parent">
    <h1>我是父组件：{{ msg }}</h1>
    <child :msg="msg" @changeMsg="changeMsgFn"></child>
  </div>
</template>

<script>
import child from './child'
export default {
  name: 'parent',
  components: {
    child,
  },
  data() {
    return {
      msg: 'hello!',
    }
  },
  methods: {
    changeMsgFn(value) {
      this.msg = value
    },
  },
}
</script>
```

## 二、v-model 方式

`v-model`是语法糖， `v-model`等价于给一个 input 框提供了 :value 属性以及 @input 事件，但是如果每次使用 input 框，都需要提供value 和 input 事件比较麻烦，所以使用`v-model`。

实现代码：

子组件：

```vue
<template>
  <div>
    我是子组件：
    <input type="text" :value="value" @input="changeValFn" />
  </div>
</template>

<script>
export default {
  name: 'child',
  props: ['value'],
  methods: {
    changeValFn(e) {
      this.$emit('input', e.target.value)
    },
  },
}
</script>
```

父组件：

```vue
<template>
  <div class="parent">
    <h1>我是父组件：{{ msg }}</h1>
    <child v-model="msg"></child>
  </div>
</template>

<script>
import child from './child'
export default {
  name: 'parent',
  components: {
    child,
  },
  data() {
    return {
      msg: 'hello!',
    }
  },
}
</script>

```

**注意：**

**由于这里子组件里使用的是 input type=‘text’ 的控件，所以定义组件的时候，子组件 props 接收的值叫 value，子传父触发的事件名叫 input。**

这里如果像在子组件中自定义接收的数据名与事件名时，需要在与 props 同级的地方配置 model。

具体实现如下：

```vue
<template>
  <div>
    我是子组件：
    <input type="text" :value="msg" @input="changeValFn" />
  </div>
</template>

<script>
export default {
  name: 'child',
  props: ['msg'],
  model: {
    prop: 'msg',
    event: 'changeMsg',
  },
  methods: {
    changeValFn(e) {
      this.$emit('changeMsg', e.target.value)
    },
  },
}
</script>

```

这样就可以把接收到的数据 value 替换成 msg，把自定义事件 input 替换成 changeMsg。

## 三、sync 修饰符方式

.sync 修饰符是 vue 2.3.0+ 新增的特性（但是在 vue3.0 中被废弃）

实现代码如下：

子组件：

（以 `update:myPropName` 的模式触发事件）

```vue
<template>
  <div>
    我是子组件：
    <input type="text" :value="msg" @input="changeValFn" />
  </div>
</template>

<script>
export default {
  name: 'child',
  props: ['msg'],
  methods: {
    changeValFn(e) {
      this.$emit('update:msg', e.target.value)
    },
  },
}
</script>
```

父组件：

（以`v-bind:msg.sync="msg"`的模式传递/接收数据）

```vue
<template>
  <div class="parent">
    <h1>我是父组件：{{ msg }}</h1>
    <!-- <child v-bind:msg.sync="msg"></child> -->
    <child :msg.sync="msg"></child>
  </div>
</template>

<script>
import child from './child'
export default {
  name: 'parent',
  components: {
    child,
  },
  data() {
    return {
      msg: 'hello!',
    }
  },
}
</script>
```

**注意:**

**带有 .sync 修饰符的 v-bind 不能和表达式一起使用 (例如 v-bind:msg.sync=”msg + ‘!’” 是无效的)。取而代之的是，你只能提供你想要绑定的 property 名，类似 v-model。**

**将 v-bind.sync 用在一个字面量的对象上，例如 v-bind.sync=”{ msg: msg }”，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。**

## 四、补充

#### vue3.0中父子组件传值

vue3.0中废弃了 `.sync` ，并把所有使用 `.sync` 的部分并将其替换为 `v-model:`

```vue
<ChildComponent :title.sync="pageTitle" />

<!-- 替换为 -->

<ChildComponent v-model:title="pageTitle" />

```

实现代码如下：

子组件：

```vue
<template>
  <label>我是子组件：</label>
  <input type="text" :value="msg" @input="changeValFn" />
</template>

<script>
export default {
  name: 'child',
  props: ['msg'],
  methods: {
    changeValFn(e) {
      this.$emit('update:msg', e.target.value)
    },
  },
}
</script>
```

父组件：

```vue
<template>
  <h1>我是父组件：{{ msg }}</h1>
  <child v-model:msg="msg"></child>
</template>

<script>
import child from './components/child'
export default {
  name: 'App',
  components: {
    child,
  },
  data() {
    return {
      msg: 'hello!',
    }
  },
}
</script>
```

注意：

子组件中对于所有不带参数的 `v-model`，请确保分别将 prop 和 event 命名更改为 `modelValue` 和 `update:modelValue`
