<template>
  <div>
    <p class="alpaca-p" v-for="(item, index) in merge" :key="index" :style="getStyle" :class="getDiff(item, index)">
      <!--<span class="alpaca-line">{{ item.line }}</span>-->
      <slot v-if="isObject(item.type)">
        <p @click="toggle(item)" class="alpaca-f">
          <span v-if="needKey" class="alpaca-k">"{{ item.key }}": </span>
          <span class="alpaca-k">{</span>
          <span class="alpaca-k" v-show="!item.show">... } {{ comma(index) }}</span>
        </p>
        <self
          v-show="item.show"
          :oldData="getOldDataObj(item)"
          :newData="getNewDataObj(item)"
          :merge="item.value"
          :objectType="true"
          :showLeaf="false"
          :indent="indent+1"
          :needKey="true">
        </self>
        <p v-show="item.show" :style="getStyle" class="alpaca-p" :class="getDiff(item, index)">
          <span class="alpaca-line">{{ item.lastLine }}</span>
          <span class="alpaca-k">} {{ comma(index) }}</span>
       </p>
      </slot>
      <slot v-else-if="isArray(item.type)">
        <p @click="toggle(item)" class="alpaca-f">
          <span v-if="needKey" class="alpaca-k">{{ item.key }} - </span>
          <span class="alpaca-k">[</span>
          <span class="alpaca-k" v-show="!item.show">... ] {{ comma(index) }}</span>
        </p>
        <self
          v-show="item.show"
          :oldData="getOldDataArr(item)"
          :newData="getNewDataArr(item)"
          :merge="item.value"
          :showLeaf="false"
          :objectType="false"
          :indent="indent+1"
          :needKey="false">
        </self>
        <p v-show="item.show" :style="getStyle" class="alpaca-p" :class="getDiff(item, index)">
          <span class="alpaca-line">{{ item.lastLine }}</span>
          <span class="alpaca-k">] {{ comma(index) }}</span>
       </p>
      </slot>
      <slot v-else>
        <span v-if="needKey" class="alpaca-k">{{ item.key }} - </span>
        <span :class="getClass(item.type)">{{ item.value }}{{ comma(index) }}</span>
      </slot>
    </p>
  </div>
</template>

<script>
import * as check from './typeof.js'
export default {
  name: 'self',
  props: {
    oldData: {
      type: [Object, Array]
    },
    newData: {
      type: [Object, Array]
    },
    merge: {
      type: [Object, Array]
    },
    showLeaf: {
      default: true
    },
    indent: {
      default: 1
    },
    needKey: {
      default: true
    },
    objectType: {
      default: true
    }
  },
  computed: {
    getStyle () {
      return {textIndent: `${(this.indent + 1) * 20}px`}
    }
  },
  methods: {
    notTree (type) {
      return type !== 'Object' && type !== 'Array'
    },
    toggle (item) {
      item.show = !item.show
    },
    isObject (type) {
      return type === 'Object'
    },
    isArray (type) {
      return type === 'Array'
    },
    comma (index) {
      if (index === this.merge.length - 1) {
        return ''
      }
      return ','
    },
    getClass (type) {
      if (type === 'Number') {
        return 'alpaca-number'
      }
      if (type === 'String') {
        return 'alpaca-string'
      }
      if (type === 'Boolean') {
        return 'alpaca-boolean'
      }
      if (type === 'Undefined') {
        return 'alpaca-undefined'
      }
      if (type === 'Null') {
        return 'alpaca-null'
      }
      // 其他不改变颜色
      return ''
    },
    getOldDataObj (item) {
      if (this.oldData.hasOwnProperty(item.key)) {
        if (check.isObject(this.oldData[item.key])) {
          return this.oldData[item.key]
        }
      }
      return {}
    },
    getNewDataObj (item) {
      if (this.newData.hasOwnProperty(item.key)) {
        if (check.isObject(this.newData[item.key])) {
          return this.newData[item.key]
        }
      }
      return {}
    },
    getOldDataArr (item) {
      if (this.oldData.hasOwnProperty(item.key)) {
        if (check.isArray(this.oldData[item.key])) {
          return this.oldData[item.key]
        }
      }
      return []
    },
    getNewDataArr (item) {
      if (this.newData.hasOwnProperty(item.key)) {
        if (check.isArray(this.newData[item.key])) {
          return this.newData[item.key]
        }
      }
      return []
    },
    getDiff (item, index) {
      let oldData = this.oldData
      let newData = this.newData
      if (this.objectType) {
        //Objecto
        if (oldData.hasOwnProperty(item.key) && newData.hasOwnProperty(item.key)) {
          // Tipos Diferentes
          if (check.type(oldData[item.key]) !== check.type(newData[item.key])) {
            return 'alpaca-upd'
          } else {
            // Mesmo tipo
            if (!check.isArray(oldData[item.key]) && !check.isObject(oldData[item.key])) {
              if (oldData[item.key] !== newData[item.key]) {
                return 'alpaca-upd'
              }
            }
          }
        }
        if (oldData.hasOwnProperty(item.key) && !newData.hasOwnProperty(item.key)) {
          return 'alpaca-del'
        }
        if (!oldData.hasOwnProperty(item.key) && newData.hasOwnProperty(item.key)) {
          return 'alpaca-add'
        }
        // Sem alterações
        return 'alpaca-hide'
      }
      // 数组，对比下标
      if (oldData.hasOwnProperty(item.key) && newData.hasOwnProperty(item.key)) {
        // Tipos Diferentes
        if (check.type(oldData[item.key]) !== check.type(newData[item.key])) {
          return 'alpaca-upd'
        } else {
          // Mesmo tipo
          if (!check.isObject(oldData[item.key]) && !check.isArray(oldData[item.key])) {
            if (oldData[item.key] !== newData[item.key]) {
              return 'alpaca-upd'
            }
          }
        }
      }
      if (oldData.hasOwnProperty(item.key) && !newData.hasOwnProperty(item.key)) {
        return 'alpaca-del'
      }
      if (!oldData.hasOwnProperty(item.key) && newData.hasOwnProperty(item.key)) {
        return 'alpaca-add'
      }
      // Sem alterações
      return 'alpaca-hide'
    }
  }
}
</script>
