<template>
  <div class="don-qa-question-cs-g" v-if="!fill">
    <don-qa-question-wrap label="题目序号">
      <pku-input
        class="wrap-input"
        ref="input"
        @change="onInputEventHandler"></pku-input>
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
        :is="reWriteName"
        selected="访员是否至少追问一次？"
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
    <don-qa-question-wrap label="核查员注意">
      <pku-input
        class="wrap-input"
        ref="attention"
        @change="onAttentionEventHandler"></pku-input>
    </don-qa-question-wrap>
    <don-qa-question-wrap>
      <pku-radio
        importKey="name"
        exportKey="id"
        :disabled="true"
        :message="options"></pku-radio>
    </don-qa-question-wrap>
    <don-qa-question-wrap>
      <pku-button
        value="保存"
        :class="{'btn-primary': true, 'btn-disabled': (questionID * groupID * inputSn.length === 0) && question.length === 0}"
        :disabled="(questionID * groupID * inputSn.length === 0) && question.length === 0"
        @callback="onSubmitEventHandler"></pku-button>
    </don-qa-question-wrap>
  </div>
  <div class="don-qa-question-cs-g" v-else>
     <pku-radio
        importKey="name"
        exportKey="id"
        :disabled="false"
        :message="options"></pku-radio>
  </div>
</template>

<script>
export default {
  name: 'donQuestionCsG',
  props: {
    reWrite: {
      type: Boolean,
      default: false
    },
    fill: {
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
    },
    res: {
      type: Object,
      default () {
        return null
      }
    },
    options: {
      type: Array,
      default () {
        return [
          { name: '是', key: 1 },
          { name: '否', key: 5 }
        ]
      }
    },
    opt: {
      type: Object,
      default () {
        return null
      }
    }
  },
  data () {
    return {
      reWriteName: 'pkuSelect',
      groupID: 0,
      questionID: 0,
      group: '',
      question: '',
      inputSn: '',
      cid: '',
      rid: '',
      tmpSelected: '',
      attention: '',
    }
  },
  watch: {
    array (val) {
      if (this.opt) {
        let tmp = val.filter(item => item.questionId === this.opt.relatedQuesGroupID)
        this.question = this.opt.quesId
        this.$refs.input.value = this.opt.quesSn
        this.$refs.attention.value = this.opt.attention
        this.$emit('groupChange', this.opt.relatedQuesGroupID)
        this.$refs.host.value = this.opt.quesText.split('____________')[0]
        this.$refs.group.value = tmp[0] ? tmp[0].quesText : '无此QuestionGroup'
      }
    },
    opt (val) {
      if (this.array) {
        let tmp = this.array.filter(item => item.questionId === this.opt.relatedQuesGroupID)
        this.question = this.opt.quesId
        this.$refs.input.value = this.opt.quesSn
        this.$refs.attention.value = this.opt.attention
        this.$emit('groupChange', this.opt.relatedQuesGroupID)
        this.$refs.host.value = this.opt.quesText.split('____________')[0]
        this.cid = this.opt.quesText.split('____________')[1].split('[')[0]
        this.rid = this.opt.quesText.split('[')[1].split(']')[0]
        console.log(this.rid)
        this.$refs.group.value = tmp[0] ? tmp[0].quesText : '无此QuestionGroup'
      }
    }
  },
  mounted () {
    if (this.fill && this.res) {
      this.res.quesOptions.forEach((item, index) => {
        if (item !== null) {
          this.$children[0].$data.value = index
        }
      })
    } else if (!this.fill && this.opt) {
      let tmp = this.array.filter(item => item.questionId === this.opt.relatedQuesGroupID)
      this.question = this.opt.quesId
      this.$refs.input.value = this.opt.quesSn
      this.$refs.attention.value = this.opt.attention
      this.$emit('groupChange', this.opt.relatedQuesGroupID)
      this.$refs.host.value = this.opt.quesText.split('____________')[0]
      this.$refs.group.value = tmp[0] ? tmp[0].quesText : '无此QuestionGroup'
    }
  },
  methods: {
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
    onInputEventHandler (val) {
      this.inputSn = val
    },
    onAttentionEventHandler  (val) {
      this.attention = val
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
        this.$refs.host.value = this.$refs.host.value + '一题,访员是否至少追问一次？'
      }
    },
    onSubmitEventHandler () {
      let content = this.questions.filter(item => item.questionId === this.question)
      this.$emit('callback', {
        'type': this.opt ? 'put' : 'post',
        'QuesOptionValues': '1,5',
        'QuesOptionTexts': '是,否',
        'questionID': this.question,
        'questionContent': this.$refs.host.value,
        'questionSn': this.inputSn,
        'cid': this.cid,
        'rid': this.rid,
        'QuesType': '3006',
        'attention': this.attention
      })
    }
  }
}
</script>
<style scoped>
.wrap-input, .wrap-select {
  margin-bottom: 20px;
}
div.don-qa-question-cs-g >>> .don-qa-question-wrap label {
  font-size: 14px;
}
</style>