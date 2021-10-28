<template>
  <v-tooltip v-model="visible" top>
    <template v-slot:activator="on">
      <div
        class="arch-tag__container"
        :class="tagClass"
        @click="copyValueText"
      >
        <div class="arch-tag__control">
          <span class="arch-tag__icon">
            <v-icon>mdi-checkbox-multiple-blank-outline</v-icon>
          </span>
          <input
            ref="archTagInput"
            type="text"
            :value="label + ' arch-tag component from external directory'"
            readonly
            class="arch-tag__control-field"
          />
        </div>
      </div>
    </template>
    <span>Copied to clipboard! <v-icon dark>mdi-check-all</v-icon></span>
  </v-tooltip>
</template>

<script lang="ts">
import { Vue, Component, Prop, Ref } from 'vue-property-decorator';

@Component
export default class ArchTag extends Vue {
  @Prop({ required: true, type: [String, Number], default: '' }) label: string | number;
  @Ref('archTagInput') textField: HTMLInputElement;
  @Prop({ type: [String], default: '' }) type: string | null;

  public visible: boolean = false;

  public get tagClass(): string {
    switch (this.type) {
      case 'default':
        return '';
      case 'primary':
        return 'code--primary';
      case 'secondary':
        return 'code--secondary';
      default:
        return '';
    }
  }

  private copyValueText(): void {
    this.textField.select();
    document.execCommand('copy');
    this.visible = true;
    setTimeout(() => {
      this.visible = false;
    }, 3000);
  }
}
</script>

<style lang="scss" scoped>
  .arch-tag__container {
    font-family: "Courier New" !important;
    background: #f1ebeb;
    border-radius: 4px;
    font-weight: bolder;
    border: dashed 1px #b1acac;
    text-align: center;
    margin: 0 !important;
    overflow-x: hidden;

    &:hover, .arch-tag__control-field:hover {
      cursor: pointer;
    }

    &:hover .arch-tag__control .arch-tag__icon {
      opacity: 1;
    }

    &:hover {
      cursor: pointer;
    }

    .arch-tag__control-field {
      padding: 4px;
      width: 100%;
      text-align: center;

      &:focus,
      &:active {
        outline: none;
      }
    }

    .arch-tag__control {
      position: relative;

      .arch-tag__icon {
        opacity: 0;
        transition: opacity 0.1s linear;
      }

      .arch-tag__icon > * {
        position: absolute;
        right: 10px;
        top: 50%;
        transform: translateY(-50%);
        font-size: 16px;
      }
    }

    &.code--primary {
      background-color: #dcf5ef;
      border-color: #67af9e;

      & > * {
        color: #67af9e;
      }
    }

    &.code--secondary {
      background-color: rgba(239, 239, 239, 1);
      border-color: rgba(181, 181, 181, 1);

      & > * {
        color: rgba(181, 181, 181, 1);
      }
    }
  }
</style>
