<template>
  <q-page class="column q-px-md q-py-xs bg-grey-10 text-white">
    <q-pull-to-refresh @refresh="refresh">
      <div class="q-py-sm">
        <q-banner inline-actions rounded dense class="bg-blue-grey-10 text-white">
          <template v-slot:avatar>
            <q-avatar text-color="orange" font-size="1em" icon="developer_board" />
            <span class="text-orange q-ml-xs">Miners:</span>
          </template>
          <span class="miners vertical-middle">{{ total }}</span>
          <template v-slot:action>
            <q-btn flat label="Join them" color="orange" @click="goto('talk')" />
          </template>
        </q-banner>
        <div class="row q-mt-sm justify-between rounded-borders q-pa-sm bg-blue-grey-10 text-white">
          <div><span class="stat-label">TBLK: </span> <span class="stat-value">{{ height }}</span></div>
          <div><span class="stat-label">DFCT: </span> <span class="stat-value">{{ numberWithCommas(difficulty) }}</span></div>
          <div><span class="stat-label">HASH: </span> <span class="stat-value">{{ hashrate.split('.')[0] }} TPS</span></div>
        </div>
      </div>
      <q-list dark bordered separator>
        <q-item v-for="item in list" :key="item.rank" clickable @click="goto(item.addr)" v-ripple class="row">
          <q-item-section avatar>
            <center class="rank_text"> {{item.rank}} </center>
          </q-item-section>
          <q-item-section>
            <q-item-label class="text-primary nick" :class="item.rank < 4 && 'text-orange'"> {{item.addr.slice(-3)}} </q-item-label>
            <q-item-label caption class="addr"> {{item.addr.slice(0, 8) + '...' + item.addr.slice(-8)}} </q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-item-label class="text-primary balance"> {{item.amount.slice(0, -8)}} <small class="text-grey">CKB</small> </q-item-label>
            <q-item-label class="blocks"> {{item.blocks}} <small class="text-grey">BLK</small> </q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
      <center class="footer"> [ zhixian@yamen.co ] </center>
    </q-pull-to-refresh>
  </q-page>
</template>

<style>
.miners {
  font-size: 1.5em;
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
.nick {
  font-size: 1.3em;
}
.addr {
  color: darkgrey;
  font-size: 0.7em;
}
.balance {
  font-weight: bold;
}
.blocks {
  color: skyblue;
  font-weight: bold;
}
.footer {
  color: grey;
  padding: 1em;
}
</style>

<script>
import axios from 'axios'
const api = 'https://api.yamen.co/'
const nervos = 'https://explorer.nervos.org:30001/api/v1/statistics'
export default {
  name: 'Index',
  data () {
    return {
      total: 0,
      list: [],
      height: 0,
      difficulty: 0,
      hashrate: 0,
      blocktime: 0
    }
  },
  methods: {
    refresh: function (done) {
      this.load().then(done)
    },
    load: async function () {
      const { data: { total, list } } = await axios.get(api)
      this.total = total
      this.list = list
      const { data: { data: { attributes: {
        current_epoch_difficulty: difficulty,
        hash_rate: hashrate,
        tip_block_number: height,
        average_block_time: blocktime
      } } } } = await axios.get(nervos, {
        data: {},
        headers: {
          'Content-Type': 'application/vnd.api+json',
          'Accept': 'application/vnd.api+json'
        }
      })
      this.difficulty = difficulty
      this.height = height
      this.hashrate = hashrate
      this.blocktime = blocktime
    },
    goto: function (where) {
      let url = 'https://explorer.nervos.org/address/' + where
      where === 'talk' && (url = 'https://talk.nervos.org/t/ckb-testnet/1869')
      window.location.href = url
    },
    numberWithCommas: function (x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',')
    }
  },
  created () {
    this.$q.addressbarColor.set('#000000')
  },
  mounted () {
    this.refresh()
  }
}
</script>
