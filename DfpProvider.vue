<template>
  <div>
    <slot name="dfpPos" :vueDfp="dfpPos" :dfpUnits="dfpUnits" :section="section"></slot>
  </div>
</template>
<script>
  import VueDfp from './Dfp.vue'
  export default {
    components: {
      'vue-dfp': VueDfp
    },
    computed: {
      dfpPos() {
        return VueDfp
      }
    },
    data() {
      return {
        adsCouldNeverBeInitilized: false,
        dfpIsLoaded: false,
        installed: false
      }
    },
    name: 'vue-dfp-provider',
    methods: {
      defineDfp() {
        let slots = [];
        document.querySelectorAll('.ad-container').forEach((slot) => {
            const _s = googletag.defineSlot(`/${this.dfpid}/${this.dfpUnits[ this.section ][ slot.getAttribute('pos') ][ 'aduid' ]}`
                                  , this.getDimensions(this.dfpUnits[ this.section ][ slot.getAttribute('pos') ][ 'dimensions' ])
                                  , this.dfpUnits[ this.section ][ slot.getAttribute('pos') ][ 'aduid' ]).addService(googletag.pubads());
            slots.push(_s)
            googletag.display(this.dfpUnits[ this.section ][ slot.getAttribute('pos') ][ 'aduid' ]);
            // googletag.pubads().refresh([_s]);

        })
        // googletag.pubads().refresh(slots);
      },
      loadDfp() {
        return new Promise((resolve) => {
          this.dfpIsLoaded = this.dfpIsLoaded || document.querySelectorAll('script[src*="googletagservices.com/tag/js/gpt.js"]').length
          if (this.dfpIsLoaded) {
            return Promise.resolve()
          }
          window.googletag = window.googletag || {}
          window.googletag.cmd = window.googletag.cmd || []
          let gads = document.createElement('script')
          gads.async = true
          gads.type = 'text/javascript'
          // Adblock blocks the load of Ad scripts... so we check for that
          gads.onerror = () => {
            this.dfpBlocked()
            this.adsCouldNeverBeInitilized = true
            resolve()
          }
          gads.onload = function() {
            // this will work with ghostery:
            if (!googletag._loadStarted_) {
              googletag._adBlocked_ = true
            }
            this.dfpIsLoaded = true
            resolve()
          }
          let useSSL = 'https:' === document.location.protocol
          gads.src = (useSSL ? 'https:' : 'http:') +
          '//www.googletagservices.com/tag/js/gpt.js'
          let node = document.getElementsByTagName('script')[0]
          node.parentNode.insertBefore(gads, node)
        })
      },
      initDfp() {
        window.googletag = window.googletag || {};
        googletag.cmd = googletag.cmd || [];
        let dfpOptions = {
          dfpID: '',
          setTargeting: {},
          setCategoryExclusion: '',
          setLocation: '',
          enableSingleRequest: true,
          collapseEmptyDivs: 'original',
          companionAds: true,
          refreshExisting: true,
          disablePublisherConsole: false,
          disableInitialLoad: false,
          setCentering: false,
          noFetch: false,
          namespace: undefined,
          sizeMapping: [],
          afterAdBlocked: undefined,
          afterEachAdLoaded: undefined,
          afterAllAdsLoaded: undefined,
          rendered: 0,
          onloaded: [],
          adUnits: []
        }
        googletag.cmd.push(() => {
          let pubadsService = googletag.pubads()
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.pubads **/
          pubadsService.setForceSafeFrame(true)
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_enableSingleRequest **/
          if (dfpOptions.enableSingleRequest) {
            pubadsService.enableSingleRequest()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setTargeting **/
          for (let x in dfpOptions.setTargeting) {
            pubadsService.setTargeting(x, dfpOptions.setTargeting[x])
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setLocation **/
          const setLocation = dfpOptions.setLocation
          if (typeof setLocation === 'object') {
            if (typeof setLocation.latitude === 'number' && typeof setLocation.longitude === 'number' &&
              typeof setLocation.precision === 'number') {
              pubadsService.setLocation(setLocation.latitude, setLocation.longitude, setLocation.precision);
            } else if (typeof setLocation.latitude === 'number' && typeof setLocation.longitude === 'number') {
              pubadsService.setLocation(setLocation.latitude, setLocation.longitude)
            }
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setCategoryExclusion **/
          if (dfpOptions.setCategoryExclusion.length > 0) {
            const exclusionsGroup = dfpOptions.setCategoryExclusion.split(',')
            let valueTrimmed
            exclusionsGroup.forEach((v) => {
              valueTrimmed = v.trim()
              if (valueTrimmed.length > 0) {
                pubadsService.setCategoryExclusion(valueTrimmed)
              }
            })
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_collapseEmptyDivs **/
          if (dfpOptions.collapseEmptyDivs) {
            pubadsService.collapseEmptyDivs()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.disablePublisherConsole **/
          if (dfpOptions.disablePublisherConsole) {
            pubadsService.disablePublisherConsole()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.CompanionAdsService_setRefreshUnfilledSlots **/
          if (dfpOptions.companionAds) {
            googletag.companionAds().setRefreshUnfilledSlots(true)
            /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_enableVideoAds **/
            if (!dfpOptions.disableInitialLoad) {
              pubadsService.enableVideoAds()
            }
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_disableInitialLoad **/
          if (dfpOptions.disableInitialLoad) {
            pubadsService.disableInitialLoad()
          }
          /** no found in googletag doc **/
          if (dfpOptions.noFetch) {
            // pubadsService.noFetch()
          }
          /** https://developers.google.com/doubleclick-gpt/reference#googletag.PubAdsService_setCentering **/
          if (dfpOptions.setCentering) {
            pubadsService.setCentering(true)
          }
          googletag.enableServices()
        })
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
    mounted() {
      this.loadDfp()
      window.addEventListener('load', (e) => {
        this.initDfp()
        this.defineDfp()
      })
      document.addEventListener('DOMContentLoaded', () => {})
      if(googletag && googletag.apiReady) {
        googletag.destroySlots()
        this.initDfp()
        this.defineDfp()
      }
    },
    props: {
      dfpid: {
        default: () => { return '' }
      },
      dfpUnits: {
        default: () => { return {} }
      },
      section: {
        default: () => { return 'default' }
      },
    },
  }
</script>
<style></style>