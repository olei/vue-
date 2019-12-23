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
      // 目标
      function Subject () {
        this.list = []
      }

      Subject.prototype.add = function (observer) {
        this.list.push(observer)
      }

      Subject.prototype.notify = function () {
        this.list.forEach(item => {
          item.updata()
        })
      }
      // 观察者
      function Watcher (node, name, data) {
        this.node = node
        this.name = name
        this.data = data
      }

      Watcher.prototype.updata = function () {
        if (this.node.nodeType === 1) this.node.value = this.data[this.name]
        else if (this.node.nodeType === 3) this.node.nodeValue = this.data[this.name]
      }

      function Mvvm(options) {
        this.$el = options.el
        this.$data = options.data
        this.subject = new Subject()
        this.bindData()
        this.$el.appendChild(this.toFragment())
      }

      Mvvm.prototype.toFragment = function () {
        const frag = document.createDocumentFragment()
        let first
        while (first = this.$el.firstChild) {
          this.factory(first)
          frag.appendChild(first)
        }
        return frag
      }

      Mvvm.prototype.factory = function (node) {
        let name
        if (node.nodeType === 1) {
          name = node.getAttribute('v-model')
          node.addEventListener('input', e => {
            this.$data[name] = e.target.value
          })
          node.value = this.$data[name]
        } else if (node.nodeType === 3) {
          name = node.nodeValue.replace(/\{\{(.*)\}\}/, '$1').trim()
          node.nodeValue = this.$data[name]
        }
        name && this.subject.add(new Watcher(node, name, this.$data))
      }

      Mvvm.prototype.bindData = function () {
        const that = this
        Object.entries(this.$data).forEach(item => {
          let val = item[1]
          Object.defineProperty(this.$data, item[0], {
            get () {
              return val
            },
            set (newVal) {
              val = newVal
              that.subject.notify()
              console.log(`run setter: ${val}`)
            }
          })
        })
      }

      const test = new Mvvm({
        el: document.getElementById('app'),
        data: {
          text: 'abc',
          text1: '123'
        }
      })
</script>
</body>
</html>
