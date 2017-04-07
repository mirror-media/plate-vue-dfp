<template>
  <div :class="`${className} ${extClass}`" :id="adunit" :pos="pos" :style="style"></div>
</template>
<script>
  export default {
    computed: {
      adunit() {
        return !this.unitId ? this.dfpUnits[ this.section ][ this.pos ][ 'aduid' ] : this.unitId
      },
      className() {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-class' ].join(' ')
      },
      style() {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-style' ].join(';')
      }
    },
    name: 'ad-container',
    methods: {
      defineDfp() {
        const _s = googletag.defineSlot(`/${this.dfpId}/${this.adunit}`
                              , this.getDimensions(this.dfpUnits[ this.section ][ this.pos ][ 'dimensions' ])
                              , this.dfpUnits[ this.section ][ this.pos ][ 'aduid' ]).addService(googletag.pubads());
        googletag.display(this.adunit);
        googletag.pubads().refresh([_s]);
      },
      getDimensions (dimes) {
        let dimensions = []
        if (typeof dimes !== 'undefined' && dimes !== '') {
          dimes.split(',').forEach((v) => {
            const dimensionSet = v.split('x')
            if(dimensionSet.length > 1) {
              dimensions.push([parseInt(dimensionSet[0], 10), parseInt(dimensionSet[1], 10)])
            } else {
              if(dimensionSet[0] === 'fluid') {
                dimensions.push(dimensionSet[0])
              }
            }
          })
        } else {
          dimensions.push([this.$el.offsetWidth, this.$el.offsetHeight])
        }
        return dimensions
      },
    },
    props: {
      extClass: {
        default: () => { return '' }
      },
      dfpId: {
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
      },
      unitId: {
        default: () => { return undefined }
      }
    },
    updated() {
      if(googletag && googletag.apiReady) {
        this.defineDfp()
      }
    },
  }
</script>
<style scoped>
  .ad-container { margin-top: 20px; }
  .ad-container.margin-top-30px { margin-top: 30px; }
  .ad-container.margin-top-0 { margin-top: 0; }
  .ad-container.center { display: flex; justify-content: center; margin-right: auto; margin-left: auto; }  
</style>