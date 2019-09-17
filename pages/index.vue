<template lang="pug">
  .container
    h1 Coins
    table
      thead
        tr
          th &nbsp;
          th Currency
          th Market Cap
          th Price
          th 24h Change
          th 24h Max
          th 24h Min
      tbody
        template(v-for="cur in currenciesSorted")
          template(v-if="cur.RAW")
            tr
              td
                img.coinlogo(:src="'https://www.cryptocompare.com/'+cur.CoinInfo.ImageUrl")
              td.currency
                | {{cur.CoinInfo.FullName}}
                br
                span {{cur.CoinInfo.Name}}
              td.price
                b {{cur.DISPLAY.USD.MKTCAP}}
              td.price {{cur.DISPLAY.USD.PRICE}}
              td.price.updown(:class="{ up: (cur.RAW.USD.CHANGE24HOUR > 0), down: (cur.RAW.USD.CHANGE24HOUR < 0) }") {{cur.DISPLAY.USD.CHANGE24HOUR}}
              td.price {{cur.DISPLAY.USD.HIGH24HOUR}}
              td.price {{cur.DISPLAY.USD.LOW24HOUR}}
    .autoload(ref='autoload', :class="{ loading: loading }")
</template>

<script>
export default {
  components: {},
  data() {
    return {
      page: 0,
      last: false,
      limit: 100,
      timeout: 2000,
      loading: false,
      updating: false,
      mounted: false,
      currencies: []
    }
  },
  computed: {
    currenciesSorted() {
      let d = []
      this.currencies.forEach((i) => {
        d = d.concat(i)
      })
      d = d.filter((el) => {
        return Boolean(el.RAW)
      })
      d.sort(function(a, b) {
        if (!a.RAW) return 0
        if (!b.RAW) return 0
        if (a.RAW.USD.MKTCAP > b.RAW.USD.MKTCAP) return -1
        if (a.RAW.USD.MKTCAP < b.RAW.USD.MKTCAP) return 1
        return 0
      })
      return d
    }
  },
  mounted() {
    this.mounted = true
    this.updateData()
    window.addEventListener('scroll', this.isVisibilityChange)
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.isVisibilityChange)
  },
  methods: {
    async loadData(page, to) {
      const path =
        'https://min-api.cryptocompare.com/data/top/mktcapfull?page=' +
        page +
        '&limit=' +
        this.limit +
        '&tsym=USD'
      const response = await this.$axios.get(path)
      const data = response.data.Data

      this.$set(this.currencies, page, data)
      if (
        this.currencies[this.currencies.length - 1].length < this.limit &&
        !this.last
      )
        this.$set(this, 'last', true)

      if (to > page) this.loadData(page + 1, to)
      else {
        this.updating = false
        this.loading = false
        if (this.mounted)
          setTimeout(
            function(that) {
              that.updateData()
            },
            this.timeout,
            this
          )
      }
    },
    updateData() {
      if (!this.updating) {
        this.updating = true
        this.loadData(0, this.page)
      }
    },
    isElementInViewport(el) {
      const rect = el.getBoundingClientRect()
      return (
        rect.top >= 0 &&
        rect.bottom <=
          (window.innerHeight || document.documentElement.clientHeight)
      )
    },
    isVisibilityChange() {
      const visible = this.isElementInViewport(this.$refs.autoload)
      if (visible && !this.updating) {
        if (!this.last) {
          this.page += 1
          this.$set(this, 'loading', true)
          this.updateData()
        }
      }
    }
  }
}
</script>

<style lang="scss">
table {
  width: 100%;

  thead {
    th {
      background: #ddd;
      padding: 3px;
    }
  }

  tbody {
    tr:nth-child(even) {
      td {
        background: #eee;
      }
    }

    td {
      padding: 3px;

      &.name {
        text-align: right;
        max-width: 30px;
        font-weight: bold;
      }

      &.price {
        text-align: right;
        font-family: monospace;
      }

      &.updown {
        &.up {
          background: rgb(188, 255, 160) !important;
        }
        &.down {
          background: rgb(255, 160, 160) !important;
        }
      }

      &.currency {
        font-size: 1.2em;
        max-width: 150px;

        span {
          opacity: 0.5;
          color: rgb(21, 64, 109);
          font-size: 0.8em;
          font-weight: bold;
        }
      }

      .coinlogo {
        max-width: 20px;
        max-height: 20px;
        margin: 0 auto;
        display: block;
      }
    }
  }
}

.autoload {
  &.updating {
    &::before {
      content: 'Loading...';
    }
  }
}
</style>
