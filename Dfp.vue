<template>
  <div :class="`${className} ${extClass}`" :id="adunit" :adunit="adunit" :pos="pos" :style="style"></div>
</template>
<script>
  export default {
    computed: {
      adunit() {
        return this.dfpUnits[ this.section ][ this.pos ][ 'aduid' ]
      },
      className() {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-class' ].join(' ')
      },
      style() {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-style' ].join(';')
      }
    },
    name: 'ad-container',
    mounted() {
    },
    props: {
      extClass: {
        default: () => { return '' }
      },
      dfpUnits: {
        default: () => { return '' }
      },
      pos: {
        default: () => { return '' }
      },
      section: {
        default: () => { return 'default' }
      }
    }
  }
</script>
<style scoped>
  .ad-container { display: inline-block; margin-top: 20px; }
  .ad-container.margin-top-30px { margin-top: 30px; }
  .ad-container.margin-top-0 { margin-top: 0; }
  .ad-container.center { display: flex; justify-content: center; }  
</style>