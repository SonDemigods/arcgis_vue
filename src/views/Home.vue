<template>
  <div>
    <arcgis-map :polygon="polygon"
                :point="point" />
  </div>
</template>

<script>
import ArcgisMap from '@/components/ArcgisMap.vue'

export default {
  name: 'Home',
  components: {
    ArcgisMap
  },
  props: {},
  data () {
    return {
      // 多边型
      polygon: [
        [
          [117.2046, 39.1364],
          [117.2046, 39.1464],
          [117.2146, 39.1464],
          [117.2146, 39.1364],
          [117.2046, 39.1364]
        ]
      ],
      // 点
      point: [
        {
          lon: 117.204665,
          lat: 39.136475
        }
      ]
    }
  },
  watch: {},
  computed: {},
  methods: {},
  created () { },
  mounted () { }
}
</script>
<style lang="scss" scoped>
</style>
