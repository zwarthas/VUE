<template>
  <div id="app">
  <table>
    <caption>这是标题</caption>
    <thead>
    <tr>
      <th></th>
      <th scope="col" v-for="(key,index) in keys" :key="index">{{key}}</th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="(lis,index) in sources.list" :key="index">
      <td class="radio">
        <input type="radio" :value="lis" v-model="selectedlis">
      </td>
      <th scope="row">{{lis[keys[0]]}}</th>
      <template v-for="key in key2" v-if="Object.keys(lis).indexOf(key)>-1">
        <template v-if="key==='sex'">
          <td v-if="lis[key]===1||lis[key]===0">
            {{sex[lis[key]]}}
          </td>
          <td v-else>
            {{""}}
          </td>
        </template>
        <td v-else>
          {{lis[key]}}
        </td>
      </template>
      <td v-else>
        {{""}}
      </td>
    </tr>
    <template v-if="pagesize>rowcount">
      <tr v-for="i in pagesize-rowcount" :key="rowcount+i">
        <td v-for="i in 11"></td>
      </tr>
    </template>
    </tbody>
    <tfoot>
    <tr>
      <td class="bottomleft" colspan="5">
        <span >显示第  {{startrecord}}  到第  {{endrecord}}  条记录，总共  {{num}}  条记录，每页显示
          <select v-model="pagesize">
            <option v-for="i in 15" :value="i+5" :key="i">{{i+5}}</option>
          </select>
          条记录</span>
      </td>
      <td class="bottomright" colspan="6">
        <span class="firstpage" v-on:click="tofirstpage">首页</span>
        <span class="last" v-on:click="lastpage">上一页</span>
        <span class="page" v-for="(i,index) in 10" v-if="startpage+i<=maxpage" v-on:click="clicknum(i)" :key="index">
       {{startpage+i}}
        </span>
        <span class="next" v-on:click="nextpage">下一页</span>
        <span class="lastpage" v-on:click="tolastpage">尾页</span>
        <span>，</span>
        <label class="jump">跳到<input class="inputjump" v-model="jumppage">页</label>
        <button class="btnjump" v-on:click="jumpto">确定</button>
      </td>
    </tr>
    </tfoot>
  </table>
    <p>
      {{selectedlis}}
    </p>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: function () {
    return {
      stindex: 0,
      pacentage: ['7%', '13.2%', '12.8%', '20%', '8%', '8%', '4%', '10%', '7%', '10%'],
      /* currentindex指的是当前页码span，从1开始 */
      currentindex: 1,
      /* startpage指的是最左边span的数值-1，从0开始  */
      startpage: 0,
      jumppage: '',
      keys: {},
      key2: {},
      sex: ['男', '女'],
      pagesize: 10,
      selectedlis:{}
    }
  },
  props: {
    sources: {
      default: []
    }
  },
  mounted: function () {
    this.setstyle(1)
  },
  created: function () {
    var lis = this.sources.list[0]
    this.keys = Object.keys(lis)
    var lis2 = Object.keys(this.sources.list[0])
    lis2.splice(0, 1)
    this.key2 = lis2
  },
  watch: {
    currentpage: function (newval, oldval) {
      this.$emit('change', newval, this.pagesize)
    },
    pagesize: function (newval, oldval) {
      this.startpage = 0
      this.currentindex = 1
      this.$emit('change', this.currentpage, newval)
    }
  },
  methods: {
    jumpto: function () {
      if (this.jumppage) {
        if (this.jumppage > 0 && this.jumppage <= this.maxpage) {
          var ob = this.jumppage % this.pagesize
          if (ob === 0) {
            this.startpage = this.jumppage - this.pagesize;
            this.currentindex = this.pagesize
            this.$nextTick(function () {
              this.setstyle(this.currentindex)
            })
          }
          else {
            this.startpage = this.jumppage - ob
            this.currentindex = ob
            this.$nextTick(function () {
              this.setstyle(this.currentindex)
            })
          }
          this.jumppage = ''
        }
      }
    },
    tofirstpage: function () {
      this.startpage = 0
      this.currentindex = 1
      this.setstyle(1)
    },
    tolastpage: function () {
      var ob = Math.ceil(this.maxpage / this.pagesize) - 1
      this.startpage = ob * this.pagesize
      var yu = this.maxpage % this.pagesize
      if (yu === 0) this.currentindex = this.pagesize
      else this.currentindex = yu
      this.setstyle(this.currentindex)
    },
    nextpage: function () {
      if (this.currentpage < this.maxpage) {
        if (this.currentindex === 10) {
          this.startpage += 10
          this.currentindex = 1
        }
        else {
          this.currentindex += 1
        }
        this.setstyle(this.currentindex)
      }
    },
    lastpage: function () {
      if (this.currentpage > 1) {
        if (this.currentindex === 1) {
          this.currentindex = 10
          this.startpage -= 10
        }
        else {
          this.currentindex -= 1
        }
        this.$nextTick(function () {
          this.setstyle(this.currentindex)
        })
      }
    },
    clicknum: function (i) {
      this.currentindex = i
      this.setstyle(i)
    },
    setstyle: function (index) {
      var doms = document.querySelectorAll('td span.page,td span.page-active')
      for (let dom of doms) {
        dom.setAttribute('class', 'page')
      }
      doms[index - 1].setAttribute('class', 'page-active')
    }
  },
  computed: {
    /*
    keys: function () {
      var lis = this.sources.list[0]
      return Object.keys(lis)
    },
    key2: function () {
      var lis2 = Object.keys(this.sources.list[0])
      lis2.splice(0, 1)
      return lis2
    },
    */
    rowcount:function(){
      return this.sources.list.length
    },
    startrecord: function () {
      return parseInt((this.currentpage - 1) * this.pagesize + 1)
    },
    endrecord: function () {
      return parseInt((this.currentpage - 1) * this.pagesize + parseInt(this.pagesize))
    },
    maxpage: function () {
      return Math.ceil(this.sources.num / this.pagesize)
    },
    num: function () {
      return this.sources.num
    },
    /* 当startpage和currentindex改变的时候，currentpage也改变，触发watch，重新加载表格 */
    currentpage: function () {
      return this.startpage + this.currentindex
    }
  }
}

