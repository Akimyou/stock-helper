<template>
  <div class="stock">
    <!-- <h1>{{lodingMsg}}</h1> -->
    <div class="header-line">
      <el-table
        size="mini"
        :data="stocks"
        style="width: 100%"
        :default-sort = "{prop: 'date', order: 'descending'}"
        >
        <el-table-column
          prop="name"
          label="股票">
        </el-table-column>

        <!-- <el-table-column
          prop="code"
          label="代码">
        </el-table-column> -->

        <el-table-column
          prop="curPrice"
          label="最新价"
          width="90"
          sortable>
        </el-table-column>
          
        <el-table-column
          prop="range"
          label="涨跌幅"
          width="90"      
          sortable>
        </el-table-column>        
        
        <el-table-column
          prop="rangePrice"
          label="涨跌额"
          width="90"      
          sortable>
        </el-table-column>
          
        <el-table-column
          label="操作">
          <template slot-scope="scope">
            <el-button
              @click.native.prevent="deleteRow(scope.$index, stocks)"
              type="text"
              width="45"
              size="mini">
              移除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="input">
      <el-row :gutter="20">
        <el-col :span="16" :offset="6">
          <el-input size="medium" v-model="input" placeholder="请输入股票代码">
            <el-button @click.native.prevent="addStock" type="number" slot="append" icon="el-icon-circle-plus" clearable >添加</el-button>
          </el-input>
        </el-col>
      </el-row>
    </div>
  </div>    
</template>
<script>
import {getStockByCode} from './api/api'
export default {
  data () {
    return {
      lodingMsg: 'loading...',
      stocks: [],
      stockCodeList: [],
      input: '',
    }
  },
  created(){
    const conShock = 'sz002183'
    this.stockCodeList = (localStorage.stocks && localStorage.stocks.split(',')) || [conShock]
    setTimeout( ()=> {
      this._getALLStock(this.stockCodeList)
    }, 20*3)

    setInterval( () => {
      this._getALLStock(this.stockCodeList)
    }, 10000)
  },
  watch: {
    stockCodeList: function() {
      localStorage.stocks = this.stockCodeList
      console.log('refresh stockCodeList', this.stocks)
    },
    stocks: function() {
      console.log('refresh stocks', this.stocks)
    }
  },
  methods: {
    deleteRow(index, rows) {
      var that = rows
      this.stockCodeList.remove(that[index].code);
      rows.splice(index, 1);
    },
    addStock() {
      console.log(this.input)
      this._getStockByCode(this.input)
    },
    _getALLStock(allStock) {
      for (let i =0; i < allStock.length; i++) {
        this._getStockByCode(allStock[i])
      }
    },
    _getStockByCode(code){
      getStockByCode(code).then(res => {
        var result = res.split('=')[1]
        if (!result) {
          console.log('no result')
          return
        }
        var itemArr = result.split('"')[1].split(',')
        var name = itemArr[0],
            toPrice = Number(itemArr[1]).toFixed(2), // 今开
            yesPrice = Number(itemArr[2]).toFixed(2), // 昨收
            curPrice = Number(itemArr[3]).toFixed(2), // 当前价
            highPrice = Number(itemArr[4]).toFixed(2), // 最高
            // lowPrice = Number(itemArr[5]).toFixed(2), // 未知
            // lowPrice = Number(itemArr[6]).toFixed(2), // 未知
            lowPrice = Number(itemArr[7]).toFixed(2), // 最低
            date = Number(itemArr[8]).toFixed(2), // 日期
            time = Number(itemArr[9]).toFixed(2) // 时间
        var rangePrice = (curPrice - yesPrice).toFixed(2)
        var range = ((curPrice - yesPrice)/yesPrice * 100 ).toFixed(2) + '%'
        var stockObj = {
          code,
          name,
          toPrice,
          yesPrice,
          curPrice: curPrice,
          highPrice,
          lowPrice,
          rangePrice,
          range,
          date,
          time
        }
        
        var indexCode  = this.stockCodeList.indexOf(code) 
        if (indexCode == -1) {
          this.input = ''
          this.stocks.push(stockObj)
          this.stockCodeList.push(code)
        } else {
          this.stocks.splice(indexCode, 1, stockObj)
          this.stockCodeList.splice(indexCode, 1, code)
        }
        console.log('1',this.stocks)
        console.log('2',this.stockCodeList)
      })
    }
  }
}
</script>

<style lang="stylus">
* {
  margin 0
  padding 0
  text-align center   
}
html {
  font-size 20px
}

.stock
  width 22rem
  height 100%
  position relative
  // background-color #eee

.input
  padding 1.5rem 0 0
</style>
