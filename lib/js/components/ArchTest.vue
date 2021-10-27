<template>
  <v-tooltip v-model="visible" top>
    <template v-slot:activator>
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
            :value="label + 'corosii'"
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
