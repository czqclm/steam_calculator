<template>
  <div>
    <el-form label-width="150px" :inline="true">
      <el-form-item label="名称">
        <el-input v-model="name"></el-input>
      </el-form-item>
      <el-form-item label="个数">
        <el-input v-model="quantity"></el-input>
      </el-form-item>
      <el-form-item label="平台价格(成本) / 个">
        <el-input v-model="platformPrice"></el-input>
      </el-form-item>
      <el-form-item label="steam 卖 / 个">
        <el-input v-model="sell"></el-input>
      </el-form-item>
      <el-form-item label="stean 税率">
        <el-input v-model="taxRate"></el-input>
      </el-form-item>
      <el-form-item label="">
        <el-button type="success" @click="calculate">计算</el-button>
      </el-form-item>
      <el-form-item label="">
        <el-button type="primary" @click="save">保存</el-button>
      </el-form-item>
      <el-form-item label="">
        <el-button type="primary" @click="saveNew">另存为</el-button>
      </el-form-item>
    </el-form>

    <div>
      <el-table
          :border="true"
          :data="this.filters(tableData)"
          style="margin: 0 auto; width: 1200px"
          empty-text="-"
          :row-class-name="tableRowClassName"
      >
        <el-table-column prop="incomeRate" label="折扣率" width="180">
        </el-table-column>
        <el-table-column prop="income" label="收益" width="180">
        </el-table-column>
        <el-table-column prop="buyerPriceByOne" label="买方出价 / 个" width="180">
        </el-table-column>
        <el-table-column prop="sellerPriceByOne" label="卖方实际得到 / 个" width="180">
        </el-table-column>
        <el-table-column prop="buyerPrice" label="买方出价" width="180">
        </el-table-column>
        <el-table-column prop="sellerPrice" label="卖方实际得到" width="180">
        </el-table-column>
      </el-table>
    </div>

    <div>
      <p>历史</p>
      <el-table :data="list" stripe style="margin: 0 auto; width: 1200px">
        <el-table-column prop="name" label="商品" width="180">
        </el-table-column>
        <el-table-column prop="quantity" label="个数" width="100">
        </el-table-column>
        <el-table-column
            prop="platformPrice"
            label="平台价格(成本)"
            width="180"
        >
        </el-table-column>
        <el-table-column prop="sell" label="steam 卖" width="180">
        </el-table-column>
        <el-table-column prop="taxRate" label="stean 税率" width="180">
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope" width="">
            <el-button size="mini" @click="handleApply(scope.$index, scope.row)"
            >应用
            </el-button
            >
            <el-button
                size="mini"
                type="danger"
                @click="handleDelete(scope.$index)"
            >删除
            </el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import {v4 as uuidv4} from "uuid";

export default {
  data() {
    return {
      id: "",
      quantity: 1,
      platformPrice: 0,
      sell: 0,
      taxRate: 13,
      name: "",
      tableData: [],
      list: [],
    };
  },
  mounted() {
    this.getList();
  },
  methods: {
    calculate() {
      this.tableData = [];
      if (this.sell !== 0) {
        var item = {};
        // 卖方实际所得
        item.sellerPrice = this.sell * (1 - this.taxRate * 0.01) * this.quantity;
        item.sellerPriceByOne = this.sell * (1 - this.taxRate * 0.01)
        // 买方出价
        item.buyerPrice = this.sell * this.quantity;
        item.buyerPriceByOne = this.sell
        // 收益
        item.income = item.sellerPrice - this.platformPrice * this.quantity;
        // 收益率
        item.incomeRate = this.platformPrice * this.quantity / item.sellerPrice;
        this.tableData.push(item);

        // 建议售价
        // 0.65
        this.tableData.push(this.discountCalculate(0.65, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.70, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.75, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.80, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.85, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.90, this.quantity, this.platformPrice, this.taxRate));
        this.tableData.push(this.discountCalculate(0.95, this.quantity, this.platformPrice, this.taxRate));
      }
    },
    /**
     * 计算推荐
     * @param expectedDiscountRate 推荐比例
     * @param quantity 数量
     * @param platformPrice 平台单价
     * @param taxRate 税率
     */
    discountCalculate(expectedDiscountRate, quantity, platformPrice, taxRate) {
      let item = {};
      item.incomeRate = 0
      item.income = 0
      item.buyerPriceByOne = 0
      item.buyerPrice = 0
      item.sellerPriceByOne = 0
      item.sellerPrice = 0
      console.log(taxRate * 1)
      let expect = platformPrice * quantity / expectedDiscountRate
      console.log("期望得到", expect)
      item.incomeRate = expectedDiscountRate
      item.buyerPrice = expect * (1 + (taxRate / 100))
      item.buyerPriceByOne = item.buyerPrice / quantity
      item.income = expect - platformPrice
      item.sellerPrice = platformPrice * quantity / expectedDiscountRate
      item.sellerPriceByOne = platformPrice * quantity / expectedDiscountRate/ quantity
      return item;
    },
    save() {
      if (this.id) {
        for (let i = 0; i < this.list.length; i++) {
          if (this.list[i].id === this.id) {
            this.list[i] = {
              id: this.id,
              quantity: this.quantity,
              name: this.name,
              platformPrice: this.platformPrice,
              sell: this.sell,
              taxRate: this.taxRate,
            }
          }
        }
      } else {
        var json = {
          id: uuidv4(),
          quantity: this.quantity,
          name: this.name,
          platformPrice: this.platformPrice,
          sell: this.sell,
          taxRate: this.taxRate,
        };
        this.list.push(json);
      }
      this.$message({
        message: '保存成功',
        type: 'success'
      });
      localStorage.setItem("list", JSON.stringify(this.list));
      this.getList()
    },
    saveNew() {
      this.id = "";
      this.save();
      this.platformPrice = 0;
      this.quantity = 1;
      this.sell = 0;
      this.taxRate = 13;
      this.name = "";
    },
    getList() {
      if (localStorage.getItem("list")) {
        var jsonList = JSON.parse(localStorage.getItem("list"));
        this.list = jsonList;
      } else {
        this.list = [];
      }
    },
    handleApply(index, row) {
      this.id = row.id;
      this.name = row.name;
      this.quantity = row.quantity;
      this.platformPrice = row.platformPrice;
      this.sell = row.sell;
      this.taxRate = row.taxRate;
      this.calculate()
      this.$message({
        message: '应用成功',
        type: 'success'
      });
    },
    handleDelete(index) {
      this.list.splice(index, 1)
      localStorage.setItem("list", JSON.stringify(this.list));
      this.getList()
      this.$message({
        message: '删除成功',
        type: 'success'
      });
    },
    filters(dataList) {
      if (dataList) {
        for (let i = 0; i < dataList.length; i++) {
          dataList[i].income = Number.parseFloat(dataList[i].income).toFixed(2)
          dataList[i].buyerPriceByOne =  Number.parseFloat(dataList[i].buyerPriceByOne).toFixed(2)
          dataList[i].sellerPriceByOne =  Number.parseFloat(dataList[i].sellerPriceByOne).toFixed(2)
          dataList[i].buyerPrice =  Number.parseFloat(dataList[i].buyerPrice).toFixed(2)
          dataList[i].sellerPrice =  Number.parseFloat(dataList[i].sellerPrice).toFixed(2)
        }
      }
      return dataList;
    },
    tableRowClassName({rowIndex}) {
      if (rowIndex !== 0) {
        return 'success-row';
      } else {
        return '';
      }
    }
  },
};
</script>

<style>
.el-table .success-row {
  background: #e9f6ff;
}
</style>
