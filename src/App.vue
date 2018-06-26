<template>
  <div id="app">
    <header>海公公的表格数据</header>
    <section>
      <div v-if="show">
        <td-table :sources="data" v-on:change="pagechange"></td-table>
      </div>
    </section>
    <p>
    </p>
  </div>
</template>

<script>
import tdtable from './components/_table'
import json from './components/JSON'
export default {
  name: 'App',
  data: function () {
    return {
      src_js: json,
      data: {
        list: {},
        num: []
      },
      page: 1,
      pagesize: 10,
      show: false
    }
  },
  computed: {
    URL: function () {
      return 'http://139.196.86.8/sso/userManager/getUserlist2.do?page=' + this.page + '&pagesize=' + this.pagesize
    }
  },
  created: function () {
    fetch(this.URL).then((response) => {
      response.json().then(text => {
        this.data = text.data
        this.show = true
      })
    })
  },
  watch: {
    URL: function () {
      this.getData()
    }
  },
  components: {
    'td-table': tdtable
  },
  methods: {
    pagechange: function (currentpage,pagesize) {
      this.page = currentpage
      this.pagesize=pagesize
    },
    getData: function () {
      fetch(this.URL).then((response) => {
        response.json().then(text => {
          this.data = text.data
        })
      })
    }
  }
}

</script>

<style scoped>
  header{
    text-align: center;
    color:green;
    font-size: 3em;
    margin: 5px 0;
  }
  section{
    width: 70%;
    margin: 0 auto;
  }
</style>
