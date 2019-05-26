<template>
  <div style="margin-top: 10px">
    <div>
      <el-checkbox-group v-model="chartSettings.dimension" :min="1" :max="2">
        <el-checkbox label="日期"></el-checkbox>
      </el-checkbox-group>

      <el-checkbox-group v-model="chartSettings.metrics" :min="1">
<!--        <el-checkbox label="日期" disabled></el-checkbox>-->
        <el-checkbox label="访问用户"></el-checkbox>
        <el-checkbox label="下单用户"></el-checkbox>
        <el-checkbox label="下单率"></el-checkbox>
        <el-checkbox label="退货率"></el-checkbox>
<!--        <el-checkbox label="选中且禁用" disabled></el-checkbox>-->
      </el-checkbox-group>
    </div>
    <div style="border:1px solid #bfcbd9;;margin: 10px;">
      <ve-histogram :data="chartData" :settings="chartSettings" :extend="extend"/>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Index',
  data() {
    // this.chartSettings = {
    //   axisSite: { right: ['下单率'] },
    //   yAxisType: ['KMB', 'percent'],
    //   yAxisName: ['数值', '比率']
    // }
    return {
      chartSettings: {
        metrics: ['访问用户'],
        dimension: ['日期'],
        axisSite: { right: [] },
        yAxisType: ['KMB', 'percent'],
        yAxisName: ['数值', '比率']
      },
      extend: {
        series: {
          label: { show: true, position: 'top' }
        }
      },
      chartData: {
        columns: ['日期', '访问用户', '下单用户', '下单率', '退货率'],
        rows: [
          { '日期': '1/1', '访问用户': 1393, '下单用户': 1093, '下单率': 0.32, '退货率': 0.02 },
          { '日期': '1/2', '访问用户': 3530, '下单用户': 3230, '下单率': 0.26, '退货率': 0.05 },
          { '日期': '1/3', '访问用户': 2923, '下单用户': 2623, '下单率': 0.76, '退货率': 0.08 },
          { '日期': '1/4', '访问用户': 1723, '下单用户': 1423, '下单率': 0.49, '退货率': 0.12 },
          { '日期': '1/5', '访问用户': 3792, '下单用户': 3492, '下单率': 0.323, '退货率': 0.01 },
          { '日期': '1/6', '访问用户': 4593, '下单用户': 4293, '下单率': 0.78, '退货率': 0.04 }
        ]
      },
      checkList: ['日期']
    }
  },
  watch: {
    'chartSettings.metrics': {
      handler(newval, ordval) {
        const percentTypeMap = ['下单率', '退货率']
        console.log(newval)
        if (newval) {
          newval.forEach((col) => {
            if (percentTypeMap.indexOf(col) !== -1) {
              if (this.chartSettings.axisSite) {
                this.chartSettings.axisSite.right.push(col)
              }
            }
          })
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
