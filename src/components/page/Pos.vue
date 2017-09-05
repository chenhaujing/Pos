<template>
  <div class="pos">
    <el-row>
      <el-col :span='8' class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border show-summary style="width: 100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="70"></el-table-column>
              <el-table-column prop="price" label="金额" width="80"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template scope="scope">
                  <el-button type="text" size="small" @click="deleteGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div>
                <p>
                  <span>数量：{{totalCount}}</span>
                </p>
                <p>
                  <span>总计：{{totalMoney}}</span>
                </p>
            </div>
            <div class="btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="deleteAll">删除</el-button>
              <el-button type="success" @click="checkOut">结账</el-button>
            </div>
          </el-tab-pane>

          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>

      </el-col>
      <el-col :span='16'>
        <div class="common-goods">
          <div class="goods-title">常用商品</div>
          <div class="common-goods-list">
            <ul>
              <li v-for="goods in commonGoods" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">{{goods.price}}</span>
              </li>
            </ul>
          </div>
        </div>

        <!--  -->
        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in goodsType1" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
              </el-tab-pane>
          
            <el-tab-pane label="小食">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in goodsType2" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in goodsType3" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in goodsType4" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            </el-tabs>
        </div>

      </el-col>
    </el-row>

  </div>
</template>

<script>

//引入axios 那个页面需要就在那个页面引入
import axios from 'axios';

export default {
  name: 'Pos',
  data() {
    return {
      msg: 'Welcome to pos system!',
      tableData: [],
      //
      commonGoods: [],
      //
      goodsType1: [],
      goodsType2: [],
      goodsType3: [],
      goodsType4: [],
      totalCount:0,
      totalMoney:0
    }
  },
  created:function(){
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
    .then(response=>{
      console.log(response);
      this.commonGoods = response.data;
    })
    .catch(error=>{
      console.log(error);
      alert('网络错误，不能访问！');
    })

    axios.get('http://jspang.com/DemoApi/typeGoods.php')
    .then(response=>{
      console.log(response);
      this.goodsType1=response.data[0];
      this.goodsType2=response.data[1];
      this.goodsType3=response.data[2];
      this.goodsType4=response.data[3];
    })
    .catch(error=>{
      console.log(error);
       alert('网络错误，不能访问！');
    })
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + 'px';
  },
  methods:{
    //添加商品
    addOrderList(goods){
        this.totalCount = 0;
        this.totalMoney = 0;
        //1.判断商品是否存在于订单列表中
        let isHave = false;
        for(let i=0; i<this.tableData.length; i++){
          if(this.tableData[i].goodsId == goods.goodsId){
            isHave = true;
          }
        }
        //根据结果处理业务逻辑
        if(isHave){
          //存在则数量增加
          let arr  = this.tableData.filter(o => o.goodsId == goods.goodsId);
          arr[0].count++;
          console.log(arr);
        }else{
          //不存在就放入数组
          let newGoods = {goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
          this.tableData.push(newGoods);
        }
        this.getTotalMoney();
    },
    //删除商品
    deleteGoods(goods){
        this.tableData = this.tableData.filter(good => good.goodsId != goods.goodsId);
        this.getTotalMoney();
    },

    //
    deleteAll(){
      this.tableData =[];
      this.totalCount = 0;
      this.totalMoney = 0;
    },
    //模拟结账
    checkOut(){
      if(this.totalCount !=0){
          this.totalCount=[];
          this.totalCount = 0;
          this.totalMoney = 0;
          this.$message({
            message:'结账成功！',
            type:'success'
          });
      }else{
        this.$message.error('没有商品，请先选择商品！');
      }

    },
    //进行数量和价格的汇总计算
    getTotalMoney(){
      this.totalCount = 0;
      this.totalMoney = 0;
      if(this.tableData){
          this.tableData.forEach((element) =>{
            this.totalCount += element.count;
            this.totalMoney = this.totalMoney + (element.price * element.count);
          });
      }
    }

  }
}
</script>

<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #666;
}

.btn {
  margin-top: 10px;
}

.goods-title {
  height: 20px;
  border-bottom: 1px solid #D3DCE6;
  background-color: #F9FAFC;
  padding: 10px;
}

.common-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #E5E9F2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
}

.o-price {
  color: #58B7FF;
}

.goods-type {
  clear: both;
}

.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #E5E9F2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
  cursor: pointer;
}

.cookList li span {
  display: block;
  float: left;
}

.foodImg {
  width: 40%;
}

.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}

.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
</style>
