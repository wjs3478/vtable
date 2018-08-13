<template>
<!-- <div  class="row"> -->
    <div id="vtb" class="col-lg-12">
      <div class="panel panel-default">
        <div class="panel-heading">
          <form id="search">
            Search <input name="query" v-model="filterKey" v-on:keyup="pageInit">
          </form>
        </div>
        <div class="panel-body">
          <table class="table table-striped table-bordered table-hover">
            <thead>
              <tr>
                <th v-for="(key,index) in columns" @click="sortBy(key)" :class="{ active: sortKey == key }">
                  {{ key | capitalize }}
                  <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'"></span>
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(entry,index) in filteredData">
                  <td v-for="(key,index) in columns">{{entry[key]}}</td>
              </tr>
            </tbody>
          </table>
            <div class="row">
              <div class="col-md-6">
                <span>총 건수 : {{total}}</span>
              </div>
              <div class="col-md-6">
                <div class="dataTables_paginate paging_simple_numbers" style="margin:0;white-space:nowrap;text-align:right">
                  <ul class="pagination" style="margin:0;white-space:nowrap">
                    <li class="paginate_button previous" v-bind:class="{disabled : isFirstPage}" tabindex="0" role="button" id="dataTables-example_previous" v-on:click="prevPage">
                      <a>Previous</a>
                    </li>
                    <template v-for="ih in toRange(this.minPage,this.maxPage)">
                    <li class="paginate_button" v-bind:class="{active : selectPage==ih}" role="button" tabindex="0" v-on:click="select">
                      <a>{{ih}}</a>
                    </li>
                    </template>
                    <li class="paginate_button next" v-bind:class="{disabled : isLastPage}" role="button" tabindex="0" id="dataTables-example_next" v-on:click="nextPage">
                      <a>Next</a>
                    </li>
                  </ul>
                </div>
              </div>
            </div>
        </div>        
      </div>
    </div>
<!-- </div> -->
</template>

<script>
export default {
//step 페이지네이션에서 한번에 보일 단계
//page는 페이지에서 보일 row 갯수
    props: {
      data: Array,
      columns: Array,
      perPage:Number,
      perStep:Number
    },

    data: function () {
    var sortOrders = {}
    this.columns.forEach(function (key) {
      sortOrders[key] = 1
    })
    return {
      sortKey: '',
      sortOrders: sortOrders,
      filterKey:'',
      total:0,
      totalpage:0,
      
      curPage:1,
      minPage:0,
      maxPage:0,
      selectPage:1
    }
  },
  computed: {
    filteredData: function () {
      var sortKey = this.sortKey
      var filterKey = this.filterKey && this.filterKey.toLowerCase()
      var order = this.sortOrders[sortKey] || 1
      var data = this.data
      if (filterKey) {
        
        data = data.filter(function (row) {
          return Object.keys(row).some(function (key) {
            return String(row[key]).toLowerCase().indexOf(filterKey) > -1
          })
        })
      }
      if (sortKey) {
        data = data.slice().sort(function (a, b) {
          a = a[sortKey]
          b = b[sortKey]
          return (a === b ? 0 : a > b ? 1 : -1) * order
        })
      }

      //총 데이터 갯수
      this.total=data.length;
      if(this.total%this.perPage!=0){
      //총 페이지 갯수
        this.totalpage=Math.trunc(this.total/this.perPage)+1
      }else{
        this.totalpage=this.total/this.perPage
      }
      //한번에 보일 페이지네이션 최대값
      //this.maxPage=this.paging*this.curPage
      this.maxPage=this.perStep*this.curPage
      
      if(this.totalpage<this.maxPage)
        this.maxPage=this.totalpage

      //한번에 보일 페이지네이션 최소값
      //this.minPage=(this.curPage-1)*this.paging+1
      this.minPage=(this.curPage-1)*this.perStep+1

      //page1:1~5
      //page2:6~10
      
      let rowMax=this.selectPage*this.perPage
      let rowMin=rowMax-this.perPage

      return data.slice(rowMin,rowMax)
    },

    isFirstPage(){
      if(this.curPage==1)
        return true
    },
    isLastPage(){
      
      let last=0;
      if(this.totalpage%this.perStep!=0)
      {
        last=Math.trunc(this.totalpage/this.perStep)+1;
      }else{
        last=Math.trunc(this.totalpage/this.perStep);
      }

      if(this.curPage==last)
        return true
    }
  },
  filters: {
    capitalize: function (str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }
  },
  methods: {
    sortBy: function (key) {
      this.sortKey = key
      this.sortOrders[key] = this.sortOrders[key] * -1
    },
    nextPage(event){
      if(this.isLastPage)
        event.preventdefault();

      ++this.curPage
    },
    prevPage(event){
      if(this.isFirstPage)
        event.preventdefault();

      --this.curPage
    },
    toRange(start,end)
    {
      let rng=[]
      for(let j=start;j<=end;j++)
      {
        rng.push(j)
      }
      return rng
    },
    select(event){
      this.selectPage=event.target.text
    },
    pageInit(){
        this.selectPage=1
        this.curPage=1
    }

  }
}


</script>

<style>

</style>
