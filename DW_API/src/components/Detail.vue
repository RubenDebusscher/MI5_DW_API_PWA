<template>
  <Detail>
      <modal name="size-modal" transition="nice-modal-fade" classes="demo-modal-class" :z-index="5" :min-width="400" :min-height="200" :pivot-y="0.5" :adaptive="true" :scrollable="true" :reset="true" width="60%" height="auto">
        <div class="size-modal-content">
          <h1>{{titel}}</h1>
          <img  class="wikifoto" alt="hier komt de (cover)foto"/>
          <div></div>
        </div>
      </modal>
  </Detail>
</template>
<script>
export default {
  data () {
    return {
      titel: [ 'Dit is de titel' ]
    }
  }
}
</script>
<style scoped>
</style>
