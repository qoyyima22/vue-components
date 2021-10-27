<template>
  <span>{{displayText}}</span>
</template>

<script>
import memory from '@/storage/memory'
import _ from "lodash"
export default {
  name: 'ArchText',
  data(){
    return {
      displayText:""
    }
  },
  props:{
    table:null,
    value:null,
    itemText:null,
    itemValue:null
  },
 async mounted() {
    let items = memory.Get(this.table)
    if(items!=null){
      let filter = {}
      filter[this.itemValue]=this.value
      var item = await _.find(items, filter)
      if(item!=null){
        this.displayText=item[this.itemText]
      }
    }
  }
}
</script>

<style scoped>

</style>
