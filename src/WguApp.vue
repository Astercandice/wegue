<template>
  <v-app id="wgu-app" data-app :class="{ 'wgu-app': true, 'wgu-app-embedded': isEmbedded }">

    <wgu-app-header :color="baseColor" />

    <wgu-app-logo />

    <v-content>
      <v-container id="ol-map-container" fluid fill-height style="padding: 0">
         <wgu-map :color="baseColor" />
         <!-- layer loading indicator -->
         <wgu-maploading-status :color="baseColor" />
      </v-container>
    </v-content>

    <template v-for="(moduleWin, index) in moduleWins">
      <component
        :is="moduleWin.type" :key="index" :ref="moduleWin.type"
        :color="baseColor"
        :draggable="moduleWin.draggable"
        :initPos="moduleWin.initPos"
    />
    </template>

    <wgu-app-footer
      :color="baseColor"
      :footerTextLeft="footerTextLeft"
      :footerTextRight="footerTextRight"
      :showCopyrightYear="showCopyrightYear"
    />

  </v-app>
</template>

<script>
  import Vue from 'vue'
  import { WguEventBus } from './WguEventBus'
  import OlMap from './components/ol/Map'
  import AppHeader from './components/AppHeader'
  import AppFooter from './components/AppFooter'
  import AppLogo from './components/AppLogo'
  import MeasureWin from './components/measuretool/MeasureWin'
  import LayerListWin from './components/layerlist/LayerListWin'
  import InfoClickWin from './components/infoclick/InfoClickWin'
  import MapLoadingStatus from './components/progress/MapLoadingStatus'

  export default {
    name: 'wgu-app',
    components: {
      'wgu-map': OlMap,
      'wgu-app-header': AppHeader,
      'wgu-app-footer': AppFooter,
      'wgu-app-logo': AppLogo,
      'wgu-measuretool-win': MeasureWin,
      'wgu-layerlist-win': LayerListWin,
      'wgu-infoclick-win': InfoClickWin,
      'wgu-maploading-status': MapLoadingStatus
    },
    data () {
      return {
        isEmbedded: false,
        moduleWins: this.getModuleWinData(),
        footerTextLeft: Vue.prototype.$appConfig.footerTextLeft,
        footerTextRight: Vue.prototype.$appConfig.footerTextRight,
        showCopyrightYear: Vue.prototype.$appConfig.showCopyrightYear,
        baseColor: Vue.prototype.$appConfig.baseColor
      }
    },
    mounted () {
      // apply the isEmbedded state to the member var
      this.isEmbedded = this.$isEmbedded;

      // make the refs (floating module window, which are not connected to their
      // related components, e.g. buttons to toggle them)
      const refs = this.$refs;
      let cmpLookup = {};
      for (const key of Object.keys(refs)) {
        cmpLookup[key] = refs[key][0];
      }
      Vue.prototype.cmpLookup = cmpLookup;
      // inform registered cmps that the app is mounted and the dynamic
      // components are available
      WguEventBus.$emit('app-mounted');
    },
    methods: {
      /**
       * Determines the module window configuration objects from app-config:
       *     moduleWins: [
       *       {type: 'wgu-layerlist-win'},
       *       {type: 'wgu-measuretool-win'}
       *     ]
       * @return {Array} module window configuration objects
       */
      getModuleWinData () {
        const appConfig = Vue.prototype.$appConfig;
        let moduleWins = [];
        for (const key of Object.keys(appConfig.modules)) {
          const moduleOpts = appConfig.modules[key];
          if (moduleOpts.win === true) {
            moduleWins.push({
              type: key + '-win',
              draggable: moduleOpts.draggable,
              initPos: moduleOpts.initPos
            });
          }
        }
        return moduleWins;
      }
    }
  }
</script>
