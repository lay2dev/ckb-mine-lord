<template>
  <q-page class="column q-px-md q-pt-xs q-pb-md bg-grey-10 text-white">
    <q-pull-to-refresh @refresh="refresh">
      <div class="q-py-sm">
        <q-banner inline-actions rounded dense class="bg-blue-grey-10 text-white">
          <template v-slot:avatar>
            <q-avatar text-color="orange" font-size="1em" icon="developer_board" />
            <span class="miners vertical-middle q-mx-xs"> EPOCH: {{ epoch.number }}</span>
            <small class="text-warning text-bold vertical-middle q-mx-xs"> [ {{ epochProgress.progress }} / {{ epochProgress.length }} ] </small>
          </template>
          <q-linear-progress class="q-px-md" dark rounded style="height: 10px" :value="epochProgress.value" color="warning" />
        </q-banner>
        <div class="row q-mt-sm justify-between rounded-borders q-pa-sm bg-blue-grey-10 text-white">
          <div><span class="stat-label">ADDR: </span> <span class="stat-value">{{ total }}</span></div>
          <div><span class="stat-label">TBLK: </span> <span class="stat-value">{{ height }}</span></div>
          <div><span class="stat-label">DFCT: </span> <span class="stat-value">{{ numberWithCommas(difficulty) }}</span></div>
          <div><span class="stat-label">HASH: </span> <span class="stat-value">{{ numberWithCommas(hashrate.split('.')[0]) }}K</span></div>
        </div>
      </div>
      <q-tabs narrow-indicator v-model="tab" class="bg-grey-10 text-grey-5" align="justify" bordered dense>
        <q-tab name='rank' class="text-primary" label='Rank' />
        <q-tab name='luck' class="text-amber" label='Luck' />
      </q-tabs>
      <q-separator dark />
      <q-tab-panels v-model="tab" animated swipeable class="bg-grey-10">
        <q-tab-panel name="rank">
          <q-list dark bordered separator>
            <q-item v-for="item in list" :key="item.rank" clickable @click="goto(item.addr)" v-ripple class="row">
              <q-item-section avatar>
                <center class="rank_text"> {{item.rank}} </center>
              </q-item-section>
              <q-item-section>
                <q-item-label class="text-primary nick"> {{item.addr.slice(-3)}} </q-item-label>
                <q-item-label caption class="addr"> {{item.addr.slice(0, 3) + '...' + item.addr.slice(-6)}} </q-item-label>
              </q-item-section>
              <q-item-section side>
                <q-item-label class="balance"> {{item.amount.slice(0, -8)}} <small class="text-grey-7">CKB-TEST</small> </q-item-label>
                <q-item-label class="exp"> {{item.exp.split('.')[0]}} <small class="text-grey-5">CKB-REAL</small> </q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </q-tab-panel>
        <q-tab-panel name="luck" class="bg-deep-blue">
          <q-list dark bordered separator>
            <q-item v-for="item in epochs" :key="item.number" clickable @click="goto(item.startNumber)" v-ripple class="row">
              <q-item-section avatar>
                <center class="rank_text"> {{item.number}} </center>
              </q-item-section>
              <q-item-section>
                <q-item-label class="text-amber nick"> {{item.lucky.slice(-3)}} </q-item-label>
                <q-item-label caption class="addr"> {{item.lucky.slice(0, 3) + '...' + item.lucky.slice(-6)}} </q-item-label>
              </q-item-section>
              <q-item-section side>
                <q-item-label class="row justify-between" style="min-width: 85px"><span>height: </span><span class="text-primary text-bold"> {{item.startNumber}} </span></q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </q-tab-panel>
      </q-tab-panels>
      <center class="footer">
        <span> Statistics on this site is for reference only. </span>
        <p> The official result shall prevail. </p>
      </center>
      <center class="footer"> [ zhixian@yamen.co ] </center>
    </q-pull-to-refresh>
  </q-page>
</template>

<script>
import axios from 'axios'
import { colors } from 'quasar'
const api = 'https://api.yamen.co/'
const nervos = 'https://explorer.nervos.org:30001/api/v1/statistics'
export default {
  name: 'Index',
  data () {
    return {
      total: 0,
      tab: 'rank',
      list: [],
      epochs: [],
      height: '-',
      difficulty: '...',
      hashrate: '...',
      blocktime: '-.0',
      epoch: {}
    }
  },
  methods: {
    refresh: function (done) {
      this.load().then(done)
    },
    load: async function () {
      try {
        const res = await Promise.all([
          axios.get(api),
          axios.get(nervos, {
            data: {},
            headers: {
              'Content-Type': 'application/vnd.api+json',
              'Accept': 'application/vnd.api+json'
            }
          })
          // axios.get(api + 'info/epochs')
        ])
        console.log('res', res)
        const { data: { total, list, epochs, epoch, height } } = res[0]
        this.total = total
        this.list = list
        this.epoch = epoch
        this.height = height
        this.epochs = epochs.reverse()

        const { data: { data: { attributes: {
          current_epoch_difficulty: difficulty,
          hash_rate: hashrate,
          average_block_time: blocktime
        } } } } = res[1]
        this.difficulty = difficulty
        this.hashrate = hashrate
        this.blocktime = blocktime

        // this.epochs = (res[2]).data.reverse()
      } catch (e) {
        console.error(e)
      }
    },
    goto: function (where) {
      let url = 'https://explorer.nervos.org/'
      url = url + (isNaN(where) ? 'address/' : 'block/') + where
      where === 'talk' && (url = 'https://talk.nervos.org/t/ckb-testnet/1869')
      window.location.href = url
    },
    numberWithCommas: function (x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',')
    }
  },
  computed: {
    epochProgress: function () {
      let { startNumber, length } = this.epoch
      let { height } = this

      try {
        startNumber = parseInt(startNumber)
        length = parseInt(length)
        height = parseInt(height)

        const progress = height - startNumber + 1
        let value = (progress / length).toFixed(2)
        value = parseFloat(value)

        colors.setBrand('warning', getColor(value))

        return { length, progress, value }
      } catch (e) {
        console.error(e)
      }

      return 0
    }
  },
  mounted () {
    this.refresh()
  }
}
function getColor (value) { // value from 0 to 1
  const hue = ((1 - value) * 100).toString(10)
  const color = ['hsl(', hue, ',54.8%,50.6%)'].join('')
  return color
}
</script>

<style lang="stylus" scoped>
.miners {
  font-size: 1.2em;
  font-weight: bold;
  color: orange;
}
.stat-label {
  font-size: 0.8em;
  color: grey;
}
.stat-value {
  font-size: 0.8em;
  color: orange;
  font-weight: bold;
}
.rank_text {
  color: wheat;
  font-size: 1.5em;
  min-width: 1.65em;
}
.q-item__section--avatar {
  min-width: inherit;
}
.q-tab-panel {
  padding: 10px 0;
}
.nick {
  font-size: 1.3em;
}
.addr {
  color: darkgrey;
  font-size: 0.7em;
}
.balance
  color $primary
  font-weight bold

.exp
  color $blue-5
  font-weight bold

.footer {
  color: grey;
}
</style>
