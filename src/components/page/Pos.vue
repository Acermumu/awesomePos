<template>
  <div class="pos">
    <div>
      <el-row >
        <el-col :span='6' class="pos-order" id="order-list">
          <el-tabs>
            <el-tab-pane label="点餐">
              <el-table :data="tableData" border   style="width: 100%" >

                <el-table-column prop="goodsName" label="商品"></el-table-column>
                <el-table-column prop="count" label="数量" width="80"></el-table-column>
                <el-table-column prop="price" label="金额" width="70"></el-table-column>
                <el-table-column  label="操作" width="100" fixed="right">
                  <template scope="scope">

                    <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                    <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>

                  </template>
                </el-table-column>
              </el-table>
              <div class="totalDiv">
                <small>数量:</small>{{totalCount}}&nbsp;&nbsp;&nbsp;&nbsp;
                <small>金额:</small>{{totalMoney}}元
              </div>
              <div class="div-btn">
                <el-button type="warning" >挂单</el-button>
                <el-button type="danger" @click=" delAllGoods()">删除</el-button>
                <el-button type="success" @click=" checkout()">结账</el-button>
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
        <!--商品展示-->
        <el-col :span="18">
          <div class="often-goods">
            <div class="title">常用商品</div>
            <div class="often-goods-list">
              <ul>
                <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                  <span>{{goods.goodsName}}</span>
                  <span class="o-price">￥{{goods.price}}元</span>
                </li>
              </ul>
            </div>
          </div>

          <div class="goods-type">

            <el-tabs>
              <el-tab-pane label="汉堡">
                <ul class='cookList'>
                <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="小食">
                <ul class='cookList'>
                  <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="饮料">
                <ul class='cookList'>
                  <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="套餐">
                <ul class='cookList'>
                  <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </el-tab-pane>

            </el-tabs>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    name: 'Pos',
    /*created(){
      axios.get('')
        .then(response=>{
          console.log(response);
          this.oftenGoods=response.data;
        })
        .catch(error=>{
          console.log(error);
          alert('网络错误，不能访问');
        })


      //读取分类商品列表
      axios.get('')
        .then(response=>{
          console.log(response);
          //this.oftenGoods=response.data;
          this.type0Goods=response.data[0];
          this.type1Goods=response.data[1];
          this.type2Goods=response.data[2];
          this.type3Goods=response.data[3];

        })
        .catch(error=>{
          console.log(error);
          alert('网络错误，不能访问');
        })
    },*/
    mounted:function(){
      var orderHeight=document.body.clientHeight;
      document.getElementById("order-list").style.height=orderHeight+'px';
    },
    methods:{
      //添加订单列表的方法
      addOrderList(goods){
        this.totalCount=0; //汇总数量清0
        this.totalMoney=0;
        let isHave=false;
        //判断是否这个商品已经存在于订单列表
        for (let i=0; i<this.tableData.length;i++){
          console.log(this.tableData[i].goodsId);
          if(this.tableData[i].goodsId==goods.goodsId){
            isHave=true; //存在
          }
        }
        //根据isHave的值判断订单列表中是否已经有此商品
        if(isHave){
          //存在就进行数量添加
          let arr = this.tableData.filter(o =>o.goodsId == goods.goodsId);
          arr[0].count++;
          //console.log(arr);
        }else{
          //不存在就推入数组
          let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
          this.tableData.push(newGoods);

        }

        //进行数量和价格的汇总计算
        this.tableData.forEach((element) => {
          this.totalCount+=element.count;
          this.totalMoney=this.totalMoney+(element.price*element.count);
        });
      },
      //删除单个商品
      delSingleGoods(goods){
        console.log(goods);
        this.tableData=this.tableData.filter(o => o.goodsId !=goods.goodsId);
        this.getAllMoney();
      },
      checkout() {
    if (this.totalCount!=0) {
      this.tableData = [];
      this.totalCount = 0;
      this.totalMoney = 0;
      this.$message({
        message: '结账成功，感谢你又为店里出了一份力!',
        type: 'success'
      });

    }else{
      this.$message.error('不能空结。老板了解你急切的心情！');
    }
  },
      //汇总数量和金额
      getAllMoney(){
        this.totalCount=0;
        this.totalMoney=0;
        if(this.tableData){
          this.tableData.forEach((element) => {
            this.totalCount+=element.count;
            this.totalMoney=this.totalMoney+(element.price*element.count);
          });
        }
      },
      //删除所有商品
      delAllGoods() {
        this.tableData = [];
        this.totalCount = 0;
        this.totalMoney = 0;
      },
    },
    data(){
      return{
        tableData: [],
        oftenGoods:[
          {
            goodsId:1,
            goodsName:'香辣鸡腿堡',
            price:18
          }, {
            goodsId:2,
            goodsName:'田园鸡腿堡',
            price:15
          }, {
            goodsId:3,
            goodsName:'和风汉堡',
            price:15
          }, {
            goodsId:4,
            goodsName:'快乐全家桶',
            price:80
          }, {
            goodsId:5,
            goodsName:'脆皮炸鸡腿',
            price:10
          }, {
            goodsId:6,
            goodsName:'魔法鸡块',
            price:20
          }, {
            goodsId:7,
            goodsName:'可乐大杯',
            price:10
          }, {
            goodsId:8,
            goodsName:'雪顶咖啡',
            price:18
          }, {
            goodsId:9,
            goodsName:'大块鸡米花',
            price:15
          }, {
            goodsId:20,
            goodsName:'香脆鸡柳',
            price:17
          }
        ],
        type0Goods:[
          {
            "goodsId": 1,
            "goodsImg": "https://gss0.baidu.com/9fo3dSag_xI4khGko9WTAnF6hhy/zhidao/pic/item/810a19d8bc3eb135f15862dfaa1ea8d3fd1f4433.jpg",
            "goodsName": "香辣鸡腿堡",
            "price": 18
          },
          {
            "goodsId": 2,
            "goodsImg": "http://img3.imgtn.bdimg.com/it/u=1185948647,811508068&fm=26&gp=0.jpg",
            "goodsName": "田园鸡腿堡",
            "price": 15
          },
          {
            "goodsId": 3,
            "goodsImg": "http://img2.imgtn.bdimg.com/it/u=2347413332,2198921548&fm=26&gp=0.jpg",
            "goodsName": "和风汉堡",
            "price": 15
          },
          {
            "goodsId": 4,
            "goodsImg": "https://gss0.baidu.com/-4o3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=015f3d905dda81cb4eb38bcb6256fc2e/f11f3a292df5e0fe35145c65506034a85edf7242.jpg",
            "goodsName": "新奥尔良烤鸡腿堡",
            "price": 18
          },
          {
            "goodsId": 5,
            "goodsImg": "https://gss0.baidu.com/-fo3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=ad6e7f69f4dcd100cdc9f02742bb6b28/7aec54e736d12f2e2f917b0443c2d5628535683a.jpg",
            "goodsName": "伴鸡伴虾堡",
            "price": 18
          },
          {
            "goodsId": 6,
            "goodsImg": "https://gss0.baidu.com/94o3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=523f59554690f60304e5944109229f23/9e3df8dcd100baa115ac92df4b10b912c9fc2ed6.jpg",
            "goodsName": "BBQ手撕猪肉堡",
            "price": 18
          },
          {
            "goodsId": 7,
            "goodsImg": "https://gss0.baidu.com/94o3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=11c74c1fdd1373f0f56a6799943f67c3/6d81800a19d8bc3e71b1d2ae8e8ba61ea8d3451d.jpg",
            "goodsName": "培根烤鸡腿堡",
            "price": 18
          },
        ],
        type1Goods:[
          {
            "goodsId": 4,
            "goodsImg": "https://f12.baidu.com/it/u=1241728953,644304632&fm=72",
            "goodsName": "大包薯条",
            "price": 18
          },
          {
            "goodsId": 5,
            "goodsImg": "http://img1.imgtn.bdimg.com/it/u=1469451641,12219662&fm=26&gp=0.jpg",
            "goodsName": "脆皮炸鸡腿",
            "price": 20
          },
          {
            "goodsId": 6,
            "goodsImg": "https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=273732511,1691876474&fm=26&gp=0.jpg",
            "goodsName": "魔法鸡块",
            "price": 20
          }
        ],
        type2Goods:[
          {
            "goodsId": 7,
            "goodsImg": "https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=846473507,140762545&fm=26&gp=0.jpg",
            "goodsName": "可乐大杯",
            "price": 10
          },
          {
            "goodsId": 8,
            "goodsImg": "http://img3.imgtn.bdimg.com/it/u=2418208511,1669004968&fm=26&gp=0.jpg",
            "goodsName": "雪顶咖啡",
            "price": 18
          }
        ],
        type3Goods:[
          {
            "goodsId": 9,
            "goodsImg": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=520671982,2791161766&fm=26&gp=0.jpg",
            "goodsName": "儿童欢乐套餐",
            "price": 25
          },
          {
            "goodsId": 10,
            "goodsImg": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=2492510839,35984329&fm=26&gp=0.jpg",
            "goodsName": "快乐全家桶",
            "price": 99
          }
        ],
        totalCount:0,
        totalMoney:0
      }
    },
  }