</script>

<style scoped>
  * {
    box-sizing: border-box;
  }

  table {
    table-layout: fixed;
    width: 100%;
    margin: 0 auto;
    border: 3px black solid;
    border-collapse: collapse;
  }

  thead th {
    background: rgb(131, 175, 155);
  }

  tbody tr:nth-child(odd) {
    background-color: rgb(249, 205, 173);
  }

  tbody tr:nth-child(even) {
    background-color: rgb(200, 200, 169);
  }

  tbody th, tbody td {
    border: 1px solid gray;
    text-align: center;
  }

  tbody th:hover, tbody td:hover {
    background-color: rgba(254, 67, 101, 0.5);
    box-shadow: inset 2px 2px 2px rgb(254, 67, 101),
    inset -2px -2px 2px rgb(254, 67, 101);
  }

  tbody tr:focus {
    background-color: rgba(254, 67, 101, 0.2);
  }

  tr {
    height: 55px;
  }

  th, td {
    padding: 5px;
  }

  caption {
    caption-side: bottom;
    text-align: right;
    padding: 10px;
    font-style: italic;
  }

  tfoot {
    background-color: rgb(131, 175, 155);
  }

  td.bottomleft {
    text-align: left;
    margin-left: 3px;
  }

  td.bottomright {
    text-align: right;
    margin-right: 3px;
  }

  tfoot tr td {
    font-size: 0.7em;
    color: rgb(254, 67, 101);
    border-bottom: 1px solid;
    margin: 0 10px;
  }

  td.bottomright span {
    cursor: pointer;
  }

  td.bottomright span.page {
    margin: 0 3px;
    font-style: italic;
  }

  td.bottomright span.page-active {
    margin: 0 3px;
    font-style: italic;
    background-image: linear-gradient(to bottom, rgba(255, 255, 255, 0.4), rgba(0, 0, 0, 0.1) 50%, rgba(255, 255, 255, 0.4));
    outline: 0.1px solid rgb(254, 67, 101);
  }

  td.bottomright span:hover {
    outline: 0.1px solid rgba(0, 0, 0, 0.8);
    background-image: linear-gradient(to bottom, rgba(255, 255, 255, 0.7), rgba(0, 0, 0, 0.2) 50%, rgba(255, 255, 255, 0.7));
  }

  tfoot tr input {
    width: 30px;
    margin: 0 5px;
  }

  /* ['7%', '13.2%', '12.8%', '20%', '8%', '8%', '4%', '10%', '7%', '10%'] */
  thead tr th:nth-child(2) {
    width: 6.5%;
  }

  thead tr th:nth-child(3) {
    width: 12.7%;
  }

  thead tr th:nth-child(4) {
    width: 12.3%;
  }

  thead tr th:nth-child(5) {
    width: 19.5%;
  }

  thead tr th:nth-child(6) {
    width: 7.5%;
  }

  thead tr th:nth-child(7) {
    width: 7.5%;
  }

  thead tr th:nth-child(8) {
    width: 3.5%;
  }

  thead tr th:nth-child(9) {
    width: 9.5%;
  }

  thead tr th:nth-child(10) {
    width: 6.5%;
  }

  thead tr th:nth-child(11) {
    width: 9.5%;
  }

  thead tr th:nth-child(1) {
    width: 3%;
  }

  input.liebiao {
    margin: 0 2px;
    width: 40px;
  }

  span.bottomleft {
    text-align: left;
    margin: 0 2px;
  }

  td.radio{
    text-align: center;
  }
</style>
