<template>
  <q-page class="flex flex-center">
    <q-table
      title="Logs"
      :data="data"
      :columns="columns"
      row-key="id"
    />
    <!-- <table>
      <tr v-for="item in data" :key="item.id">
        <td>{{getTime(item.startAt)}}</td>
        <td>{{getCallTime(item.startAt, item.endAt)}}</td>
        <td>{{isSuccess(item.status)}}</td>
        <td>{{getError(item.status, item.error)}}</td>
      </tr>
    </table> -->
  </q-page>
</template>
<script>
export default {
  name: 'Logs',
  data () {
    return {
      columns: [
        { name: 'id', label: 'ID', align: 'left', field: row => row.id },
        { name: 'date', label: 'Date', align: 'left', field: row => row.date },
        { name: 'usedTime', label: 'Used Time', align: 'left', field: row => row.usedTime },
        { name: 'isSuccess', label: 'Is successed', align: 'left', field: row => row.isSuccess },
        { name: 'errorInfo', label: 'error Info', align: 'left', field: row => row.errorInfo }
      ],
      data: []
    }
  },
  mounted () {
    const db = openDatabase('callLogs', '1.0', 'logs of API calling', 2 * 1024 * 1024)
    let obj = {}
    db.transaction(tx => {
      tx.executeSql('SELECT * FROM LOGS', [], (tx, res) => {
        for (let i = 0; i < res.rows.length; i++) {
          const rr = res.rows[i]
          obj.id = rr.id
          obj.date = this.getTime(rr.startAt)
          obj.usedTime = this.getCallTime(rr.startAt, rr.endAt)
          obj.isSuccess = this.isSuccess(rr.status)
          obj.errorInfo = this.getError(rr.status, rr.error)
          this.data.push(obj)
          obj = {}
        }
      })
    })
  },
  methods: {
    getTime (str) {
      const dStr = new Date(parseInt(str))
      const y = dStr.getFullYear()
      const m = dStr.getMonth()
      const d = dStr.getDay()
      const h = dStr.getHours()
      const m1 = dStr.getMinutes()
      const s = dStr.getSeconds()
      return y + '/' + m + '/' + d + ' ' + h + ':' + m1 + ':' + s
    },
    getCallTime (s, e) {
      return parseInt(e) - parseInt(s) + 'ms'
    },
    isSuccess (code) {
      return code === '200' ? 'Success' : 'Failed'
    },
    getError (code, error) {
      return code === '200' ? 'none' : error
    }
  }
}
</script>