</script>
<style scoped>
.pos-order{
  background-color: #F9FAFC;
  border-right:1px solid #C0CCDA;
  height:100%;
}
 .div-btn{
   margin-top:10px;
   margin-left:130px;
 }
.title{
  height: 20px;
  border-bottom:1px solid #D3DCE6;
  background-color: #F9FAFC;
  padding:10px;
}
.often-goods-list ul li{
  list-style: none;
  float:left;
  border:1px solid #E5E9F2;
  padding:10px;
  margin:5px;
  background-color:#fff;
}
.o-price{
  color:#58B7FF;
}
  .goods-type{
    clear: both;
  }
.cookList li{
  list-style: none;
  width:23%;
  border:1px solid #E5E9F2;
  height: auto;
  overflow: hidden;
  background-color:#fff;
  padding: 2px;
  float:left;
  margin: 2px;

}
.cookList li span{

  display: block;
  float:left;
}
.foodImg{
  width: 45%;
}
.foodName{
  font-size: 18px;
  padding-left: 20px;
  color:brown;

}
.foodPrice{
  font-size: 16px;
  padding-left: 45px;
  padding-top:10px;
}
  .totalDiv{
    background-color: #fff;
    padding: 10px;
    border-bottom: 1px solid #D3dce6;
  }
</style>
