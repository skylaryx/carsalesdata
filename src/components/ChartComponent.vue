<template>
  <div>
    <el-container>
      <el-header>
        <h1>汽车销量数据分析</h1>
      </el-header>
      <el-main>
        <el-row :gutter="20">
          <el-col :span="12">
            <div ref="totalSalesChart" style="width: 100%; height: 400px;"></div>
          </el-col>
          <el-col :span="12">
            <div ref="priceRangeChart" style="width: 100%; height: 400px;"></div>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <div ref="brandTopChart" style="width: 100%; height: 400px;"></div>
          </el-col>
          <el-col :span="12">
            <div ref="typeMonthlyTrendChart" style="width: 100%; height: 400px;"></div>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8">
            <div ref="typeTotalSalesChart" style="width: 100%; height: 400px;"></div>
          </el-col>
          <el-col :span="8">
            <div ref="seriesTotalSalesChart" style="width: 100%; height: 400px;"></div>
          </el-col>
          <el-col :span="8">
            <div ref="mpvSeriesSalesChart" style="width: 100%; height: 400px;"></div>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <div ref="seriesMonthlyTrendChart" style="width: 100%; height: 400px;"></div>
          </el-col>
          <el-col :span="12">
            <div ref="seriesTopChart" style="width: 100%; height: 400px;"></div>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="24">
            <el-card style="max-height: 400px; overflow-y: auto;">
              <div slot="header" class="fixed-header">
                <span>汽车销量排行榜</span>
                <el-select v-model="selectedMonth" placeholder="选择月份" @change="handleMonthChange">
                  <el-option v-for="month in months" :key="month" :label="month" :value="month"></el-option>
                </el-select>
              </div>
              <el-table :data="rankings" stripe style="width: 100%">
                <el-table-column prop="排名" label="销量排名" width="100"></el-table-column>
                <el-table-column prop="汽车图片" label="汽车图片" width="120">
                  <template v-slot="scope">
                    <img v-if="scope.row && scope.row.汽车图片" :src="scope.row.汽车图片" alt="Car Image" style="width: 80px; height: 60px;">
                    <span v-else>暂无图片</span>
                  </template>
                </el-table-column>
                <el-table-column prop="车名" label="车名"></el-table-column>
                <el-table-column prop="品牌" label="品牌"></el-table-column>
                <el-table-column prop="车型" label="车型"></el-table-column>
                <el-table-column prop="车系" label="车系"></el-table-column>
                <el-table-column prop="价格" label="价格"></el-table-column>
                <el-table-column prop="销量" label="销量"></el-table-column>
              </el-table>
            </el-card>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>


<script>
import * as echarts from 'echarts';
import Papa from 'papaparse';

