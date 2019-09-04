<template>
  <transition
    name="dialog-fade"
    @after-enter="afterEnter"
    @after-leave="afterLeave">
    <div
      v-show="visible"
      class="el-dialog__wrapper"
      @click.self="handleWrapperClick">
      <div
        role="dialog"
        :key="key"
        aria-modal="true"
        :aria-label="title || 'dialog'"
        :class="['el-dialog', { 'is-fullscreen': fullscreen, 'el-dialog--center': center }, customClass]"
        ref="dialog"
        :style="style">
        <div ref="headerBox">
          <div ref="header" class="el-dialog__header">
            <slot name="title">
              <span class="el-dialog__title">{{ title }}</span>
            </slot>
            <button
              type="button"
              class="el-dialog__headerbtn"
              aria-label="Close"
              v-if="showClose"
              @click="handleClose">
              <i class="el-dialog__close el-icon el-icon-close"></i>
            </button>
          </div>
        </div>
        <div ref="body" class="el-dialog__body" v-if="!lazy||rendered"><slot></slot></div>
        <div ref="footerBox">
          <div v-show="showTips" ref="tips" class="el-dialog__footertips">
            提示：滚动鼠标可显示更多内容...
          </div>
          <div ref="footer" class="el-dialog__footer" v-if="$slots.footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import Popup from 'element-ui/src/utils/popup';
  import Migrating from 'element-ui/src/mixins/migrating';
  import emitter from 'element-ui/src/mixins/emitter';

  export default {
    name: 'ElDialog',

    mixins: [Popup, emitter, Migrating],

    props: {
      title: {
        type: String,
        default: ''
      },

      modal: {
        type: Boolean,
        default: true
      },

      modalAppendToBody: {
        type: Boolean,
        default: true
      },

      appendToBody: {
        type: Boolean,
        default: false
      },

      lockScroll: {
        type: Boolean,
        default: true
      },

      closeOnClickModal: {
        type: Boolean,
        default: true
      },

      closeOnPressEscape: {
        type: Boolean,
        default: true
      },

      showClose: {
        type: Boolean,
        default: true
      },

      width: String,

      fullscreen: Boolean,

      customClass: {
        type: String,
        default: ''
      },

      top: {
        type: String,
        default: '15vh'
      },
      beforeClose: Function,
      center: {
        type: Boolean,
        default: false
      },

      destroyOnClose: Boolean,

      lazy: {
        type: Boolean,
        default: true
      }
    },

    data() {
      return {
        closed: false,
        key: 0,
        showTips: false,
        mutationObserver: null
      };
    },

    watch: {
      visible(val) {
        if (val) {
          this.closed = false;
          this.$emit('open');
          this.$el.addEventListener('scroll', this.updatePopper);
          this.$el.addEventListener('scroll', this.handleScroll);
          this.$nextTick(() => {
            this.$el.scrollTop = 0;
            this.$refs.dialog.scrollTop = 0;
            this.handleScroll();
            if (window.MutationObserver && !this.mutationObserver) {
              this.mutationObserver = new MutationObserver(() => {
                // 监听body区域所有子元素及其后代属性、节点等变化，此时要处理一下footer的固定
                this.handleScroll();
              });
            }
            if (this.mutationObserver) {
              this.mutationObserver.observe(this.$refs.body, {attributes: true, childList: true, characterData: true, subtree: true});
            }
          });
          if (this.appendToBody) {
            document.body.appendChild(this.$el);
          }
        } else {
          this.$el.removeEventListener('scroll', this.updatePopper);
          this.$el.removeEventListener('scoll', this.handleScroll);
          if (!this.closed) this.$emit('close');
          if (this.destroyOnClose) {
            this.$nextTick(() => {
              this.key++;
            });
          }
          if (this.mutationObserver) {
            this.mutationObserver.disconnect();
          }
        }
      }
    },

    computed: {
      style() {
        let style = {};
        if (!this.fullscreen) {
          style.marginTop = this.top;
          if (this.width) {
            style.width = this.width;
          }
        }
        return style;
      }
    },

    methods: {
      getMigratingConfig() {
        return {
          props: {
            'size': 'size is removed.'
          }
        };
      },
      handleWrapperClick() {
        if (!this.closeOnClickModal) return;
        this.handleClose();
      },
      handleClose() {
        if (typeof this.beforeClose === 'function') {
          this.beforeClose(this.hide);
        } else {
          this.hide();
        }
      },
      hide(cancel) {
        if (cancel !== false) {
          this.$emit('update:visible', false);
          this.$emit('close');
          this.closed = true;
        }
      },
      updatePopper() {
        this.broadcast('ElSelectDropdown', 'updatePopper');
        this.broadcast('ElDropdownMenu', 'updatePopper');
      },
      afterEnter() {
        this.$emit('opened');
      },
      afterLeave() {
        this.$emit('closed');
      },
      handleScroll() {
        const headerBox = this.$refs.headerBox;
        const { top: headerTop } = headerBox.getBoundingClientRect();
        const header = this.$refs.header;
        headerBox.style.height = `${headerBox.offsetHeight}px`;
        header.style.width = `${headerBox.offsetWidth}px`;
        header.style.zIndex = this.$el.style.zIndex;
        if (headerTop <= 0) {
          header.style.position = 'fixed';
          header.style.top = 0;
          header.style.boxShadow = '0 0 10px rgba(0, 0, 0, 0.12)';
        } else {
          header.style.position = 'initial';
          header.style.boxShadow = '0 0 0 #ccc';
        }
        const footerBox = this.$refs.footerBox;
        footerBox.style.height = `${footerBox.offsetHeight}px`;
        const { bottom: footerBottom } = footerBox.getBoundingClientRect();
        const footer = this.$refs.footer;
        const tips = this.$refs.tips;
        if (footer) {
          footer.style.width = `${footerBox.offsetWidth}px`;
          footer.style.zIndex = this.$el.style.zIndex;
        }
        tips.style.zIndex = this.$el.style.zIndex;
        if (footerBottom >= innerHeight) {
          this.showTips = !this.fullscreen;
          if (footer) {
            footer.style.position = 'fixed';
            footer.style.bottom = 0;
            footer.style.boxShadow = '0 0 10px rgba(0, 0, 0, 0.12)';
          }
          tips.style.bottom = footer ? footer.offsetHeight + 'px' : 0;
          tips.style.width = `${footerBox.offsetWidth}px`;
        } else {
          this.showTips = false;
          if (footer) {
            footer.style.position = 'initial';
            footer.style.boxShadow = '0 0 0 #ccc';
          }
        }
      }
    },

    mounted() {
      if (this.visible) {
        this.rendered = true;
        this.open();
        if (this.appendToBody) {
          document.body.appendChild(this.$el);
        }
      }
    },

    destroyed() {
      // if appendToBody is true, remove DOM node after destroy
      if (this.appendToBody && this.$el && this.$el.parentNode) {
        this.$el.parentNode.removeChild(this.$el);
      }
    }
  };
</script>
