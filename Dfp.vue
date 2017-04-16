<template>
  <div :class="`${className} ${extClass}`" :id="adunit" :pos="pos" :style="style"></div>
</template>
<script>
  export default {
    computed: {
      adunit () {
        return !this.unitId ? this.dfpUnits[ this.section ][ this.pos ][ 'aduid' ] : this.unitId
      },
      className () {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-class' ].join(' ')
      },
      currPath () {
        return this.$route.fullPath
      },
      style () {
        return this.dfpUnits[ this.section ][ this.pos ][ 'cont-style' ].join(';')
      }
    },
    name: 'ad-container',
    methods: {
      defineDfp () {
        const slot = document.querySelector(`#${this.adunit}`)
        const _ifSlotVisible = slot ? (slot.currentStyle ? slot.currentStyle.display : window.getComputedStyle(slot, null).display) : null
        if (_ifSlotVisible === 'none') { return }
        const _s = googletag.defineSlot(`/${this.dfpId}/${this.adunit}`
                              , this.getDimensions(this.dfpUnits[ this.section ][ this.pos ][ 'dimensions' ])
                              , this.adunit).addService(googletag.pubads())
        const mapping = (this.options[ 'sizeMapping' ] && this.dfpUnits[ this.section ][ this.pos ][ 'size-mapping' ]) ? this.options[ 'sizeMapping' ][ this.dfpUnits[ this.section ][ this.pos ][ 'size-mapping' ] ] : undefined
        if (mapping) {
            // Convert verbose to DFP format
          let map = googletag.sizeMapping()
          mapping.forEach((k, v) => {
            map = map.addSize(k.browser, k.ad_sizes)
          })
          _s.defineSizeMapping(map.build())
        }
        // googletag.display(this.adunit);
        googletag.pubads().refresh([ _s ])
      },
      getDimensions (dimes) {
        const dimensions = []
        if (typeof dimes !== 'undefined' && dimes !== '') {
          dimes.split(',').forEach((v) => {
            const dimensionSet = v.split('x')
            if (dimensionSet.length > 1) {
              dimensions.push([ parseInt(dimensionSet[ 0 ], 10), parseInt(dimensionSet[ 1 ], 10) ])
            } else {
              if (dimensionSet[ 0 ] === 'fluid') {
                dimensions.push(dimensionSet[0])
              }
            }
          })
        } else {
          dimensions.push([ this.$el.offsetWidth, this.$el.offsetHeight ])
        }
        return dimensions
      }
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
      options: {
        default: () => {
          return {
            dfpID: '',
            setTargeting: {},
            setCategoryExclusion: '',
            setLocation: '',
            enableSingleRequest: true,
            collapseEmptyDivs: 'original',
            refreshExisting: true,
            disablePublisherConsole: false,
            disableInitialLoad: true,
            setCentering: false,
            setForceSafeFrame: false,
            noFetch: false,
            namespace: undefined,
            sizeMapping: undefined,
            afterAdBlocked: undefined,
            afterEachAdLoaded: undefined,
            afterAllAdsLoaded: undefined,
            rendered: 0,
            onloaded: [],
            adUnits: []
          }
        }
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
    mounted () {
      if (window && window[ 'googletag' ] && window[ 'googletag' ][ 'apiReady' ]) {
        this.defineDfp()
        googletag.display(this.adunit)
      }
    },
    updated () {
      if (window && window[ 'googletag' ] && window[ 'googletag' ][ 'apiReady' ]) {
        googletag.display(this.adunit)
      }
    },
    watch: {
      currPath: function () {
        if (googletag && googletag.apiReady) {
          this.defineDfp()
        }
      }
    }
  }
</script>
<style scoped>
  .ad-container { margin-top: 20px; }
  .ad-container.margin-top-30px { margin-top: 30px; }
  .ad-container.margin-top-0 { margin-top: 0; }
  .ad-container.center { display: flex; justify-content: center; margin-right: auto; margin-left: auto; }  
</style>
