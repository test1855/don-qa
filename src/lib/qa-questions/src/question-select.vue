<template>
  <div class="don-qa-question-select" v-if="!fill">
    <!-- <pku-tab
      :list="list"> -->
      <!-- <div slot="slot_0"> -->
        <don-qa-question-wrap label="题目序号" v-if="!fill" ref="t1">
          <pku-input
            class="wrap-input"
            ref="input"
            @change="onSnEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="问题组">
          <pku-select
            class="wrap-select"
            selected="选择问题组"
            ref="group"
            importKey="quesText"
            exportKey="questionId"
            :list="array"
            @callback="onGroupEventHandler"></pku-select>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="题干">
          <component
            class="wrap-select"
            selected="选择问题"
            :is="reWriteName"
            importKey="quesText"
            exportKey="questionId"
            ref="host"
            :style="{width: reWrite ? '80%' : 'unset'}"
            :list="questions"
            :html="true"
            @callback="onQuestionEventHandler">
          </component>
          <pku-edit v-if="opt || (reWrite && this.question )" :showFunc="showFunc" :cancelFunc="cancelFunc" :submitFunc="submitFunc" ref="edit"></pku-edit>
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
              disableText="其他请注明"
              :message="selectType(id) + '#' + new Date().valueOf()"
              @callback="onSelectEventHandler($event, id, 'checktype')"></pku-switch>
          </don-qa-question-wrap>
        </transition-group>
        <don-qa-question-wrap label="核查员注意">
          <pku-input
            class="wrap-input"
            ref="attention"
            @change="onAttentionEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap>
          <pku-button
            class="btn-primary"
            value="保存"
            :class="{'btn-primary': true, 'btn-disabled': !inputSn}"
            :disabled="!inputSn"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      <!-- </div> -->
      <!-- <div slot="slot_1"></div> -->
    <!-- </pku-tab> -->
  </div>
  <div class="don-qa-question-select" v-else>
     <pku-radio
        importKey="name"
        exportKey="value"
        :disabled="false"
        :message="options"></pku-radio>
    <div class="don-qa-question-select" v-for="option in options" :key="option.value">
      <pku-input class="wrap-input" :disabled="!option.quesOptionsWithInput"></pku-input>
    </div>
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
          { name: '是', value: 1, quesOptionsWithInput: false },
          { name: '无法判断', value: 9, quesOptionsWithInput: false }
        ]
      }
    },
    reWrite: {
      type: Boolean,
      default: false
    },
    array: {
      type: Array,
      default () {
        return ['题目1', '题目2', '题目3']
      }
    },
    questions: {
      type: Array,
      default () {
        return ['题目1', '题目2', '题目3']
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
      selecttype: [],
      reWriteName: 'pkuSelect',
      tmpSelected: '',
      attention: ''
    }
  },
  mounted () {
    if (this.fill && this.res) {
      this.$children.forEach((item, index) => {
        if (index !== 0) {
          item.value = this.res.quesOptions[index-1]
        }
      })
      this.res.quesOptions.forEach((item, index) => {
        if (item !== null) {
          this.$children[0].$data.value = index
        }
      })
    } else {
      this.$refs.t1.$children[0].$data.value = this.opt.quesText
      this.$refs.t2.$children[0].$data.value = this.opt.quesSn
      this.selects = []
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
  watch: {
    array (val) {
      console.log('opt', this.opt, val)
      if (this.opt) {
        let tmp = val.filter(item => item.questionId === this.opt.relatedQuesGroupID)
        this.question = this.opt.quesId
        this.$refs.input.value = this.opt.quesSn
        this.$emit('groupChange', this.opt.relatedQuesGroupID)
        this.$refs.host.value = this.opt.quesText.split('____________')[0]
        this.$refs.group.value = tmp[0] ? tmp[0].quesText : '无此QuestionGroup'
      }
    },
    opt (val) {
      console.log('opt1', this.opt, val)
      if (this.array) {
        let tmp = this.array.filter(item => item.questionId === this.opt.relatedQuesGroupID)
        this.question = this.opt.quesId
        this.$refs.input.value = this.opt.quesSn
        this.$emit('groupChange', this.opt.relatedQuesGroupID)
        this.$refs.host.value = this.opt.quesText.split('____________')[0]
        this.$refs.group.value = tmp[0] ? tmp[0].quesText : '无此QuestionGroup'
      }
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
    onAttentionEventHandler  (val) {
      this.attention = val
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
    showFunc () {
      this.tmpSelected = this.$refs.host.value
      this.reWriteName = 'pkuInput'
      this.$nextTick(() => {
        if (this.opt) {
          this.$refs.host.value = this.tmpSelected
        } else {
          // let tmp = this.questions.filter(item => item.questionId === this.question)
          // this.$refs.host.value = '访员是否提问了 ' + tmp[0].quesText + ' 一题相关内容？'
          this.$refs.host.value = this.tmpSelected
        }
      })
    },
    submitFunc () {
      let title = this.$refs.host.value
      this.reWriteName = 'pkuSelect'
      this.$nextTick(() => {
        if (this.opt) {
          this.$refs.host.value = title
        } else {
          this.$refs.host.value = title
          // let tmp = this.questions.filter(item => item.questionId === this.question)
          // this.$refs.host.value = '访员是否提问了 ' + tmp[0].quesText + ' 一题相关内容？'
        }
      })
      this.tmpSelected = ''
    },
    cancelFunc () {
      this.reWriteName = 'pkuSelect'
      this.$nextTick(() => {
        if (this.opt) {
          // this.$refs.host.value = this.opt.quesText.split('____________')[0]
          this.$refs.host.value = this.tmpSelected
        } else {
          // let tmp = this.questions.filter(item => item.questionId === this.question)
          // this.$refs.host.value = '访员是否提问了 ' + tmp[0].quesText + ' 一题相关内容？' || '访员是否提问了下面这一题相关内容？'
          this.$refs.host.value = this.tmpSelected
        }
      })
    },
    onGroupEventHandler (val) {
      if (val) {
        this.group = val
        this.groupID++
        if (this.reWrite) {
          this.reWriteName = 'pkuSelect'
        }
        if (this.opt) {
          this.$refs.edit.show = false
          this.$refs.host.value = this.opt.quesText.split('____________')[0]
        } else {
          this.$nextTick(() => {
            this.$refs.host.reset()
          })
        }
        this.question = ''
        this.questionID = 0
        this.$emit('groupChange', val)
      }
    },
    onQuestionEventHandler (val) {
      if (val) {
        this.question = val
        this.questionID++
        // this.$refs.host.value = '访员是否提问了 ' + this.$refs.host.value + ' 一题相关内容？'
        // console.log(3434, this.$refs.host.value)
      }
    },
    onSubmitEventHandler () {
    //   let content = this.title.filter(item => item.questionId === this.select)
      let value = []
      let sns = []
      let ctype = []
      this.selects.forEach(item => {
        // console.log(item)
        value.push(item.val)
        sns.push(item.sn)
        if (item.checktype === '') {
          item.checktype = true
        }
        ctype.push(item.checktype)
      })
      // console.log(this.inputTitle)
      this.$emit('callback', {
        'QuesOptionValues': sns.toString(),
        'QuesOptionTexts': value.toString(),
        'optionWithoutInput': ctype.toString(),
        // 'questionID': this.select,
        'questionContent': this.$refs.host.value + ' 一题，' + this.inputTitle,
        'questionSn': this.inputSn,
        'QuesType': '0001',
        'attention': this.attention
        //temp1.$children[3].$data.value =2
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
