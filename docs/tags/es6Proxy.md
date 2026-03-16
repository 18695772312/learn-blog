---
title: 认识Proxy
date: 2025-06-03
categories: javaScript
tags:
- js
---
## 认识Proxy
### 基本语法
```js
    const proxy = new Proxy(target, handler);
    // target：要代理的目标对象

    // handler：也是一个对象，定义拦截行为，如果没有设置任何拦截，那就等同于直接通向原对象。
```
### Proxy 实例的方法

#### get()

get 方法用于拦截某个属性的读取操作。

get 方法可以继承，即拦截操作定义在原型对象上时，读取目标对象继承的属性时，拦截会生效。

```js
const person = {
  name: '张三',
  age: 30
};

const proxy = new Proxy(person, {
  get(target, property, receiver) {
    console.log(`读取属性: ${property}`);
    if (property in target) {
      return target[property];
    } else {
      return '属性不存在';
    }
  }
});

console.log(proxy.name);  // 读取属性: name → 张三
console.log(proxy.gender); // 读取属性: gender → 属性不存在
```
#### set()
set 方法用于拦截某个属性的赋值操作。

如果目标对象的某个属性不可写也不可配置，那么 set 不得改变这个属性的值，只能返回同样的值，否则会报错。
```js
const validator = {
  set(target, property, value) {
    if (property === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('年龄必须是整数');
      }
      if (value < 0 || value > 150) {
        throw new RangeError('年龄必须在0-150之间');
      }
    }
    target[property] = value;
    return true; // 表示设置成功
  }
};

const person = new Proxy({}, validator);
person.age = 25;  // 正常
// person.age = '25';  // TypeError: 年龄必须是整数
// person.age = 200;   // RangeError: 年龄必须在0-150之间
```
### 常用

1. 数据验证
   ```js
   
   function createValidator(target, validators) {
        return new Proxy(target, {
            set(target, property, value) {
            if (validators[property]) {
                validators[property](value);
            }
            target[property] = value;
            return true;
            }
        });
    }

    const user = createValidator({}, {
    name: val => {
        if (typeof val !== 'string') throw new Error('名字必须是字符串');
    },
    age: val => {
        if (val < 0) throw new Error('年龄不能为负数');
    }
    });

    user.name = '小明';  // 正常
    // user.name = 123;   // Error: 名字必须是字符串
   ```
2. 访问控制与权限管理
```js
// 基于角色的权限控制
function createProtectedObject(obj, userRole) {
  const permissions = {
    admin: ['read', 'write', 'delete'],
    editor: ['read', 'write'],
    viewer: ['read']
  };

  return new Proxy(obj, {
    get(target, prop) {
      if (prop.startsWith('_')) {
        throw new Error('无权访问私有属性');
      }
      return target[prop];
    },
    set(target, prop, value) {
      if (!permissions[userRole].includes('write')) {
        throw new Error('无权修改数据');
      }
      target[prop] = value;
      return true;
    },
    deleteProperty(target, prop) {
      if (!permissions[userRole].includes('delete')) {
        throw new Error('无权删除数据');
      }
      delete target[prop];
      return true;
    }
  });
}

const data = { name: '项目', _secret: '机密' };
const viewer = createProtectedObject(data, 'viewer');
console.log(viewer.name);    // 项目
// console.log(viewer._secret); // Error: 无权访问私有属性
// viewer.name = '新项目';       // Error: 无权修改数据
```
3. 数据绑定与响应式系统
```js
// 简单的响应式系统
function createReactive(obj, callback) {
  return new Proxy(obj, {
    set(target, prop, value) {
      const oldValue = target[prop];
      target[prop] = value;
      callback(prop, value, oldValue);
      return true;
    }
  });
}

// Vue3 风格的响应式简化版
const data = { count: 0, message: 'hello' };
const reactive = createReactive(data, (prop, newVal, oldVal) => {
  console.log(`属性 ${prop} 从 ${oldVal} 变为 ${newVal}`);
  updateUI(prop, newVal); // 更新界面
});

reactive.count++;  // 属性 count 从 0 变为 1
reactive.message = 'world'; // 属性 message 从 hello 变为 world
```
