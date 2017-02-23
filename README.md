# kc-vue-dfp
A package for google dfp in framwork vue.

# lib/index.js
This package is inspired by [dfp-vue](https://github.com/hinablue/dfp-vue) as well as [埋設 DFP - DoubleClick for Publishers 廣告](https://amobiz.github.io/2016/08/11/react-dfp/), and takes both of them as reference.

# Usage

Installing this package from npm like:

  `$ npm install kc-vue-dfp@latest --save `

Then, just import it like:

  `import VueDfpProvider from 'kc-vue-dfp';`

And with a few steps like:

1、 have `VueDfpProvider` be a component:
```
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

2、 have to compose an object, its structure looks like:
```
adunits: {
  'section-id': {
    'POSITION1': {
      'aduid': 'ADUNIT-ID-1'
      , 'dimensions': '970x250'
      , 'cont-class': [ 'class-1', 'center' ] 
    },
    'POSITION2': { 
      'aduid': 'ADUNIT-ID-2'
      , 'dimensions':'300x250,320x250'
      , 'cont-class': [ 'class-1' ] 
    }
  }
}
```
Put object `adunits` to data:
```
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

3、 do some markups in template:
```
<template>
  <vue-dfp-provider :dfpUnits="dfpUnits" :dfpid="dfpid" :section="sectionId">
    <template scope="props" slot="dfpPos">
      <vue-dfp :is="props.vueDfp" extclass="ext-class-1" pos="POSITION1" :dfpUnits="props.dfpUnits" :section="props.section" />
      <vue-dfp :is="props.vueDfp" extclass="ext-class-2 ext-class-3" pos="POSITION2" :dfpUnits="props.dfpUnits" :section="props.section" />
    </template>
  </vue-dfp-provider>
</template>
```

Then, the template will be rendered to:
```
<div>
  <div class="class-1 center ext-class-1" id="ADUNIT-ID-1" adunit="ADUNIT-ID-1" pos="POSITION1"></div> 
  <div class="class-2 ext-class-2 ext-class-3" id="ADUNIT-ID-2" adunit="ADUNIT-ID-2" pos="POSITION2"></div>
</div>
```

As soon as the DFP code is loaded, the DFP ad will be show correctly.


# More
See how to use it in [mirror-media/plate-vue](https://github.com/mirror-media/plate-vue).