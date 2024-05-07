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
      inputStep: ''
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

      console.log('initData', initData.data.text)

      //测试加一个小按钮
      // initData.data.text = initData.data.text.replace(/\<u\>[\s\S]+\<\/u\>/g,`<div style="width:50px;border:1px solid #000;"></div>`)
      // initData.data.text = initData.data.text.replace(/\<u\>[\s\S]+?\<\/u\>/g,`<div style="width:50px;border:1px solid #000;"></div>`)
      let u_arr = initData.data.text.match(/\<u\>[\s\S]+?\<\/u\>/g)
      console.log('u_arr', u_arr)
      u_arr.forEach((u_item,u_index) => {
        initData.data.text = initData.data.text.replace(
          u_item,
          `${u_item.substring(0, 2)} id="u_${u_index}" style="color:rgba(0,0,0,0);border-bottom: 2px solid #ccc;" ${u_item.substring(2)}`
        )
      })

      this.$bus.$emit('setData', initData)

      setTimeout(()=>{

      let u_1 = document.getElementById("u_1")
        console.log("u_1", u_1)
      },500)

      console.log('resData', initData.data.text)
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
</style>
