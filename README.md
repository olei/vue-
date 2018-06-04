# vue双向绑定

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>mvvm</title>
</head>
<body>
<div id="app">
  <input type="text" v-model="text">
  {{ text }}
  <input type="text" v-model="text1">
  {{ text1 }}
</div>
<script>

function observe (data) {
  Object.keys(data).forEach(key => {
    let val = data[key];
    Object.defineProperty(data, key, {
      get () {
        return val;
      },
      set (newVal) {
        if (newVal === val) return;
        val = newVal;
        Dep.trigger();
      }
    })
  });
}

function toFragment (node, data) {
  const flag = document.createDocumentFragment();
  let child;
  while(child = node.firstChild) {
    factory(child, data);
    flag.appendChild(child);
  }
  return flag
}

function factory (node, data) {
  const reg = /\{\{(.*)\}\}/;
  if (node.nodeType === 1) {
    const attr = node.attributes;
    for (let i = 0, l = attr.length; i < l; i++) {
      if (attr[i].nodeName === 'v-model') {
        const name = attr[i].nodeValue
        node.addEventListener('input', e => {
          data[name] = e.target.value;
        });
        new Watcher(node, data, name, 'input');
        node.removeAttribute('v-model');
      }
    }
  } else if (node.nodeType === 3 && reg.test(node.nodeValue)) {
    let name = RegExp.$1;
    name = name.trim();
    new Watcher(node, data, name, 'text');
  }
}

//订阅者
class Watcher {
  constructor(node, data, name, type) {
    this.node = node;
    this.data = data;
    this.name = name;
    this.type = type;
    this.update();
    Dep.subs.push(this);
  }

  update () {
    if (this.type === 'text') {
      this.node.nodeValue = this.data[this.name];
      return;
    }
    if (this.type === 'input') {
      this.node.value = this.data[this.name];
    }
  }
}

// 发布者
const Dep = {
  subs: [],
  add (obj) {
    this.subs.push(obj)
  },
  trigger () {
    this.subs.forEach(item => {
      item.update()
    })
  }
}

// mvvm 构造器
function Mvvm(opts) {
  this.data = opts.data;
  observe(this.data);
  const el = document.getElementById(opts.el);
  const dom = toFragment(el, this.data);
  el.appendChild(dom);
}

let v = new Mvvm({
  el: 'app',
  data: {
    text: 'hello word',
    text1: 'test'
  }
});
</script>
</body>
</html>
```

### defineProperty

```
user = {}
nameValue = 'Joe';
Object.defineProperty(user, 'name', {
  get: function(){ return nameValue }, 
  set: function(newValue){ nameValue = newValue; },
  configurable: true//to enable redefining the property later
});
user.name //Joe 
user.name = 'Bob'
user.name //Bob
nameValue //Bob
```
