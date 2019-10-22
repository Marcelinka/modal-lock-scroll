<template>
  <transition name="wrapper-fade" @leave="leave" @before-enter="beforeEnter" @enter="afterEnter">
    <div class="modal-wrapper" v-show="modalVisible" @click="onOutsideClicked" ref="modal-wrapper" :id="modalId">
      <transition name="fade">
        <div class="modal-inside" v-show="modalVisible" :style="modalStyleObj">
          <!-- Close button -->
          <a v-if="closable" class="modal-close modal-close_large" role="button" @click.prevent="hideModal"></a>

          <!-- Modal title -->
          <div class="modal-header">
            <div class="modal-title">{{ modalTitle }}</div>
          </div>

          <!-- Modal content -->
          <div class="modal-content">
            <!-- Modal content slot -->
            <slot />
          </div>
        </div>
      </transition>
    </div>
  </transition>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';

/**
 * Modal wrapper component - wraps anything into modal
 */
@Component
export default class ModalComponent extends Vue {
  @Prop() public modalId!: string;
  @Prop() public modalTitle!: string;
  @Prop({ default: true })
  public closable!: boolean;
  @Prop() public modalStyleObj!: any;
  @Prop() public eventBus!: Vue;
  @Prop({default: true})
  public outsideClosable!: boolean;

  public modalVisible: boolean = false;
  public scrollY: number = 0;

  public mounted() {
    if (this.eventBus) {
      this.eventBus.$on('showModal', this.showModalByName);
      this.eventBus.$on('closeModal', this.hideModalByName);
      this.eventBus.$on('closeAllModals', this.hideModal);
    }
  }

  @Watch('modalVisible')
  public watchModalVisible(value: boolean): void {
    const html: HTMLElement | null = document.querySelector('html');

    if (value) {
      document.body.style.top = `${-this.scrollY}px`;
      document.body.style.height = `calc(${this.scrollY}px + 100vh)`;
      document.body.classList.add('scroll-lock');
      html && html.classList.add('scroll-lock');
    } else {
      document.body.style.top = '0';
      document.body.style.height = '100%';
      document.body.classList.remove('scroll-lock');
      html && html.classList.remove('scroll-lock');
    }
  }

  public leave(): void {
    (this.$refs['modal-wrapper'] as HTMLElement).style.overflow = 'hidden';
  }

  public beforeEnter(): void {
    (this.$refs['modal-wrapper'] as HTMLElement).style.overflow = 'auto';
  }

  public afterEnter(): void {
    (this.$refs['modal-wrapper'] as HTMLElement).scrollTop = 0;
  }

  public onOutsideClicked(e: MouseEvent) {
    if (!this.outsideClosable) {
      return;
    }
    if ((e.target && (e.target as Element).closest('.modal-inside')) || !this.closable) {
      return;
    }

    this.hideModal();
  }

  public showModalByName(name: string): void {
    if (this.modalId === name) {
      this.scrollY = window.pageYOffset;
      this.modalVisible = true;
      this.eventBus.$emit(`openModal:${this.modalId}`);
    }
  }

  public hideModalByName(name: string): void {
    if (this.modalId === name) {
      this.hideModal();
    }
  }

  public hideModal(): void {
    this.eventBus.$emit(`closeModal:${this.modalId}`);
    this.modalVisible = false;
    this.$nextTick(() => {
      window.scrollTo(0, this.scrollY);
    });
  }
}
</script>

<style lang="less" >
@import '../less/modules/_vars';

html.scroll-lock,
body.scroll-lock {
  position: fixed;
  width: 100%;
  overflow: hidden;
  -webkit-overflow-scrolling: touch;
}

.modal-wrapper {
  position: fixed;
  overflow-y: auto;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  color: black;
  opacity: 1 !important;
  z-index: 209;
  -webkit-overflow-scrolling: touch;
  text-align: center;

  &::before {
    content: '';
    display: inline-block;
    vertical-align: middle;
    height: 100%;
  }

  .modal-inside {
    box-sizing: border-box;
    position: relative;
    z-index: 210;
    background-color: #fff;
    max-width: 856px;
    width: 100%;
    border-radius: 4px;
    padding: 28px 31px 32px;
    text-align: left;
    display: inline-block;
    vertical-align: middle;
    line-height: initial;

    .modal-title {
      font-size: 24px;
      line-height: 1.7;
      font-weight: bold;
    }

    .modal-close {
      -webkit-transform: translate(0, -50%);
      -moz-transform: translate(0, -50%);
      -o-transform: translate(0, -50%);
      -ms-transform: translate(0, -50%);
      transform: translate(0, -50%);
      z-index: 20;
      position: absolute;
      top: 30px;
      right: 26px;
      margin-right: -4px;
    }

    .modal-close_large {
      display: block;
      width: 80px;
      height: 80px;
      top: 41px;
      right: 4px;

      &:hover {
        &::after {
          opacity: 0.1;
        }
      }

      &::before {
        content: '';
        display: block;
        position: absolute;
        top: 50%;
        left: 50%;
        margin-top: -9px;
        margin-left: -9px;
        width: 18px;
        height: 18px;
        background: url('data:image/svg+xml;charset=utf-8,<svg width="18" height="18" viewBox="0 0 18 18" fill="%231aa34f" xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="none"><path fill="%23000" fill-opacity="0" d="M0 0h18v18H0z"/><path fill-rule="evenodd" clip-rule="evenodd" d="M9 10.414l7.293 7.293 1.414-1.414L10.414 9l7.293-7.293L16.293.293 9 7.586 1.707.293.293 1.707 7.586 9 .293 16.293l1.414 1.414L9 10.414z"/></svg>')
          no-repeat;
        -webkit-background-size: 100% 100%;
        -moz-background-size: 100% 100%;
        background-size: 100% 100%;
      }

      &::after {
        content: '';
        position: absolute;
        -webkit-border-radius: 50%;
        -moz-border-radius: 50%;
        border-radius: 50%;
        width: 40px;
        height: 40px;
        top: 50%;
        left: 50%;
        margin-top: -20px;
        margin-left: -20px;
        background: #1aa34f;
        transition: all 0.2s ease-out;
        opacity: 0;
        -ms-filter: 'progid:DXImageTransform.Microsoft.Alpha(Opacity=0)';
        filter: alpha(opacity=0);
      }
    }

    .modal-content {
      margin-top: 20px;
    }
  }

  &_big{
    .modal-inside { max-width: 856px }
  }
  &_medium {
    .modal-inside { max-width: 578px }
  }
  &_small {
    .modal-inside { max-width: 450px }
  }
}

@media @phone-768 {
  .modal-wrapper {
    .modal-inside {
      border-radius: 0;
      width: 100%;
      min-height: 100%;
      padding: 17px 15px 32px;
      margin-bottom: 0;
      font-size: 13px;
      vertical-align: top;

      .modal-title {
        font-style: normal;
        font-weight: bold;
        font-size: 15px;
        line-height: normal;
      }

      .modal-close_large {
        width: 55px;
        height: 55px;
        top: 28px;
        right: 5px;
      }
    }
  }
}
</style>
