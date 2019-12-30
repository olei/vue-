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
    function Dep() {
      this.observerList = []
    }
    Dep.prototype.add = function(observer) {
      this.observerList.push(observer)
    }
    Dep.prototype.notify = function(watch) {
      this.observerList.forEach(item => {
        item.updata && item.updata()
      })
    }

    function Observer(node, name, data) {
      this.node = node
      this.name = name
      this.data = data
    }
    Observer.prototype.updata = function() {
      if (this.node.nodeType === 1) {
        this.node.value = this.data[this.name]
      } else if (this.node.nodeType === 3) {
        this.node.nodeValue = this.data[this.name]
      }
    }

    function Mvvm(options) {
      this.$data = options.data
      this.$el = document.getElementById(options.el)
      this.subject = new Dep()
      this.bindData(this.$data)
      const dom = this.toFragment()
      this.$el.appendChild(dom)
      this.$watch = options.watch
    }

    Mvvm.prototype.toFragment = function() {
      const frag = document.createDocumentFragment()
      let first
      while (first = this.$el.firstChild) {
        frag.appendChild(first)
        this.factory(first)
      }
      return frag
    }

    Mvvm.prototype.factory = function(node) {
      let name
      if (node.nodeType === 1) {
        name = node.getAttribute('v-modal')
        node.value = this.$data[name]
        node.addEventListener('input', e => {
          this.$data[name] = e.target.value
        })
      } else if (node.nodeType === 3) {
        name = node.nodeValue.replace(/\{\{(.*)\}\}/, '$1').trim()
        node.nodeValue = this.$data[name]
      }
      name && this.subject.add(new Observer(node, name, this.$data))
    }

    Mvvm.prototype.watch = function(name, val, oldVal) {
      this.$watch[name] && this.$watch[name](val, oldVal)
    }

    Mvvm.prototype.bindData = function(data) {
      Object.entries(data).forEach(item => {
        let [key, val] = item
        Object.defineProperty(data, key, {
          get: () => val,
          set: newVal => {
            const oldVal = val
            val = newVal
            this.watch(key, val, oldVal)
            this.subject.notify()
          }
        })
        Array.isArray(val) && this.bindArray(val, key)
      })
    }

    Mvvm.prototype.bindArray = function(arr, key) {
      const defArray = Object.create(Array.prototype);
      ['push', 'pop'].forEach(item => {
        Object.defineProperty(defArray, item, {
          enumerable: true,
          configurable: true,
          value: (...args) => {
            const oldVal = arr.slice()
            Array.prototype.push.apply(arr, args)
            this.subject.notify()
            this.watch(key, arr, oldVal)
          }
        })
      })
      arr.__proto__ = defArray
    }

    const app = new Mvvm({
      el: 'app',
      data: {
        text: 'abc',
        arr: [1, 2]
      },
      watch: {
        text (val, oldVal) {
          console.log(val, oldVal)
        },
        arr (val, oldVal) {
          console.log(val, oldVal)
        },
      }
    })
</script>
</body>
</html>
