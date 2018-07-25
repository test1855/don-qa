<template>
  <div class="don-qa-question-select" v-if="!fill">
    <pku-tab
      :list="list">
      <div slot="slot_0">
        <don-qa-question-wrap label="题目序号" v-if="!fill" ref="t1">
          <pku-input
            class="wrap-input"
            ref="input"
            @change="onSnEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="题目标题" v-if="!fill" ref="t2">
          <pku-input
            class="wrap-input"
            ref="input"
            @change="onTitleEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap>
          <pku-button
            class="btn-warning"
            value="增加选项"
            @callback="onClickEventHadnler"></pku-button>
        </don-qa-question-wrap>
        <transition-group name="list" tag="div" ref="t3">
          <don-qa-question-wrap 
            class="list-item"
            v-for="(item, id) in selects"
            :key="id"
            :label="selectID(id)">
            <pku-input class="question-input" placeholder="选项序号" @change="onSelectEventHandler($event, id, 'sn')"></pku-input>
            <pku-input class="question-input" placeholder="选项名称" @change="onSelectEventHandler($event, id, 'val')"></pku-input>
            <pku-switch 
              class="switch-online"
              ableText="无"
              disableText="请注明"
              :message="selectType(id) + '#' + new Date().valueOf()"
              @callback="onSelectEventHandler($event, id, 'checktype')"></pku-switch>
          </don-qa-question-wrap>
        </transition-group>
        <don-qa-question-wrap>
          <pku-button
            class="btn-primary"
            value="保存"
            :class="{'btn-primary': true, 'btn-disabled': !inputSn}"
            :disabled="!inputSn"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      </div>
      <div slot="slot_1"></div>
    </pku-tab>
  </div>
  <div class="don-qa-question-select" v-else>
     <pku-radio
        importKey="name"
        exportKey="value"
        :disabled="false"
        :message="options"></pku-radio>
  </div>
</template>

<script>
export default {
  name: 'donQuestionSelect',
  props: {
    fill: {
      type: Boolean,
      default: false
    },
    res: {
      type: Object,
      default () {
        return null
      }
    },
    opt: {
      type: Object
    },
    options: {
      type: Array,
      default () {
        return [
          { name: '是', value: 1 },
          { name: '无法判断', value: 9 }
        ]
      }
    }
  },
  data () {
    return {
      list: ['问题设置'],
      selects: [
        { sn: '', val: '', checktype: ''}
      ],
      inputSn: undefined,
      inputTitle: undefined,
      selecttype: []
    }
  },
  mounted () {
    if (this.fill && this.res) {
      this.res.quesOptions.forEach((item, index) => {
        if (item !== null) {
          this.$children[0].$data.value = index
        }
      })
    } else {
      this.$refs.t1.$children[0].$data.value = this.opt.quesText
      this.$refs.t2.$children[0].$data.value = this.opt.quesSn
      this.selects = []
      console.log(this, 2)
      this.opt.quesOptions.forEach((item, id) => {
        this.selects.push({
          sn: item,
          val: this.opt.quesOptionTexts[id]
        })
      })
      this.$nextTick(() => {
        this.opt.quesOptions.forEach((item, id) => {
          this.$refs.t3.$children[id].$children[0].$data.value = item
          this.$refs.t3.$children[id].$children[1].$data.value = this.opt.quesOptionTexts[id]
        })
      })
    }
  },
  methods: {
    selectID (val) {
      return '选项' + String(val + 1)
    },
    selectType (val) {
      if (val < this.selecttype.length) {
        return this.selecttype[val]
      } else {
        this.selecttype.push(true)
        return this.selecttype[val]
      }
    },
    onClickEventHadnler () {
      this.selects.push({
        sn: '',
        val: '',
        checktype: ''
      })
    },
    onSnEventHandler (val) {
      this.inputSn = val
    },
    onTitleEventHandler (val) {
      this.inputTitle = val
    },
    onSelectEventHandler (val, index, type) {
      if (type === 'checktype') {
        this.selecttype[index] = val
      }
      this.selects[index][type] = val
    },
    onSubmitEventHandler () {
    //   let content = this.title.filter(item => item.questionId === this.select)
      let value = []
      let sns = []
      let ctype = []
      this.selects.forEach(item => {
        console.log(item)
        value.push(item.val)
        sns.push(item.sn)
        if (item.checktype === '') {
          item.checktype = true
        }
        ctype.push(item.checktype)
      })
      this.$emit('callback', {
        'QuesOptionValues': sns.toString(),
        'QuesOptionTexts': value.toString(),
        'optionWithoutInput': ctype.toString(),
        // 'questionID': this.select,
        'questionContent': this.inputTitle,
        'questionSn': this.inputSn,
        'QuesType': '0001'
      })
    }
  }
}
</script>
<style scoped>
.wrap-input, .wrap-select {
  margin-bottom: 20px;
}
div.don-qa-question-select >>> .don-qa-question-wrap label {
  font-size: 14px;
}
.question-input {
  width: 48%;
}
</style>
