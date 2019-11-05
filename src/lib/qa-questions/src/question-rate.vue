<template>
  <div class="don-qa-question-rate" v-if="!fill">
    <pku-tab
      :list="list">
      <div slot="slot_0">
        <don-qa-question-wrap label="题目序号" ref="t1">
          <pku-input
            class="wrap-input"
            ref="input"
            @change="onSnEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="题目标题" ref="t2">
          <pku-input
            class="wrap-input"
            ref="input"
            @change="onTitleEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="净值个数">
          <pku-input
            class="wrap-input"
            ref="input1"
            :message="inputMax"
            @change="onMaxEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="净值初始值">
          <pku-input
            class="wrap-input"
            ref="input2"
            :message="inputVoidNum"
            @change="onNumEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap>
          <pku-rate
            :max="Number(inputMax)"
            :voidShow="true"
            :voidStart="Number(inputVoidNum)"
          ></pku-rate>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="核查员注意">
          <pku-input
            class="wrap-input"
            ref="attention"
            @change="onAttentionEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap>
          <pku-button
            value="保存"
            :class="{'btn-primary': true, 'btn-disabled': (!inputSn || !inputTitle || inputMax === '' || inputVoid === '')}"
            :disabled="(!inputSn || !inputTitle || inputMax === '' || inputVoid === '')"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      </div>
      <div slot="slot_1">
        <don-qa-question-wrap label="题目类型" v-if="!fill">
            <pku-radio
              importKey="name"
              exportKey="key"
              class="wrap-input"
              ref="type"
              :message="options"
              @callback="onTypeEventHandler"></pku-radio>
        </don-qa-question-wrap>
        <don-qa-question-wrap>
          <pku-button
            value="保存"
            :class="{'btn-primary': true, 'btn-disabled': (!inputSn || !inputTitle || inputMax === '' || inputVoid === '')}"
            :disabled="(!inputSn || !inputTitle || inputMax === '' || inputVoid === '')"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      </div>
    </pku-tab>
  </div>
  <div class="don-qa-question-rate" v-else>
    <pku-rate
      :max="Number(inputMax)"
      :voidShow="true"
      :voidStart="Number(inputVoidNum)"
    ></pku-rate>
  </div>
</template>

<script>
export default {
  name: 'donQuestionRate',
  props: {
    max: {
      type: Number,
      default: 5
    },
    voidNum: {
      type: Number,
      default: 1
    },
    res: {
      type: Object,
      default () {
        return null
      }
    },
    opt: {
      type: Object,
      default () {
        return null
      }
    },
    fill: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      list: ['问题设置', '个性设置'],
      options: [
        { name: '星形题', key: '0600' },
        { name: '数值题', key: '0601' }
      ],
      inputSn: undefined,
      inputTitle: undefined,
      inputMax: this.max,
      inputVoidNum: this.voidNum,
      type: '0600',
      attention: ''
    }
  },
  mounted () {
    if (this.fill) {
      if (this.res) {
        this.$children[0].$data.start = Number(this.res.recomStartNum)
        this.$children[0].$data.max = Number(this.res.valueCount)
        this.$children[0].$data.clickID = this.res.quesOptions[0] - this.res.recomStartNum
        this.$children[0].$data.nowValue = this.res.quesOptions[0] - this.res.recomStartNum
      }
    } else if (!this.fill && this.opt) {
      this.$refs.t1.$children[0].$data.value = this.opt.quesSn
      this.$refs.t2.$children[0].$data.value = this.opt.quesText.split('____________')[0]
      this.$refs.input1.value = this.opt.valueCount
      this.$refs.input2.value = this.opt.recomStartNum
      this.$refs.type.$data.value = this.opt.quesType
      this.$refs.attention.value = this.opt.attention
    } else {
      this.$refs.type.$data.value = '0600'
    }
  },
  methods: {
    onSnEventHandler (val) {
      this.inputSn = val
      console.log('Sn', this.inputSn, (!this.inputSn || !this.inputTitle || this.inputMax === '' || this.inputVoid === ''))
    },
    onAttentionEventHandler  (val) {
      this.attention = val
    },
    onTitleEventHandler (val) {
      this.inputTitle = val
      console.log('Title', this.inputTitle, (!this.inputSn || !this.inputTitle || this.inputMax === '' || this.inputVoid === ''))
    },
    onMaxEventHandler (val) {
      this.inputMax = val
    },
    onTypeEventHandler (val) {
      console.log('rate-typehandler', val)
      this.type = val
    },
    onNumEventHandler (val) {
      this.inputVoidNum = Number(val)
    },
    onSubmitEventHandler () {
      this.$emit('callback', {
        'type': this.opt ? 'put' : 'post',
        'questionContent': this.inputTitle,
        'valueCount': this.inputMax,
        'recomStartNum': this.inputVoidNum,
        'questionSn': this.inputSn,
        'QuesType': this.type,
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
div.don-qa-question-rate >>> .don-qa-question-wrap label {
  font-size: 14px;
}
</style>
