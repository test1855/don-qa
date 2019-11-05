<template>
  <div class="don-qa-question-input" v-if="!fill">
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
            style="width: 50%;"
            :message="message"
            @change="onTitleEventHandler"></pku-input>
          <pku-select
            class="wrap-select"
            importKey="propertyName"
            exportKey="propertyCode"
            label="增加变量"
            style="width: 49%;"
            :list="propertyInfo"
            @callback="onPropertyEventHandler">
          </pku-select>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="核查员注意">
          <pku-input
            class="wrap-input"
            ref="attention"
            @change="onAttentionEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap v-if="!fill">
          <pku-button
            value="保存"
            class="btn-primary"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      </div>
      <div slot="slot_1">
        <don-qa-question-wrap label="题目类型" v-if="!fill">
          <pku-radio
            importKey="name"
            exportKey="key"
            :message="options"
            class="wrap-input"
            ref="type"
            @callback="onRadioEventHandler"></pku-radio>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="最小长度(值)" v-if="!fill">
          <pku-input
            type="number"
            class="wrap-input"
            ref="input1"
            @change="onMinEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap label="最大长度(值)" v-if="!fill">
          <pku-input
            type="number"
            class="wrap-input"
            ref="input2"
            @change="onMaxEventHandler"></pku-input>
        </don-qa-question-wrap>
        <don-qa-question-wrap v-if="!fill">
          <pku-button
            value="保存"
            :class="{'btn-primary': true, 'btn-disabled': (!inputSn || !inputTitle)}"
            :disabled="(!inputSn || !inputTitle)"
            @callback="onSubmitEventHandler"></pku-button>
        </don-qa-question-wrap>
      </div>
    </pku-tab>
  </div>
  <div class="don-qa-question-input" v-else>
    <pku-input class="wrap-input" v-if="importType === '0100' || importType === '0102'" @change="onCheckMinMaxHandler" ref="inputstr" :disabled="disflag"></pku-input>
    <pku-input class="wrap-number-input" v-else type="number" min="minval" max="maxval" @change="onCheckMinMaxHandler"  ref="inputnum" :disabled="disflag"></pku-input>
  </div>
</template>

<script>
export default {
  name: 'donQuestionInput',
  props: {
    fill: {
      type: Boolean,
      default: false
    },
    opt: {
      type: Object,
      default () {
        return null
      }
    },
    importType: {
      type: String,
      default: '0100'
    },
    res: {
      type: Object,
      default () {
        return null
      }
    },
    disflag: {
      type: Boolean,
      default: false
    },
    minval: {
      type: Number,
      default: 0
    },
    maxval: {
      type: Number,
      default: 9999
    },
    propertyInfo: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    return {
      list: ['问题设置', '个性设置'],
      options: [
        { name: '文本', key: '0100' },
        // { name: '密码', key: '0101' },
        { name: '纯文字', key: '0102' },
        { name: '数值', key: '0103' },
      ],
      inputSn: undefined,
      inputTitle: undefined,
      inputMin: '0',
      inputMax: '9999',
      message: '',
      // type: this.opt.quesType || '0100',
      type: '0100',
      attention: ''
    }
  },
  mounted () {
    if (this.fill && this.res) {
      this.$children[0].$data.value = this.res.quesOptions[0]
    } else if (!this.fill) {
      if (this.opt === null || this.opt === undefined) {
        this.$refs.input1.value = '0'
        this.$refs.input2.value = '9999'
      } else {
        this.$refs.type.$data.value = this.opt.quesType
        this.$refs.t1.$children[0].$data.value = this.opt.quesSn
        this.$refs.t2.$children[0].$data.value = this.opt.quesText.split('____________')[0]
        this.$refs.attention.value = this.opt.attention
        if (this.opt.minCharacter === '' || this.opt.minCharacter === undefined) {
          this.$refs.input1.value = '0'
        } else {
          this.$refs.input1.value = this.opt.minCharacter
        }
        if (this.opt.maxCharacter === '' || this.opt.maxCharacter === undefined) {
          this.$refs.input2.value = '9999'
        } else {
          this.$refs.input2.value = this.opt.maxCharacter
        }
      }
    }
  },
  methods: {
    onRadioEventHandler (val) {
      console.log('input-typehandler', val)
      this.type = val
    },
    // 这个题目类型是用questType来区别的。文本题：0100、密码题：0101、纯文字：0102。
    // 你截图部分的是文本化的显示。原系统是用onclick事件做的。
    onSnEventHandler (val) {
      this.inputSn = val
    },
    onAttentionEventHandler  (val) {
      this.attention = val
    },
    onTitleEventHandler (val) {
      this.inputTitle = val
    },
    onMaxEventHandler (val) {
      this.inputMax = val
    },
    onMinEventHandler (val) {
      this.inputMin = val
    },
    onPropertyEventHandler (val) {
      this.message = this.inputTitle + '${Sample_' + val +'}'
    },
    onCheckMinMaxHandler (val) {
      if((this.importType === '0100' || this.importType === '0102') && val !== undefined && val !== '') {
        if (val.length > this.maxval || val.length < this.minval) {
          this.$refs.inputstr.value = ''
          alert('输入长度在 ' + this.minval + '~' + this.maxval + ' 之间')
        }
      } else if(this.importType === '0103' && val !== undefined && val !== '') {
        if (val > this.maxval || val < this.minval) {
          this.$refs.inputnum.value = ''
          alert('输入范围在 ' + this.minval + '~' + this.maxval + ' 之间')
        }
      }
    },
    onSubmitEventHandler () {
      if (!this.inputTitle || !this.inputSn) {
        this.$notice({
          title: '保存失败！',
          message: '题目序号或标题未填写',
          type: 'error',
          time: 3000
        })
      } else if (this.type !== '0103' && (Number(this.inputMin) >= Number(this.inputMax) || Number(this.inputMin) < 0 || Number(this.inputMax) < 0)) {
        this.$notice({
          title: '保存失败！',
          message: '非数值题的最小长度应小于最大长度且长度不能为负数',
          type: 'error',
          time: 3000
        })
      } else if (Number(this.inputMin) >= Number(this.inputMax)) {
        this.$notice({
          title: '保存失败！',
          message: '数值题的最小值应小于最大值',
          type: 'error',
          time: 3000
        })
      } else {
        this.$emit('callback', {
          'type': this.opt ? 'put' : 'post',
          'questionContent': this.inputTitle,
          'questionSn': this.inputSn,
          'QuesType': this.type,
          'minCharacter': Number(this.inputMin),
          'maxCharacter': Number(this.inputMax),
          'attention': this.attention
        })
      }
    }
  }
}
</script>
<style scoped>
.wrap-input, .wrap-select {
  margin-bottom: 20px;
}
.wrap-number-input {
  min-width: 100px;
}
div.don-qa-question-input >>> .don-qa-question-wrap label {
  font-size: 14px;
}
</style>
