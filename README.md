# Vue-element
The frontend project about high imitated element app.

# sell

> sell app

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader)

##精度丢失优化方法

``` bash
// 解决精度丢失问题(加减乘除)
    // 加法
    accAdd (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let times = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
        len1 = 0
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
        len2 = 0
      }
      times = Math.pow(10, Math.max(len1, len2))
      result = (arg1 * times + arg2 * times) / times
      return result
    },
    // 减法
    accSub (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let times = 0
      let maxPrecision = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
        len1 = 0
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
        len2 = 0
      }
      times = Math.pow(10, Math.max(len1, len2))
      maxPrecision = (len1 > len2) ? len1 : len2
      result = ((arg1 * times + arg2 * times) / times).toFixed(maxPrecision)
      return result
    },
    // 乘法
    accMul (arg1, arg2) {
      let s1 = arg1.toString()
      let s2 = arg2.toString()
      let times = 0
      let result = 0
      try {
        times += s1.split('.')[1].length
      } catch (e) {
      }
      try {
        times += s2.split('.')[1].length
      } catch (e) {
      }
      result = Number(s1.replace('.', '')) * Number(s2.replace('.', '')) / Math.pow(10, times)
      return result
    },
    // 除法
    accDiv (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let r1 = 0
      let r2 = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
      }
      r1 = Number(arg1.toString().replace('.', ''))
      r2 = Number(arg2.toString().replace('.', ''))
      result = this.accMul((r1 / r2), Math.pow(10, len2 - len1))
      return result
    }
```