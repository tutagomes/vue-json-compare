<template>
  <div class="alpaca-json">
    <slot v-if="isTheSameType">
      <p v-if="isObject" @click="parent=!parent" class="alpaca-f">
        <span v-show="!parent"></span>
      </p>
      <p v-else @click="parent=!parent" class="alpaca-f">
        [ <span v-show="!parent">... ]</span>
      </p>
      <tree v-show="parent" :oldData="oldData" :newData="newData" :merge="merge" :needKey="isObject" :objectType="isObject"></tree>
      <p v-if="isObject" v-show="parent"></p>
      <p v-else v-show="parent">]</p>
    </slot>
    <slot v-else>Tipos inconstantes, impossível comparar</slot>
  </div>
</template>

<script>
import { isObject, isArray, type } from './typeof.js'
import tree from './tree'
import parseData from './parsing'
const mergeArr = (arr1, arr2) => {
  let longer = []
  let merged = []
  if (arr1.length > arr2.length) {
    longer = arr1
  } else {
    longer = arr2
  }
  longer.forEach((item, index) => {
    if (arr1.hasOwnProperty(index) && arr2.hasOwnProperty(index)) {
      if (type(arr2[index]) === type(arr1[index])) {
        if (isArray(arr2[index])) {
          merged.push(mergeArr(arr1[index], arr2[index]))
        } else if (isObject(arr2[index])) {
          merged.push(mergeObj(arr1[index], arr2[index]))
        } else {
          merged.push(arr2[index])
        }
      } else {
        merged.push(arr2[index])
      }
    } else {
      if (arr2.hasOwnProperty(index)) {
        merged.push(arr2[index])
      } else {
        merged.push(arr1[index])
      }
    }
  })
  return merged
}
const mergeObj = (obj1, obj2) => {
  let key1 = Object.keys(obj1)
  let key2 = Object.keys(obj2)
  let mergeKey = [...new Set(key1.concat(key2))]
  let merged = Object.create(null)
  mergeKey.forEach((key) => {
    if (obj1.hasOwnProperty(key) && obj2.hasOwnProperty(key)) {
      if (type(obj2[key]) === type(obj1[key])) {
        if (isObject(obj2[key])) {
          merged[key] = mergeObj(obj1[key], obj2[key])
        } else if (isArray(obj2[key])) {
          merged[key] = mergeArr(obj1[key], obj2[key])
        } else {
          merged[key] = obj2[key]
        }
      } else {
        merged[key] = obj2[key]
      }
    } else {
      if (obj2.hasOwnProperty(key)) {
        merged[key] = obj2[key]
      } else {
        merged[key] = obj1[key]
      }
    }
  })
  return merged
}
export default {
  props: ['oldData', 'newData'],
  components: {
    tree
  },
  data () {
    return {
      isObject: true,
      parent: true,
      merge: []
    }
  },
  created () {
    this.getMergedData()
  },
  watch: {
    oldData (val) {
      this.getMergedData()
    },
    newData (val) {
      this.getMergedData()
    }
  },
  computed: {
    isTheSameType () {
      return type(this.oldData) === type(this.newData)
    }
  },
  methods: {
    getMergedData () {
      let mergeData = {}
      if (isObject(this.newData)) {
        this.isObject = true
      } else {
        this.isObject = false
      }
      if (isObject(this.newData) && isObject(this.oldData)) {
        mergeData = mergeObj(this.oldData, this.newData)
      }
      if (isArray(this.newData) && isArray(this.oldData)) {
        mergeData = mergeArr(this.oldData, this.newData)
      }
      this.merge = parseData(mergeData)
    }
  }
}
</script>

<style>
  .alpaca-hide {
    display: none;
  }
  .alpaca-json {
    padding: 10px 20px;
    border-radius: 5px;
    color: #032f62;
    background-color: #fafbfc;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 14px;
  }
  .alpaca-p {
    position: relative;
    margin: 0;
  }
  .alpaca-line {
    position: absolute;
    text-indent: 0;
    left: 5px;
    top: 2px;
    font-size: 12px;
    color: #808695;
    z-index: 5;
    user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    -webkit-user-select: none;
  }
  .alpaca-k {
    color: #5cadff;
  }
  .alpaca-f {
    cursor: pointer;
    margin: 0;
  }
  .alpaca-f:hover .alpaca-k {
    color: #19be6b;
    font-weight: bold;
  }
  .alpaca-number {
    color: #ae81ff;
  }
  .alpaca-string {
    color: #a6e22e;
  }
  .alpaca-boolean {
    color: #6F73FF;
  }
  .alpaca-null {
    color: #66d9ef;
  }
  .alpaca-undefined {
    color: #f92672;
  }
  .alpaca-del {
    position: relative;
    background-color: #ffeef0;
  }
  .alpaca-add {
    position: relative;
    background-color: #e6ffed;
  }
  .alpaca-upd {
    position: relative;
    background-color: #fffde6;
  }
  .alpaca-add:after {
    content: '+';
    position: absolute;
    left: -10px;
    top: 0px;
    text-indent: 0;
    color: #78ef9a;
  }
  .alpaca-del:after {
    content: '-';
    position: absolute;
    left: -10px;
    top: 0px;
    text-indent: 0;
    color: #f13e53;
  }
  .alpaca-upd:after {
    content: '*';
    position: absolute;
    left: -10px;
    top: 5px;
    text-indent: 0;
    color: #f1e234;
  }
  .alpaca-upd .alpaca-del, .alpaca-upd .alpaca-add {
    background-color: #fffde6;
  }
  .alpaca-upd .alpaca-del:after, .alpaca-upd .alpaca-add:after {
    content: '*';
    color: #f1e234;
  }
</style>