export default {
  data() {
    return {
      csvData: null,
      selectedMonth: null,
      months: [],
      rankings: []
    };
  },
  mounted() {
    this.fetchAndProcessCSV();
  },
  methods: {
    async fetchAndProcessCSV() {
      try {
        const csvPath = '/sorted_data_with_series2.csv'; // 请确认路径是否正确
        const response = await fetch(csvPath);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const csvText = await response.text();
        console.log('Fetched CSV text:', csvText);

        Papa.parse(csvText, {
          header: true,
          complete: (result) => {
            console.log('Parsed CSV data:', result.data);
            // 数据加载完成后，将销量转换为数字类型
            this.csvData = result.data.map(car => ({
              ...car,
              销量: parseInt(car.销量, 10)
            }));
            console.log('this.csvData:', this.csvData);
            this.extractMonths();
            this.renderCharts();
          },
          error: (error) => {
            console.error('CSV parse error:', error);
          }
        });
      } catch (error) {
        console.error('Failed to fetch or parse CSV:', error);
      }
    },
    extractMonths() {
      if (!this.csvData || this.csvData.length === 0) {
        console.error('CSV data is empty or invalid');
        return;
      }
      this.months = Array.from(new Set(this.csvData.map(car => car.月份))).sort();
      // 设置默认月份，确保 renderRankings 在初始渲染时可以正确执行
      this.selectedMonth = this.months[0];
      console.log('Extracted months:', this.months);
      console.log('Selected month:', this.selectedMonth);

      // 初始化排行榜
      this.renderRankings();
    },
    renderCharts() {
      if (!this.csvData || this.csvData.length === 0) {
        console.error('CSV data is empty or invalid');
        return;
      }

      this.renderTotalSalesChart();
      this.renderPriceRangeChart();
      this.renderBrandTopChart();
      this.renderTypeMonthlyTrendChart();
      this.renderTypeTotalSalesChart();
      this.renderMPVSeriesSalesChart();
      this.renderSeriesTotalSalesChart();
      this.renderSeriesMonthlyTrendChart();
      this.renderSeriesTopChart();
      
    },
    renderTotalSalesChart() {
      const chart = echarts.init(this.$refs.totalSalesChart);
      const groupedData = this.groupDataByMonth(this.csvData);
      console.log('Grouped data by month:', groupedData);
      const months = Object.keys(groupedData).filter(month => month !== 'undefined');
      const totalSales = months.map(month => groupedData[month].reduce((sum, car) => sum + parseInt(car.销量), 0));
      console.log('Total sales data:', totalSales);

      const option = {
        title: { text: '全国各月总销量' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: months },
        yAxis: { type: 'value' },
        series: [{ data: totalSales, type: 'line' }]
      };

      chart.setOption(option);
    },
    renderPriceRangeChart() {
      const chart = echarts.init(this.$refs.priceRangeChart);
      const priceRanges = ['0-10', '10-20', '20-30', '30-50', '50-100', '100+'];
      const salesByPriceRange = this.csvData.reduce((acc, car) => {
        const price = parseFloat(car['平均价格（万）']);
        if (price <= 10) acc['0-10'] += parseInt(car.销量);
        else if (price <= 20) acc['10-20'] += parseInt(car.销量);
        else if (price <= 30) acc['20-30'] += parseInt(car.销量);
        else if (price <= 50) acc['30-50'] += parseInt(car.销量);
        else if (price <= 100) acc['50-100'] += parseInt(car.销量);
        else if (price > 100) acc['100+'] += parseInt(car.销量);
        return acc;
      }, { '0-10': 0, '10-20': 0, '20-30': 0, '30-50': 0, '50-100': 0, '100+': 0 });
      console.log('Sales by price range:', salesByPriceRange);

      const option = {
        title: { text: '各个价格区间的销量占比' },
        tooltip: { trigger: 'item' },
        series: [
          {
            type: 'pie',
            data: priceRanges.map(range => ({ value: salesByPriceRange[range], name: range })),
            label: {
              normal: {
                show: true,
                position: 'outside',
                formatter: '{b}: {d}%' // {b} 表示数据项名称, {d} 表示百分比
              }
            }
          }
        ]
      };

      chart.setOption(option);
    },
    renderBrandTopChart() {
      const chart = echarts.init(this.$refs.brandTopChart);
      const brandSales = this.csvData.reduce((acc, car) => {
        acc[car.品牌] = (acc[car.品牌] || 0) + parseInt(car.销量);
        return acc;
      }, {});
      const topBrands = Object.entries(brandSales).sort((a, b) => b[1] - a[1]).slice(0, 10);
      console.log('Top brands:', topBrands);

      const option = {
        title: { text: '品牌销量TOP10' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: topBrands.map(item => item[0]) },
        yAxis: { type: 'value' },
        series: [{ data: topBrands.map(item => item[1]), type: 'bar' }]
      };

      chart.setOption(option);
    },
    renderTypeMonthlyTrendChart() {
      const chart = echarts.init(this.$refs.typeMonthlyTrendChart);
      const groupedData = this.groupDataByTypeAndMonth(this.csvData);
      const types = Object.keys(groupedData).filter(month => month !== 'undefined');
      console.log('Grouped data by type and month:', groupedData);

  // 过滤掉 undefined 的月份
      const validMonths = this.months.filter(month => groupedData[types[0]][month] !== undefined); // 以第一个类型的数据为基准检查

      const option = {
        title: { text: '各类车型每月销量趋势' },
        tooltip: { trigger: 'axis' },
        legend: { data: types },
        xAxis: { type: 'category', data: validMonths },
        yAxis: { type: 'value' },
        series: types.map(type => ({
          name: type,
          type: 'line',
          data: validMonths.map(month => groupedData[type][month] || 0)
          }))
      };

      chart.setOption(option);
    },
    renderTypeTotalSalesChart() {
      const chart = echarts.init(this.$refs.typeTotalSalesChart);
      const groupedData = this.groupDataByType(this.csvData);
      console.log('Grouped data by type:', groupedData);
      const types = Object.keys(groupedData);
      const totalSales = types.map(type => groupedData[type].reduce((sum, car) => sum + parseInt(car.销量), 0));
      console.log('Total sales by type:', totalSales);

      const option = {
        title: { text: '各类车型总销量占比' },
        tooltip: { trigger: 'item' },
        series: [
          {
            type: 'pie',
            radius: '50%',
            data: types.map((type, index) => ({ value: totalSales[index], name: type })),
            emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
            label: {
              normal: {
                show: true,
                position: 'outside',
                formatter: '{b}: {d}%' // {b} 表示数据项名称, {d} 表示百分比
              }
            }
          }
        ]
      };

      chart.setOption(option);
    },
    renderSeriesTotalSalesChart() {
      const chart = echarts.init(this.$refs.seriesTotalSalesChart);
      const groupedData = this.groupDataBySeries(this.csvData);
      console.log('Grouped data by series:', groupedData);
      const series = Object.keys(groupedData);
      const totalSales = series.map(ser => groupedData[ser].reduce((sum, car) => sum + parseInt(car.销量), 0));
      console.log('Total sales by series:', totalSales);

      const option = {
        title: { text: '各系列车型总销量占比' },
        tooltip: { trigger: 'item' },
        series: [
          {
            type: 'pie',
            radius: '50%',
            data: series.map((ser, index) => ({ value: totalSales[index], name: ser })),
            emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
            label: {
              normal: {
                show: true,
                position: 'outside',
                formatter: '{b}: {d}%' // {b} 表示数据项名称, {d} 表示百分比
              }
            }
          }
        ]
      };

      chart.setOption(option);
    },
    renderSeriesMonthlyTrendChart() {
      const chart = echarts.init(this.$refs.seriesMonthlyTrendChart);
      const groupedData = this.groupDataBySeriesAndMonth(this.csvData);
      const series = Object.keys(groupedData).filter(ser => ser !== 'undefined');
      console.log('Grouped data by series and month:', groupedData);
  
  // Filter out undefined months
      const validMonths = this.months.filter(month => month !== undefined);
  
      const option = {
        title: { text: '各系车型销量趋势' },
        tooltip: { trigger: 'axis' },
        legend: { data: series },
        xAxis: { type: 'category', data: validMonths },
        yAxis: { type: 'value' },
        series: series.map(ser => ({
          name: ser,
          type: 'line',
          data: validMonths.map(month => groupedData[ser][month] || 0)
         }))
      };
      chart.setOption(option);
    },
    renderSeriesTopChart() {
      const chart = echarts.init(this.$refs.seriesTopChart);
      const seriesSales = this.csvData.reduce((acc, car) => {
        acc[car.车系] = (acc[car.车系] || 0) + parseInt(car.销量);
        return acc;
      }, {});
      const topSeries = Object.entries(seriesSales).sort((a, b) => b[1] - a[1]).slice(0, 10);
      console.log('Top series:', topSeries);

      const option = {
        title: { text: '车系销量TOP' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: topSeries.map(item => item[0]).filter(category => category !== 'undefined') },
        yAxis: { type: 'value' },
        series: [{ data: topSeries.map(item => item[1]), type: 'bar' }]
      };

      chart.setOption(option);
    },
    renderMPVSeriesSalesChart() {
      const chart = echarts.init(this.$refs.mpvSeriesSalesChart);
      const mpvSales = this.csvData.filter(car => car.车型 === 'MPV');
      const mpvSeriesSales = mpvSales.reduce((acc, car) => {
        acc[car.车系] = (acc[car.车系] || 0) + parseInt(car.销量);
        return acc;
      }, {});
      console.log('MPV series sales:', mpvSeriesSales);

      const option = {
        title: { text: 'MPV 各车系销量占比' },
        tooltip: { trigger: 'item' },
        series: [
          {
            type: 'pie',
            radius: '50%',
            data: Object.entries(mpvSeriesSales).map(([series, sales]) => ({ value: sales, name: series })),
            emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
            label: {
              normal: {
                show: true,
                position: 'outside',
                formatter: '{b}: {d}%' // {b} 表示数据项名称, {d} 表示百分比
              }
            }
          }
        ]
      };

      chart.setOption(option);
    },
    renderRankings() {
      this.rankings = this.csvData
        .filter(car => car.月份 === this.selectedMonth)
        .sort((a, b) => parseInt(b.销量) - parseInt(a.销量))
        .map((car, index) => ({
          排名: index + 1,
          汽车图片: car['车辆图片链接'],
          车名: car.车名,
          品牌: car.品牌,
          车型: car.车型,
          车系: car.车系,
          价格: car['平均价格（万）'],
          销量: car.销量
        }));
    },
    groupDataByMonth(data) {
      return data.reduce((acc, car) => {
        const month = car.月份;
        if (!acc[month]) {
          acc[month] = [];
        }
        acc[month].push(car);
        return acc;
      }, {});
    },
    groupDataByTypeAndMonth(data) {
      return data.reduce((acc, car) => {
        const type = car.车型;
        const month = car.月份;
        if (!acc[type]) {
          acc[type] = {};
        }
        if (!acc[type][month]) {
          acc[type][month] = 0;
        }
        acc[type][month] += parseInt(car.销量);
        return acc;
      }, {});
    },
    groupDataByType(data) {
      return data.reduce((acc, car) => {
        const type = car.车型;
        if (!acc[type]) {
          acc[type] = [];
        }
        acc[type].push(car);
        return acc;
      }, {});
    },
    groupDataBySeries(data) {
      return data.reduce((acc, car) => {
        const series = car.车系;
        if (!acc[series]) {
          acc[series] = [];
        }
        acc[series].push(car);
        return acc;
      }, {});
    },
    groupDataBySeriesAndMonth(data) {
      return data.reduce((acc, car) => {
        const series = car.车系;
        const month = car.月份;
        if (!acc[series]) {
          acc[series] = {};
        }
        if (!acc[series][month]) {
          acc[series][month] = 0;
        }
        acc[series][month] += parseInt(car.销量);
        return acc;
      }, {});
    },
    handleMonthChange() {
      this.renderRankings();
    }
  }
};
</script>

<style scoped>
.fixed-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  background-color: #fff;
  z-index: 1000;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
</style>


