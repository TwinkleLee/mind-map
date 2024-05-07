<template>
  <div class="demonstrateContainer" :class="{ isDark: isDark }">
    <el-tooltip
      class="item"
      effect="dark"
      :content="$t('demonstrate.demonstrate')"
      placement="top"
    >
      <div class="btn iconfont iconyanshibofang" @click="enterDemoMode"></div>
    </el-tooltip>
    <div
      class="exitDemonstrateBtn"
      @click="exit"
      ref="exitDemonstrateBtnRef"
      v-if="isEnterDemonstrate"
    >
      <span class="icon iconfont iconguanbi"></span>
    </div>
    <div class="stepBox" ref="stepBoxRef" v-if="isEnterDemonstrate">
      <div class="jump" @click="prev" :class="{ disabled: curStepIndex <= 0 }">
        <span class="icon el-icon-back"></span>
      </div>
      <div class="step">{{ curStepIndex + 1 }} / {{ totalStep }}</div>
      <div
        class="jump"
        @click="next"
        :class="{ disabled: curStepIndex >= totalStep - 1 }"
      >
        <span class="icon el-icon-right"></span>
      </div>
      <div class="input">
        <input
          type="text"
          v-model="inputStep"
          @keyup.enter.stop="onEnter"
          @keydown.stop
        />
      </div>
    </div>

    <div class="u_button_list" ref="u_button_list" v-if="isEnterDemonstrate">
      <button @click="uShowAll">全部显示</button>
      <button @click="uHideAll">全部隐藏</button>
      <button @click="uHideCurrent" :disabled="u_custom_index==-1">上一个</button>
      <button @click="uShowNext" :disabled="u_custom_index==u_custom_length-1">下一个</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    mindMap: {
      type: Object
    },
    isDark: {
      type: Boolean
    }
  },
  data() {
    return {
      isEnterDemonstrate: false,
      curStepIndex: 0,
      totalStep: 0,
      inputStep: '',

      u_custom_length: 0,
      u_custom_index: -1
    }
  },
  created() {
    this.$bus.$on('demonstrate_jump', this.onJump)
    this.$bus.$on('exit_demonstrate', this.onExit)
  },
  methods: {
    enterDemoMode() {
      this.isEnterDemonstrate = true
      this.$nextTick(() => {
        const el = document.querySelector('#mindMapContainer')
        el.appendChild(this.$refs.exitDemonstrateBtnRef)
        // el.appendChild(this.$refs.stepBoxRef) //FIXME 不再显示页码

        el.appendChild(this.$refs.u_button_list)
      })
      this.mindMap.demonstrate.enter()
    },

    exit() {
      this.mindMap.demonstrate.exit()
    },

    onExit() {
      this.isEnterDemonstrate = false
      this.curStepIndex = 0
      this.totalStep = 0
    },

    onJump(index, total) {
      this.curStepIndex = index
      this.totalStep = total

      // highlightEl 蒙层出现,对其添加点击穿透
      this.mindMap.demonstrate.highlightEl.style.cssText +=
        'pointer-events:none;'

      let initData = JSON.parse(JSON.stringify(this.mindMap.getData(), null, 2))

      console.log('initData', initData)
      // console.log('initData', initData.data.text)

      let flat_initData = this.u2Flat([initData])
      console.log('flat_initData', flat_initData)

      let u_index = 0
      flat_initData.forEach(item => {
        // console.log(item.data.text)
        let temp_u_arr = item.data.text.match(/\<u[\s\S]+?\<\/u\>/g)
        // console.log(temp_u_arr)
        if (temp_u_arr) {
          temp_u_arr.forEach(u_item => {
            let text_content = u_item.match(/^\<u[\s\S]+>([\s\S]+)\<\/u\>$/)
            console.log("text_content",text_content)

            item.data.text = item.data.text.replace(
              u_item,
              `${u_item.substring(
                0,
                2
              )} data-custom-u-${u_index}  style="color:rgba(0,0,0,0);border-bottom: 2px solid #ccc;" ${u_item.substring(
                2
              )}`
            )
            u_index++
          })
        }
      })

      this.u_custom_length = u_index
      this.u_custom_index = -1

      this.$bus.$emit('setData', initData)

      // console.log('resData', initData.data.text)
    },

    prev() {
      this.mindMap.demonstrate.prev()
    },

    next() {
      this.mindMap.demonstrate.next()
    },

    onEnter() {
      const num = Number(this.inputStep)
      if (Number.isNaN(num)) {
        this.inputStep = ''
      } else if (num >= 1 && num <= this.totalStep) {
        this.mindMap.demonstrate.jump(num - 1)
      }
    },

    u2Flat(list) {
      let arr = []
      list.forEach(item => {
        arr.push(item)
        if (item.children && item.children.length > 0) {
          arr.push(...this.u2Flat(item.children))
        }
      })
      return arr
    },

    uShowAll() {
      for (let u_index = 0; u_index < this.u_custom_length; u_index++) {
        let u_el = document.querySelector(`u[data-custom-u-${u_index}]`)
        u_el.style.color = 'inherit'
      }
      this.u_custom_index = this.u_custom_length - 1
    },
    uHideAll() {
      for (let u_index = 0; u_index < this.u_custom_length; u_index++) {
        let u_el = document.querySelector(`u[data-custom-u-${u_index}]`)
        u_el.style.color = 'rgba(0,0,0,0)'
      }
      this.u_custom_index = -1
    },
    uShowNext() {
      this.u_custom_index++
      let u_el = document.querySelector(
        `u[data-custom-u-${this.u_custom_index}]`
      )
      u_el.style.color = 'inherit'
    },
    uHideCurrent() {
      let u_el = document.querySelector(
        `u[data-custom-u-${this.u_custom_index}]`
      )
      u_el.style.color = 'rgba(0,0,0,0)'
      this.u_custom_index--
    }
  }
}
</script>

<style lang="less" scoped>
.demonstrateContainer {
  display: flex;
  align-items: center;

  &.isDark {
    .btn {
      color: hsla(0, 0%, 100%, 0.6);
    }
  }

  .item {
    margin-right: 12px;

    &:last-of-type {
      margin-right: 0;
    }
  }

  .btn {
    cursor: pointer;
    font-size: 24px;
  }
}

.exitDemonstrateBtn {
  position: absolute;
  right: 40px;
  top: 20px;
  cursor: pointer;
  z-index: 10001;

  .icon {
    font-size: 28px;
    // color: #fff;
    color: #000; //FIXME
  }
}

.stepBox {
  position: absolute;
  right: 40px;
  bottom: 20px;

  z-index: 10001;
  display: flex;
  align-items: center;

  .step {
    color: #fff;
    margin: 0 12px;
  }

  .jump {
    color: #fff;
    cursor: pointer;

    &.disabled {
      cursor: not-allowed;
      color: #999;
    }
  }

  .input {
    margin-left: 12px;
    display: flex;
    align-items: center;

    input {
      width: 50px;
      height: 30px;
      text-align: center;
      background-color: transparent;
      border: 1px solid #999;
      outline: none;
      color: #fff;
    }
  }
}

.u_button_list {
  position: fixed;
  right: 20px;
  bottom: 10px;
  z-index: 10000;
  button {
    padding: 4px 10px;
    margin: 10px;
    user-select: none;
  }
}
</style>
