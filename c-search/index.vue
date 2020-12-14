<template>
  <div class="c-search">
    <Select
      v-model="s_arr"
      :disabled="isdisabled"
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
import { deepCopy } from '@/common/js/utils'
import { atrToNum_handle } from '@/common/js/c_common'
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
    },
    isdisabled: {
      type: Boolean,
      default () {
        return false
      }
    },
    trans_selList: {
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
              // console.log(e)
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
              // console.log(e)
            }
            self.unselList.push(o)
          }
        }, 100)
      }
    },
    openChange (isopen) {
      if (!isopen) {
        var res = this.backList_handle(this.selectedList)
        this.$emit('func', res)
      }
    },
    // 返回选项列表处理
    backList_handle (list) {
      var res = []
      list.forEach(item => {
        res.push(item.value)
      })
      return res
    },
    allList_setValue () {
      const self = this
      self.unselList.forEach(temp => {
        self.allList.push(temp.value)
        self.init_list.push(temp)
      })
    },
    // 拿到的选项处理
    selList_handle (trans, unsel) {
      const self = this
      setTimeout(() => {
        if (trans && trans.length > 0) {
          const sel = []
          const un_sel = []
          unsel.forEach(function (item, index) {
            if (item.value && trans.indexOf(item.value) !== -1) {
              sel.push(item)
            } else if (item.value && trans.indexOf(item.value) === -1) {
              un_sel.push(item)
            }
          })
          self.selectedList = sel
          // s_arr保存的仅是value
          self.selectedList.forEach(item => {
            self.s_arr.push(item.value)
          })
          self.unselList = un_sel
          const res = this.backList_handle(self.selectedList)
          this.$emit('func', res)
        } else if (trans && trans.length === 0) {
          self.selectedList = []
          self.unselList = []
          self.init_list.forEach(item => {
            self.unselList.push(item)
          })
          self.s_arr = []
          const res = this.backList_handle(self.selectedList)
          this.$emit('func', res)
        }
      }, 0)
    }
  },
  watch: {
    trans_selList (n, o) {
      const trans = n.length === 0 ? [] : atrToNum_handle(n)
      this.selectAllFun(false)
      this.selList_handle(trans, this.init_list)
    },
    trans_unselList (n, o) {
      if (n && n.length > 0) {
        this.unselList = deepCopy(n)
        this.allList_setValue()
      }
    }
  },
  created () {
    this.unselList = deepCopy(this.trans_unselList)
    this.allList_setValue()
    setTimeout(() => {
      if (this.trans_selList && this.trans_selList.length > 0) {
        const n = this.trans_selList
        const trans = n.length === 0 ? [] : atrToNum_handle(n)
        this.selectAllFun(false)
        this.selList_handle(trans, this.init_list)
      }
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
/deep/ .ivu-select-disabled {
  .ivu-tag {
    display: none;
  }
}
.ivu-select-multiple .ivu-select-item-selected:after {
    top: -10px;
    left: 0px;
    right: unset;
}
</style>

