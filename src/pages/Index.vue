<template>
  <q-page class="flex flex-center">
    <q-table
      title="Result"
      :data="resp"
      :columns="columns"
      row-key="urlName"
    />
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      columns: [
        { name: 'urlName', label: 'Url name', align: 'left', field: row => row.key },
        { name: 'url', label: 'Url', align: 'left', field: row => row.val }
      ],
      resp: [],
      timer: {},
      cId: 0,
      log: {
        id: 0,
        startAt: 0,
        endAt: 0,
        status: 0,
        error: ''
      }
    }
  },
  mounted () {
    // Create WebSQL talbe for logs
    const db = openDatabase('callLogs', '1.0', 'logs of API calling', 2 * 1024 * 1024)
    db.transaction(tx => {
      tx.executeSql('DROP TABLE IF EXISTS LOGS')
      tx.executeSql('CREATE TABLE IF NOT EXISTS LOGS (id unique, startAt, endAt, status, error)')
    })
    // Call API every 5s
    this.timer = window.setInterval(() => {
      this.cId += 1
      this.log.id = this.cId
      this.log.startAt = Date.now()
      this.callGitAPI()
    }, 5000)
  },
  destroyed () {
    clearInterval(this.timer)
  },
  methods: {
    callGitAPI () {
      const db = openDatabase('callLogs', '1.0', 'logs of API calling', 2 * 1024 * 1024)
      this.log.status = 0
      this.log.error = ''
      this.$axios.get('https://api.github.com').then(resp => {
        this.log.endAt = Date.now()
        this.log.status = resp.status
        if (resp.status === 200) {
          // Insert log to WebSQL
          this.addLog(db, this.log)
          // Show result to the index page
          Object.getOwnPropertyNames(resp.data).forEach(key => {
            const obj = { key: key, val: resp.data[key] }
            this.resp.push(obj)
          })
        }
      // eslint-disable-next-line handle-callback-err
      }).catch(err => {
        if (this.log.status !== 200) {
          this.log.endAt = Date.now()
          this.log.status = -1
          this.log.error = err
          // Insert log
          this.addLog(db, this.log)
        }
      })
    },
    addLog (db, obj) {
      // const db = openDatabase('callLogs', '1.0', 'logs of API calling', 2 * 1024 * 1024)
      db.transaction(tx => {
        tx.executeSql('INSERT INTO LOGS (id, startAt, endAt, status, error) VALUES ("' + obj.id + '", "' +
          obj.startAt + '", "' + obj.endAt + '", "' + obj.status + '", "' + obj.error + '")')
      })
    }
  }
}
</script>
