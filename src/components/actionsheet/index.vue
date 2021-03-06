<template>
  <div class="vux-actionsheet">
    <transition name="vux-actionsheet-mask">
      <div class="weui-mask weui-mask_transparent" @click="onClickingMask" v-show="show"></div>
    </transition>
    <div class="weui-skin_android" v-if="theme === 'android'">
      <transition name="vux-android-actionsheet">
        <div class="weui-actionsheet" v-show="show">
          <div class="weui-actionsheet__menu">
            <div class="weui-actionsheet__cell" v-for="(text, key) in menus" @click="
              onMenuClick(text, key)" v-html="$t(text.label || text)">
            </div>
          </div>
        </div>
      </transition>
    </div>
    <div class="weui-actionsheet" :class="{'weui-actionsheet_toggle': show}" v-else>
      <div class="weui-actionsheet__menu">
        <div class="weui-actionsheet__cell" v-if="hasHeaderSlot">
          <slot name="header"></slot>
        </div>
        <div class="weui-actionsheet__cell" v-for="(text, key) in menus" @click="onMenuClick(text, key)" v-html="$t(text.label || text)" :class="`vux-actionsheet-menu-${text.type || 'default'}`">
        </div>
      </div>
      <div class="weui-actionsheet__action" @click="emitEvent('on-click-menu', 'cancel')" v-if="showCancel">
        <div class="weui-actionsheet__cell">{{cancelText || $t('cancel')}}</div>
      </div>
    </div>
  </div>
</template>

<i18n>
cancel:
  en: cancel
  zh-CN: 取消
</i18n>

<script>
export default {
  mounted () {
    this.hasHeaderSlot = !!this.$slots.header
    this.$nextTick(() => {
      this.$tabbar = document.querySelector('.weui-tabbar')
    })
  },
  props: {
    value: Boolean,
    showCancel: Boolean,
    cancelText: String,
    theme: {
      type: String,
      default: 'ios'
    },
    menus: {
      type: [Object, Array],
      default: () => ({})
    },
    closeOnClickingMask: {
      type: Boolean,
      default: true
    },
    closeOnClickingMenu: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      hasHeaderSlot: false,
      show: false
    }
  },
  methods: {
    onMenuClick (text, key) {
      if (typeof text === 'string') {
        this.emitEvent('on-click-menu', key)
      } else {
        if (text.type !== 'disabled' && text.type !== 'info') {
          if (text.value) {
            this.emitEvent('on-click-menu', text.value)
          } else {
            this.show = false
          }
        }
      }
    },
    onClickingMask () {
      this.$emit('on-click-mask')
      this.closeOnClickingMask && (this.show = false)
    },
    emitEvent (event, menu) {
      if (event === 'on-click-menu' && !/.noop/.test(menu)) {
        this.$emit(event, menu)
        this.$emit(`${event}-${menu}`)
        this.closeOnClickingMenu && (this.show = false)
      }
    },
    fixIos (zIndex) {
      if (this.$el.parentNode && this.$el.parentNode.className.indexOf('v-transfer-dom') !== -1) {
        return
      }
      if (this.$tabbar && /iphone/i.test(navigator.userAgent)) {
        this.$tabbar.style.zIndex = zIndex
      }
    }
  },
  watch: {
    show (val) {
      this.$emit('input', val)
      if (val) {
        this.fixIos(-1)
      } else {
        setTimeout(() => {
          this.fixIos(100)
        }, 200)
      }
    },
    value: {
      handler: function (val) {
        this.show = val
      },
      immediate: true
    }
  },
  beforeDestroy () {
    this.fixIos(100)
  }
}
</script>

<style lang="less">
@import '../../styles/weui/widget/weui_tips/weui_mask';
@import '../../styles/weui/widget/weui_tips/weui_actionsheet';

.vux-actionsheet-menu-primary {
  color: @actionsheet-label-primary-color;
}
.vux-actionsheet-menu-warn {
  color: @actionsheet-label-warn-color;
}
.vux-actionsheet-menu-default {
  color: @actionsheet-label-default-color;
}
.vux-actionsheet-menu-disabled {
  color: @actionsheet-label-disabled-color;
}
.vux-actionsheet-mask-enter,
.vux-actionsheet-mask-leave-active,
.vux-android-actionsheet-enter,
.vux-android-actionsheet-leave-active {
  opacity: 0;
}
.vux-actionsheet-mask-leave-active,
.vux-actionsheet-mask-enter-active,
.vux-android-actionsheet-leave-active,
.vux-android-actionsheet-enter-active {
  transition: opacity 300ms!important;
}
</style>
