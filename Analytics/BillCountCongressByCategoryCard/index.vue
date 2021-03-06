<template>
  <div :class="{ phone: isPhone }" class="analytic-card">
    <h1 class="analytic-card-title">Bill count by category</h1>
    <div class="analytic-card-body">
      <div class="analytic-card-info-block">
        <span class="label">Congress ({{ congressRange[0] }} - {{ congressRange[1] }})</span>
        <Slider
          v-model="congressRange" :step="1" :min="congressMin" :max="congressMax" class="slider" showStops
          range @on-change="updateChart"/>
      </div>
      <div class="chart-container">
        <div v-if="isChartLoading" class="chart-loading-overlay">
          <Spinner/>
        </div>
        <BarChart ref="chart" :class="{ isLoading: isChartLoading }" :chartData="chartData" :options="chartOptions" class="chart"/>
      </div>
    </div>
  </div>

</template>

<script>
import Spinner from '~/components/Spinner'
import CategoryByCongressQuery from '~/apollo/queries/Analytics/CategoryByCongress'

export default {
  components: {
    Spinner
  },
  props: {
    categories: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      isChartLoading: true,
      congressRange: [this.$store.state.currentCongress-1, this.$store.state.currentCongress],
      billIdsByCategory: {},
      chartOptions: {
        layout: {
          padding: {
            left: 0,
            right: 20,
            top: 0,
            bottom: 0
          }
        },
        scales: {
          yAxes: [
            {
              ticks: {
                beginAtZero: true
              },
              gridLines: {
                display: true
              }
            }
          ],
          xAxes: [
            {
              gridLines: {
                display: false
              },
              categoryPercentage: 1.0,
              barPercentage: 0.5
            }
          ]
        },
        legend: {
          display: false
        },
        responsive: true,
        maintainAspectRatio: false
      }
    }
  },
  computed: {
    isPhone () {
      return this.$store.getters.isPhone
    },
    congressMax () {
      return this.$store.state.currentCongress
    },
    congressMin () {
      return this.$store.state.earliestCongress
    },
    congress () {
      let congress = []
      for (var i = this.congressRange[0]; i <= this.congressRange[1]; i++) {
        congress.push(i)
      }
      return congress
    },
    chartData () {
      const labels = this.categories.map(category => category.name.replace(' & ', '&').split(' '))
      const data = this.categories.map(category => this.billIdsByCategory[category.id])
      const dataSet = {
        backgroundColor: 'rgba(0, 51, 78, 0.3)',
        borderColor: 'rgba(0, 51, 78, 1)',
        borderWidth: 1,
        data: data
      }
      return {
        labels,
        datasets: [dataSet]
      }
    }
  },
  mounted () {
    this.updateChart()
  },
  methods: {
    prefetchBillIdsByCategory (categoryId) {
      return this.$apollo.query({
        query: CategoryByCongressQuery,
        variables: { lang: this.locale, congress: this.congress, categories: [categoryId] }
      })
    },
    async updateChart () {
      let self = this
      let bills = {}

      this.isChartLoading = true

      Promise.all(this.categories.map(category => this.prefetchBillIdsByCategory(category.id))).then(function (results) {
        self.categories.forEach((category, index) => {
          bills[category.id] = results[index].data.bills[0].prefetchIds.length
          console.log('000', results[index].data.bills[0].prefetchIds.length)
        })
        console.log('bills', bills)
        self.isChartLoading = false
        self.billIdsByCategory = bills
      })
    }
  }
}
</script>

<style scoped lang="scss">
@import 'assets/css/app';
@import 'assets/css/typography';
@import 'assets/css/colors';

.analytic-card {
  @extend .card;
  margin-bottom: 20px;

  .analytic-card-title {
    @extend .card-title;
  }
}

.analytic-card-info-block {
  @extend .card-info-block;
}

.slider {
  margin: 0 5px;
}

.chart-container {
  position: relative;

  .chart-loading-overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .chart {
    max-height: 360px;
    position: relative;

    &.isLoading {
      opacity: 0.5;
    }
  }
}
</style>

<style lang="scss">
@import 'assets/css/app';
@import 'assets/css/typography';
@import 'assets/css/colors';

.ivu-slider-bar {
  background: $twIndigo;
}

.ivu-slider-button,
.ivu-slider-button-dragging,
.ivu-slider-button:hover {
  border-color: $twIndigo;
}
</style>
