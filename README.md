# plate-vue-dfp
A package for google dfp in framework vue.

This package is inspired by <U>[dfp-vue](https://github.com/hinablue/dfp-vue)</U> as well as <U>[埋設 DFP - DoubleClick for Publishers 廣告](https://amobiz.github.io/2016/08/11/react-dfp/)</U>, and takes both of them as reference.

# Usage

Installing this package from npm like:

  `$ npm install plate-vue-dfp@latest --save `

Then, just import it like:

  `import VueDfpProvider from 'plate-vue-dfp/DfpProvider.vue';`


Do with following steps in any vue file where you want to put dfp ad:
### 1. have `VueDfpProvider` be a component:
```javascript
<script>
  export default {
    ...
    components: {
      ...
      VueDfpProvider,
      ...
    },
    ...
  }
</script>
```

### 2、 have to compose an object, its structure looks like:
```javascript
adunits: {
  'section-id': {
    'POSITION1': {
      'aduid': 'ADUNIT-ID-1'
      , 'dimensions': '970x250'
      , 'cont-class': [ 'ad-container', 'class-1', 'center' ]
      // notice: class 'ad-container' is required.
      , 'cont-style': [ '' ]
    },
    'POSITION2': { 
      'aduid': 'ADUNIT-ID-2'
      , 'dimensions':'300x250,320x250'
      , 'cont-class': [ 'ad-container', 'class-1' ] 
      , 'cont-style': [ '' ]
    }
  },
  'section-id2': {
    ...
  }
}
```

then, put object `adunits` to data:

```javascript
  export default {
    ...
    data() {
      return {
        dfpid: '40170002',
        dfpUnits: adunits,
        sectionId: 'section-id'
      }
    },
    ...
  }
```

### 3、 do some markups in template:
```html
<template>
  <vue-dfp-provider :dfpUnits="dfpUnits" :dfpid="dfpid" :section="sectionId" :options="dfpOptions" >
    <template scope="props" slot="dfpPos">
      <!-- Put all markups in this slot would be a better way to use this pack. And, "<vue-dfp ... />" is the primary component to place DFP ads -->
      <vue-dfp :is="props.vueDfp" extclass="ext-class-1" pos="POSITION1" :dfpUnits="props.dfpUnits" :section="props.section" />
      <vue-dfp :is="props.vueDfp" extclass="ext-class-2 ext-class-3" pos="POSITION2" :dfpUnits="props.dfpUnits" :section="props.section" />
    </template>
  </vue-dfp-provider>
</template>
```

Then, the template above will be rendered to:
```html
<div>
  <div class="class-1 center ext-class-1" id="ADUNIT-ID-1" adunit="ADUNIT-ID-1" pos="POSITION1"></div> 
  <div class="class-2 ext-class-2 ext-class-3" id="ADUNIT-ID-2" adunit="ADUNIT-ID-2" pos="POSITION2"></div>
</div>
```

As soon as the DFP code is loaded, the DFP ad will be show correctly.


# How to override the DFP options
Put the props 'options' into `<vue-dfp-provider>...</vue-dfp-provider>` like:

```html
<vue-dfp-provider :dfpUnits="dfpUnits" :dfpid="dfpid" section="home" :options="dfpOptions">
    ...
</vue-dfp-provider>
```

And the options would be an object like:

```javascript
{
      setTargeting: {},
      setCategoryExclusion: '',
      setLocation: {},
      enableSingleRequest: true,
      collapseEmptyDivs: 'original',
      companionAds: true,
      refreshExisting: true,
      disablePublisherConsole: false,
      disableInitialLoad: true,
      setCentering: false,
      noFetch: false,
      afterEachAdLoaded: (event) => {},
}
```

What a pity is this tool doesn't implement all options of DFP_OPTIONS yet at this moment. If any, we could implement them for you.


# More
See how to use it in [mirror-media/plate-vue](https://github.com/mirror-media/plate-vue).
