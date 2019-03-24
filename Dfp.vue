<template>
  <div :class="`${className} ${extClass}`" :id="adunit" :pos="pos" :style="style" v-if="!isEmpty" :sectionTempId="sectionTempId"></div>
</template>
<script>
  const debug = require('debug')('CLIENT:Dfp')
  export default {
    computed: {
      adunit () {
        let _unitId = !this.unitId
          ? this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'aduid' ]
          : this.unitId
        if (this.ifDevMode) {
          _unitId = `test_${_unitId}`
        }
        return _unitId
      },
      className () {
        return this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'cont-class' ].join(' ')
      },
      currPath () {
        return this.$route.fullPath
      },
      currConfig () {
        return Object.assign({
          dfpUnits: this.dfpUnits,
          section: this.section,
          dfpId: this.dfpId,
          mode: this.mode,
          options: this.options
        }, this.config)
      },
      ifDevMode () {
        return this.config.mode === 'dev'
      },
      isEmpty () {
        return !this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ]
      },
      sectionTempId () {
        return this.currConfig.sectionTempId
      }
    },
    data () {
      return {
        style: ''
      }
    },
    name: 'Dfp',
    methods: {
      defineDfp () {
        googletag.cmd.push(() => {
          if (window.adSlots[ this.pos ]) {
            googletag.destroySlots([ window.adSlots[ this.pos ] ])
            googletag.pubads().clear([ window.adSlots[ this.pos ] ])
          }
          if (this.isEmpty) { return }
          const slot = document.querySelector(`#${this.adunit}`)
          if (slot) {
            debug(`GOING TO REMOVE SLOT#${this.adunit} LAGACY STYLE.`)
            slot.removeAttribute('style')
            slot.innerHtml = ''
            slot.setAttribute('style', this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'cont-style' ].join(';'))
          } else {
            debug(`SLOT #${this.adunit} DOESN'T EXIST AT THIS MOMENT.`)
          }
          const isSlotVisible = slot ? (slot.currentStyle ? slot.currentStyle.display : window.getComputedStyle(slot, null).display) : null
          if (isSlotVisible === 'none') {
            delete window.adSlots[ this.pos ]
            return
          }

          const isOutOfPage = this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ].outOfPage
          let _s = {}
          if (!isOutOfPage) {
            _s = googletag.defineSlot(`/${this.currConfig.dfpId}/${this.adunit}`
                      , this.getDimensions(this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'dimensions' ])
                      , this.adunit)
            try {
              _s.addService(googletag.pubads())
            } catch (err) {
              debug('UNABLED TO RENDER AD', this.adunit)
              debug('ERR', err)
              return
            }
            const mapping = (this.currConfig.options[ 'sizeMapping' ] && this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'size-mapping' ]) ? this.currConfig.options[ 'sizeMapping' ][ this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'size-mapping' ] ] : undefined
            if (mapping) {
              // Convert verbose to DFP format
              let map = googletag.sizeMapping()
              mapping.forEach((k, v) => {
                map = map.addSize(k.browser, k.ad_sizes)
              })
              _s.defineSizeMapping(map.build())
            }
            window.adSlots[ this.pos ] = _s
          } else {
            debug('##### OOP DETECTED #####')
            // _s = googletag.defineOutOfPageSlot(`/${this.currConfig.dfpId}/${this.adunit}`
            //                       , this.adunit)
            _s = googletag.defineOutOfPageSlot(`/${this.currConfig.dfpId}/${this.adunit}`, this.adunit)
            try {
              _s.addService(googletag.pubads())
            } catch (err) {
              debug('UNABLED TO RENDER OOP AD', this.adunit)
              debug('ERR', err)
              return
            }
            // _s = googletag.pubads().defineOutOfPagePassback(`/${this.currConfig.dfpId}/${this.adunit}`)
            window.adSlots[ this.pos ] = _s
            window.adSlots[ this.pos ].isOutOfPage = true
          }

          window.adSlots[ this.pos ].adId = this.adunit
          window.adSlots[ this.pos ].displayFlag = false
          window.adSlots[ this.pos ].refreshFlag = false

          googletag.cmd.push(() => {
            if (!window.adSlots[ this.pos ].refreshFlag) {
              googletag.pubads().refresh([ window.adSlots[ this.pos ] ])
              window.adSlots[ this.pos ].refreshFlag = true
            }
          })
        })
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
      },
      isClient () {
        const browser = typeof window !== 'undefined'
        return browser
      }
      // style () {
      //   return this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'cont-style' ].join(';')
      // }
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
      },
      mode: {
        default: () => { return 'prod' }
      },
      config: {
        default: () => {
          return {
            mode: 'prod'
          }
        }
      }
    },
    mounted () {
      debug(`AD ${this.pos} MOUNTED.`)
      if (window && window[ 'googletag' ] && window[ 'googletag' ][ 'apiReady' ]) {
        debug(`AD ${this.pos} IS GONNA DEFINE ITSELF.`)
        this.defineDfp()
      }
      this.style = this.currConfig && !this.isEmpty && this.currConfig.dfpUnits[ this.currConfig.section ][ this.pos ][ 'cont-style' ].join(';')
    },
    updated () {
      if (window && window[ 'googletag' ] && window[ 'googletag' ][ 'apiReady' ]) {
        debug(`AD ${this.pos} IS GONNA DEFINE ITSELF.(UPDATED)`)
        this.defineDfp()
      }
    },
    watch: {
      currPath: function () {
        if (this.isClient() && window && window[ 'googletag' ] && window[ 'googletag' ][ 'apiReady' ]) {
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
