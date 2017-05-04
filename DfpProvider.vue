<template>
  <div>
    <slot name="dfpPos" :vueDfp="dfpPos" :dfpUnits="dfpUnits" :section="section" :dfpId="dfpid"></slot>
  </div>
</template>
<script>
  import VueDfp from './Dfp.vue'
  export default {
    components: {
      'vue-dfp': VueDfp
    },
    computed: {
      currPath () {
        return this.$route.fullPath
      },
      dfpPos () {
        return VueDfp
      }
    },
    data () {
      return {
        adsCouldNeverBeInitilized: false,
        dfpIsLoaded: false,
        dfpInstalled: false,
        dfpOptions: {
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
        },
        firstDfpRender: false
      }
    },
    name: 'vue-dfp-provider',
    methods: {
      defineDfp () {
        googletag.cmd.push(() => {
          document.querySelectorAll('.ad-container').forEach((slot) => {
            const _aduid = slot.getAttribute('id')
            const _pos = slot.getAttribute('pos')
            const _ifSlotVisible = slot.currentStyle ? slot.currentStyle.display : window.getComputedStyle(slot, null).display

            if (window.adSlots[ _pos ]) {
              googletag.destroySlots([ window.adSlots[ _pos ] ])
              googletag.pubads().clear([ window.adSlots[ _pos ] ])
            }

            if (_ifSlotVisible === 'none') {
              delete window.adSlots[ _pos ]
              return
            }
            const _s = googletag.defineSlot(`/${this.dfpid}/${_aduid}`
                                  , this.getDimensions(this.dfpUnits[ this.section ][ _pos ][ 'dimensions' ])
                                  , _aduid)
            try {
              _s.addService(googletag.pubads())
            } catch (err) {
              console.log('unabled to render ad', _aduid)
              console.log('err', err)
              return
            }
            const mapping = (this.options[ 'sizeMapping' ] && this.dfpUnits[ this.section ][ _pos ][ 'size-mapping' ]) ? this.options[ 'sizeMapping' ][ this.dfpUnits[ this.section ][ _pos ][ 'size-mapping' ] ] : undefined
            if (mapping) {
              // Convert verbose to DFP format
              let map = googletag.sizeMapping()
              mapping.forEach((k, v) => {
                map = map.addSize(k.browser, k.ad_sizes)
              })
              _s.defineSizeMapping(map.build())
            }
            window.adSlots[ _pos ] = _s
            window.adSlots[ _pos ].adId = _aduid
            window.adSlots[ _pos ].displayFlag = false
            window.adSlots[ _pos ].refreshFlag = false
          })
        })
      },
      loadDfp () {
        return new Promise((resolve) => {
          this.dfpIsLoaded = this.dfpIsLoaded || document.querySelectorAll('script[src*="googletagservices.com/tag/js/gpt.js"]').length
          if (this.dfpIsLoaded) {
            this.dfpInstalled = true
            window.dfpInstalled = true
            return resolve()
          }
          window.googletag = window.googletag || {}
          window.googletag.cmd = window.googletag.cmd || []
          const gads = document.createElement('script')
          gads.async = true
          gads.type = 'text/javascript'
          // Adblock blocks the load of Ad scripts... so we check for that
          gads.onerror = () => {
            this.dfpBlocked()
            this.adsCouldNeverBeInitilized = true
            resolve()
          }
          // gads.onload = function() {
          //   // this will work with ghostery:
          //   if (!googletag._loadStarted_) {
          //     googletag._adBlocked_ = true
          //   }
          //   this.dfpIsLoaded = true
          //   resolve()
          // }
          // take https://developers.google.com/doubleclick-gpt/common_implementation_mistakes for ref and revise the above codes to below codes
          googletag.cmd.push(() => {
            if (!googletag._loadStarted_) {
              googletag._adBlocked_ = true
            }
            this.dfpIsLoaded = true
            window.dfpIsLoaded = true
            this.dfpInstalled = true
            window.dfpInstalled = true
            resolve()
          })

          const useSSL = document.location.protocol === 'https:'
          gads.src = (useSSL ? 'https:' : 'http:') +
          '//www.googletagservices.com/tag/js/gpt.js'
          const node = document.getElementsByTagName('script')[ 0 ]
          node.parentNode.insertBefore(gads, node)
        })
      },
      initDfp () {
        window.googletag = window.googletag || {}
        googletag.cmd = googletag.cmd || []
        this.dfpOptions = Object.assign(this.dfpOptions, this.options)

        googletag.cmd.push(() => {
          const pubadsService = googletag.pubads()
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.pubads **/

          if (this.dfpOptions.setForceSafeFrame) {
            pubadsService.setForceSafeFrame(true)
          }

          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_enableSingleRequest **/
          if (this.dfpOptions.enableSingleRequest) {
            pubadsService.enableSingleRequest()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setTargeting **/
          for (const x in this.dfpOptions.setTargeting) {
            pubadsService.setTargeting(x, this.dfpOptions.setTargeting[x])
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setLocation **/
          const setLocation = this.dfpOptions.setLocation
          if (typeof setLocation === 'object') {
            if (typeof setLocation.latitude === 'number' && typeof setLocation.longitude === 'number' &&
              typeof setLocation.precision === 'number') {
              pubadsService.setLocation(setLocation.latitude, setLocation.longitude, setLocation.precision)
            } else if (typeof setLocation.latitude === 'number' && typeof setLocation.longitude === 'number') {
              pubadsService.setLocation(setLocation.latitude, setLocation.longitude)
            }
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setCategoryExclusion **/
          if (this.dfpOptions.setCategoryExclusion.length > 0) {
            const exclusionsGroup = this.dfpOptions.setCategoryExclusion.split(',')
            let valueTrimmed
            exclusionsGroup.forEach((v) => {
              valueTrimmed = v.trim()
              if (valueTrimmed.length > 0) {
                pubadsService.setCategoryExclusion(valueTrimmed)
              }
            })
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_collapseEmptyDivs **/
          if (this.dfpOptions.collapseEmptyDivs) {
            pubadsService.collapseEmptyDivs()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.disablePublisherConsole **/
          if (this.dfpOptions.disablePublisherConsole) {
            pubadsService.disablePublisherConsole()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.CompanionAdsService_setRefreshUnfilledSlots **/
          if (this.dfpOptions.companionAds) {
            googletag.companionAds().setRefreshUnfilledSlots(true)
            /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_enableVideoAds **/
            if (!this.dfpOptions.disableInitialLoad) {
              pubadsService.enableVideoAds()
            }
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_disableInitialLoad **/
          if (this.dfpOptions.disableInitialLoad) {
            pubadsService.disableInitialLoad()
          }
          /** no found in googletag doc **/
          if (this.dfpOptions.noFetch) {
            // pubadsService.noFetch()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setCentering **/
          if (this.dfpOptions.setCentering) {
            pubadsService.setCentering(true)
          }
          if (this.dfpOptions[ 'afterEachAdLoaded' ]) {
            googletag.pubads().addEventListener('slotRenderEnded', this.dfpOptions[ 'afterEachAdLoaded' ])
          }
          googletag.enableServices()
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
                dimensions.push(dimensionSet[ 0 ])
              }
            }
          })
        } else {
          dimensions.push([ this.$el.offsetWidth, this.$el.offsetHeight ])
        }
        return dimensions
      },
      removeDfp () {
        for (const slotPos in window.adSlots) {
          googletag.destroySlots([ window.adSlots[ slotPos ] ])
          document.querySelector(`#${window.adSlots[slotPos].adId}`).innerHtml = ''
        }
        window.adSlots = {}
        document.querySelector('head').removeChild(document.querySelector('script[src*="googletagservices.com/tag/js/gpt.js"]'))
        delete window.googletag
        this.dfpInstalled = false
        this.dfpIsLoaded = false
        window.dfpInstalled = false
        window.dfpIsLoaded = false
      }
    },
    mounted () {
      this.loadDfp().then(() => {
        if (this.dfpInstalled === true) {
          googletag.cmd.push(() => {
            googletag.destroySlots()
          })
          // this.dfpInstalled = false
          // window.dfpInstalled = true
          // return
        }
        if (!window.adSlots) {
          window.adSlots = {}
        }
        this.initDfp()
        this.defineDfp()
        googletag.cmd.push(() => {
          for (const slotPos in window.adSlots) {
            if (!window.adSlots[slotPos].displayFlag) {
              googletag.display(window.adSlots[slotPos].adId)
              window.adSlots[slotPos].displayFlag = true
            }
          }
          for (const slotPos in window.adSlots) {
            if (!window.adSlots[slotPos].refreshFlag) {
              if (this.dfpOptions.disableInitialLoad === true) {
                googletag.pubads().refresh([ window.adSlots[slotPos] ])
              }
              window.adSlots[slotPos].refreshFlag = true
            }
          }
        })
        this.firstDfpRender = true
      })
    },
    props: {
      dfpid: {
        default: () => { return '' }
      },
      dfpUnits: {
        default: () => { return {} }
      },
      options: {
        default: () => { return {} }
      },
      section: {
        default: () => { return 'default' }
      }
    },
    watch: {
      currPath: function () {
        googletag.destroySlots()
      }
    },
    updated () {
      if (this.dfpInstalled !== true || this.firstDfpRender !== true) {
        return
      } else {
        googletag.cmd.push(() => {
          // for (const slotPos in window.adSlots) {
          //   if (!window.adSlots[slotPos].displayFlag) {
          //     googletag.display(window.adSlots[slotPos].adId)
          //     window.adSlots[slotPos].displayFlag = true
          //   }
          // }
          for (const slotPos in window.adSlots) {
            if (!window.adSlots[slotPos].refreshFlag) {
              googletag.pubads().refresh([ window.adSlots[slotPos] ])
              window.adSlots[slotPos].refreshFlag = true
            }
          }
        })
      }
    }
  }
</script>
<style></style>
