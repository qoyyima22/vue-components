<template>
  <tr
    :key="item[itemKey]"
    @dblclick.stop="() => $emit('dblclick:row', item)"
    :style="`height: ${rowHeight + (isRowExpanded ? 250 : 0)}px; flex-basis: auto; flex-wrap: ${isRowExpanded ? 'wrap' : 'nowrap'};`"
    :class="isSelected(item) ? 'v-data-table__selected' : ''"
    tabindex="0"
    @keydown.tab="onTabRow"
    @keydown.esc.stop="onEscRow"
    :data-index="item.index"
    @click="onClickRow(item)"
    v-intersect="(ent, obs, isShow) => lastLine(ent, obs, isShow, item.index)"
    :id="`row-${item[itemKey]}`"
  >
    <td v-if="$scopedSlots['expanded-item']" :style="{
        width: '60px',
        minWidth: '60px',
        flex: 1,
      }">
      <v-btn x-small icon class="ma-auto" @click.stop="onClickToggleExpand(item.index)">
        <v-icon small>{{isRowExpanded ?  'mdi-chevron-up' : 'mdi-chevron-down'}}</v-icon>
      </v-btn>
    </td>
    <td
      :style="{
        width: header.width,
        minWidth: header.width,
        flex: header.value === 'actions' ? 0 : 1,
        height: isRowExpanded ? `${rowHeight}px` : '100%' 
      }"
      v-for="(header, j) in headers"
      :key="header.value"
      :class="[header.cellClass, header.value]"
      @keydown.up.stop
      @keydown.down.stop
      @keydown.esc.stop="
        () => onEscapeCustomColumn(item, `item.${header.value}`)
      "
    >
      <div
        v-if="header.value === 'base_no'"
        style="display: flex; align-items: center"
      >
        {{ getItemNo(item) }}
      </div>
      <div
        v-else-if="Object.keys($scopedSlots).includes(`item.${header.value}`)"
        :key="`${i}-item.${header.value}-${item[itemKey]}`"
        @keydown.f9="() => copyPrevRow({ header, index: item.index })"
        @keydown.tab.stop="(e) => onTabScopedSlot(e, `item.${header.value}`)"
        style="height: 100%; display: flex; align-items: center"
      >
        <slot
          :name="`item.${header.value}`"
          :item="item"
          :loading="false"
          :rowSelected="activeRowComputed === item.index"
          :ready="() => tez(item.index, i, j, header.value)"
        />
      </div>
      <div
        v-else-if="
          header.value === 'actions' &&
          actions &&
          (actions.delete || actions.edit)
        "
        style="height: 100%"
      >
        <v-btn
          v-if="actions.edit"
          class="actions_edit"
          @keydown.esc.stop="
            () => onEscapeCustomColumn(item, 'item.actions_edit')
          "
          icon
          @keydown.tab.stop
          @click.stop="actions.edit(item)"
        >
          <v-icon>mdi-pencil-box-outline </v-icon>
        </v-btn>
        <v-sheet v-else-if="false" :color="`grey lighten-2`">
          <v-skeleton-loader height="20px" width="24px"> </v-skeleton-loader>
        </v-sheet>
        <v-btn
          v-if="actions.delete"
          icon
          class="actions_delete"
          color="red"
          @click.stop="actions.delete(item)"
          @keydown.tab.stop="(e) => !e.shiftKey && e.preventDefault()"
          @keydown.esc.stop="
            () => onEscapeCustomColumn(item, 'item.actions_delete')
          "
        >
          <v-icon>mdi-trash-can</v-icon>
        </v-btn>
        <v-sheet v-else-if="false" :color="`grey lighten-2`">
          <v-skeleton-loader height="20px" width="24px"> </v-skeleton-loader>
        </v-sheet>
      </div>
      <div class="regular" v-else :title="item[header.value]">
        {{ item[header.value] }}
      </div>
    </td>
    <td v-if="isRowExpanded" class="expanded-rows" style="padding: 10px 2%;height: 100%;width: 100%;background: #fff;">
      <slot name="expanded-item" :headers="headers" :item="item">
      </slot>
    </td>
  </tr>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class ArchDataTableRow extends Vue {
  @Prop({ default: () => {} })
  item: object | any;
  @Prop({ default: () => [] })
  headers: any[];
  @Prop({ default: () => [] })
  items: object[];
  @Prop({ default: "id" })
  itemKey: string;
  @Prop({ default: 36 })
  rowHeight: number;
  @Prop({ default: 0 })
  i: number;
  @Prop({ default: () => () => {} })
  isSelected: any;
  @Prop({ default: () => () => {} })
  select: any;
  @Prop({ default: () => () => {} })
  lastLine: any;
  @Prop({ default: () => () => {} })
  copyPrevRow: any;
  @Prop({ default: () => () => {} })
  tez: any;
  @Prop({ default: () => {} })
  actions: object;
  @Prop({ default: () => {} })
  getItemNo: any;
  @Prop({ default: null })
  activeRow: null | number;
  @Prop({ default: null })
  tableUid: number;
  @Prop({ default: [] })
  expandedRows: number[];

  loadedItems: any = [0, 0];

  get activeRowComputed() {
    return this.activeRow;
  }

  set activeRowComputed(v) {
    this.$emit("update:activeRow", v);
  }
  
  get expandedRowsComputed() {
    return this.expandedRows
  }

  set expandedRowsComputed(v) {
    this.$emit("update:expandedRows", v);
  }

  get isRowExpanded() {
    return this.expandedRowsComputed.includes(this.item.index)
  }

  onClickToggleExpand(i) {
    if(this.expandedRowsComputed.includes(i)) {
      this.expandedRowsComputed = this.expandedRowsComputed.filter(el => el !== i)
    } else {
      this.expandedRowsComputed = [...this.expandedRowsComputed, i]
    }
  }

  public onTabRow(e) {
    // @ts-ignore
    let activeColumn = localStorage.getItem(`active-column-${this.tableUid}`);
    if (e.key === "Tab" && !e.shiftKey && activeColumn) {
      e.preventDefault();
      let target =
        e.target.querySelector(`.${activeColumn.replace("item.", "")} input`) ||
        e.target.querySelector(`.${activeColumn.replace("item.", "")}`);
      if (target?.nodeName === "BUTTON") {
        target.focus();
      } else {
        target?.select();
      }
    } else if (e.key === "Tab" && e.shiftKey) {
      e.preventDefault();
    }
  }
  public onEscRow(e) {
    this.activeRowComputed = null;
    // @ts-ignore
    document?.activeElement?.blur();
    this.select(this.item, false);
  }
  public onEscapeCustomColumn(item, slot) {
    // @ts-ignore
    localStorage.setItem(`active-column-${this.tableUid}`, slot);
    // @ts-ignore
    document.querySelector(`#data-table-${this.tableUid} #row-${item[this.itemKey]}`)?.focus();
  }
  firstScopedSlotHeader = null;
  public onTabScopedSlot(e, slot) {
    let headers = this.headers;
    let scopedSlotHeaders = Object.keys(this.$scopedSlots).map((el) =>
      el.replace("item.", "")
    );
    let lastHeader = headers[headers.length - 1].value;
    if (!this.firstScopedSlotHeader) {
      for (let i = 0; i < headers.length; i++) {
        if (scopedSlotHeaders.includes(headers[i].value)) {
          this.firstScopedSlotHeader = headers[i].value;
          break;
        }
      }
    }
    if (lastHeader !== "actions" && slot.includes(lastHeader) && !e.shiftKey) {
      e.preventDefault();
    } else if (slot.includes(this.firstScopedSlotHeader) && e.shiftKey) {
      e.preventDefault();
    }
  }

  public onClickRow(item) {
    this.activeRowComputed = item.index;
  }
}
</script>

<style scoped lang="scss">
.v-data-table {
  td.sticky,
  th.sticky {
    position: sticky !important;
    right: 0;
    background: #fff !important;
    padding: 0 4px !important;
    height: 100%;
  }
}
tr {
  display: flex;
  flex: 1;
  align-items: flex-end;
  flex-basis: auto;
  width: 100%;
  td {
    display: flex;
    /* height: 100% !important; */
    div {
      width: 100%;
      overflow: hidden;
      flex: 1;
      white-space: nowrap;
      text-overflow: ellipsis;
      margin: 0;
      &.regular {
        margin: auto 0;
      }
    }
  }
}
.v-data-table table tbody tr:hover {
  background-color: #f2f2f2 !important;
}
.v-data-table table tbody tr.v-data-table__selected {
  background-color: #e0e0e0 !important;
}
.v-data-table table tbody tr:focus-visible,
.v-data-table table tbody tr:focus {
  outline-color: #c0c0c0;
  outline-width: 4px;
}
</style>
