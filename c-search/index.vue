<template>
  <div class="c-search">
    <Select
      v-model="s_arr"
      :placeholder="placeholder"
      :max-tag-count="0"
      multiple
      filterable
      label-in-value
      @on-select="selectOption"
      @on-open-change="openChange"
    >
      <Option
        v-for="i in selectedList"
        :value="i.value"
        :key="i.value"
        :title="i.label"
        >{{ i.label }}</Option
      >
      <Option
        value="all"
        key="all"
        v-if="trans_unselList.length > 0"
        style="padding: 0px"
      >
        <p
          @click="
            selectAllFun(
              s_arr.length !== unselList.length + selectedList.length
            )
          "
        >
          {{
            s_arr.length !== unselList.length + selectedList.length
              ? "全选"
              : "取消全选"
          }}
        </p>
      </Option>
      <Option
        v-for="item in unselList"
        :value="item.value"
        :key="item.value"
        :title="item.label"
        >{{ item.label }}</Option
      >
    </Select>
  </div>
</template>
<script>
export default {
  data () {
    return {
      unselList: [], // 未选数组
      selectedList: [], // 已选数组
      s_arr: [], // v-model绑定选项数组
      add_tag: true, // 新增选项标记
      allList: [], // 全选v-model绑定选项数组
      init_list: [] // 全选已选数组
    }
  },
  props: {
    placeholder: {
      type: String,
      default () {
        return ''
      }
    },
    trans_unselList: {
      type: Array,
      default () {
        return []
      }
    }
  },
  methods: {
    selectAllFun (isAll) {
      const self = this
      setTimeout(() => {
        self.s_arr = []
        if (isAll) {
          self.selectedList = []
          self.init_list.forEach(item => {
            // 采用这种方式的拷贝，不会只是拷贝了对象的别名（只拷贝别名会导致不期望的被改动）
            self.selectedList.push(item)
          })
          self.unselList = []
          self.allList.forEach(item => {
            self.s_arr.push(item)
          })
        } else {
          self.selectedList = []
          self.unselList = []
          self.init_list.forEach(item => {
            self.unselList.push(item)
          })
          self.s_arr = []
        }
      }, 0)
    },
    add_or_del (o) {
      const self = this
      try {
        self.selectedList.forEach(function (item) {
          if (item.value === o.value) {
            self.add_tag = false
            throw new Error('')
          }
        })
      } catch (e) {
        return ''
      }
      self.add_tag = true
      return ''
    },
    selectOption (o) {
      const self = this
      if (o.value !== 'all') {
        setTimeout(() => {
          self.add_or_del(o)
          if (self.add_tag) {
            try {
              self.unselList.forEach(function (item, index) {
                if (item.value === o.value) {
                  self.unselList.splice(index, 1)
                  throw new Error('')
                }
              })
            } catch (e) {
              console.log(e)
            }
            self.selectedList.push(o)
          } else {
            try {
              self.selectedList.forEach(function (item, index) {
                if (item.value === o.value) {
                  self.selectedList.splice(index, 1)
                  throw new Error('')
                }
              })
            } catch (e) {
              console.log(e)
            }
            self.unselList.push(o)
          }
        }, 100)
      }
    },
    openChange (isopen) {
      console.log('isopen: ' + isopen)
      if (this.selectedList.length > 0 && !isopen) {
        console.log('this.selectedList： ' + this.selectedList)
      }
    },
    allList_setValue () {
      const self = this
      self.unselList.forEach(temp => {
        self.allList.push(temp.value)
        self.init_list.push(temp)
      })
    }
  },
  created () {
    const self = this
    setTimeout(() => {
      this.unselList = this.trans_unselList
      self.allList_setValue()
    }, 500)
  }
}
</script>
<style lang="stylus" scoped>
@import '~common/styles/c-common';

.ivu-select-dropdown-list {
  width: 130px
}

.ivu-select-dropdown-list li {
  width: 120px
  height: 35px;
  line-height: 35px;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  box-sizing: border-box;
  padding: 0 6px;
}
.ivu-select {
    width: 130px;
}
/deep/ .ivu-select-input {
  width: 60px !important
  overflow: hidden
}
p {
  background: $cBlue;
  color: #fff;
  text-align: center;
  font-weight: 600;
}
/deep/ .ivu-tag {
    display: inline-block;
    height: 22px;
    line-height: 22px;
    margin: 2px 0px 2px 0;
    padding: unset;
    border: unset;
    border-radius: unset;
    background: unset;
    font-size: 12px;
    vertical-align: middle;
    opacity: 1;
    overflow: hidden;
}
.ivu-select-multiple .ivu-select-item-selected:after {
    top: -10px;
    left: 0px;
    right: unset;
}
</style>
