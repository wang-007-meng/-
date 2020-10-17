<template>
  <div>
    <el-dialog
      :title="info.isAdd ? '添加活动' : '编辑活动'"
      :visible.sync="info.isshow"
      @closed="close"
    >
      <el-form ref="form" :model="form" label-width="80px">
        <el-form-item label="活动名称">
          <el-input v-model="form.title"></el-input>
        </el-form-item>
        <!-- 时间开始 -->

        <!-- <div class="block">
          <span class="demonstration">默认</span>
          <el-date-picker
            v-model="value1"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
          >
          </el-date-picker>
        </div> -->
        <el-form-item label="活动期限">
          <el-date-picker
            v-model="value1"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
          >
          </el-date-picker>
        </el-form-item>

        <!-- 时间结束 -->

        <el-form-item label="一级分类">
          <el-select v-model="form.first_cateid" @change="changeFirst">
            <el-option label="请选择" value="" disabled></el-option>
            <el-option
              v-for="item in cateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级分类">
          <el-select v-model="form.second_cateid" @change="changeSecond">
            <el-option label="请选择" value="" disabled></el-option>
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="商品">
          <el-select v-model="form.goodsid">
            <el-option label="请选择" value="" disabled></el-option>
            <el-option
              v-for="item in thirdGoodsList"
              :key="item.id"
              :label="item.goodsname"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="状态">
          <el-switch
            v-model="form.status"
            :active-value="1"
            :inactive-value="2"
          ></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" @click="add" v-if="info.isAdd"
          >添 加</el-button
        >
        <el-button type="primary" @click="update" v-else>修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import E from "wangeditor";
import { mapGetters, mapActions } from "vuex";
import {
  reqCateList,
  reqGoodsList,
  reqSeckillAdd,
  reqSeckillUpdate,
  reqSeckillDetail,
} from "../../../utils/request";
import { successAlert, warningAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  components: {},
  data() {
    return {
      value1: [],
      form: {
        title: "",
        begintime: "",
        endtime: "",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      },
      // 二级分类的list
      secondCateList: [],
      // 商品属性的list
      goodsAttrList: [],
      // sanji
      thirdGoodsList: [],
    };
  },
  computed: {
    ...mapGetters({
      //  商品分类list
      cateList: "cate/list",
      // 商品管理list
      goodsList: "goods/list",
    }),
  },
  methods: {
    ...mapActions({
      // 请求商品分类list
      reqCateList: "cate/reqListAction",
      // goods list
      reqListAction: "goods/reqListAction",
      // seckill list
      reqSeckillListAction: "seckill/reqListAction",
    }),

    // 一级分类修改了  获取二级分类的list
    changeFirst() {
      // 一级分类变了，二级分类的值应该置空
      this.form.second_cateid = "";
      this.getSecondList();
    },
    // 获取二级分类list
    getSecondList() {
      reqCateList({ pid: this.form.first_cateid }).then((res) => {
        // 二级分类list
        this.secondCateList = res.data.list;
      });
    },

    // 获取三级分类的数据
    changeSecond() {
      reqGoodsList({
        fid: this.form.first_cateid,
        sid: this.form.second_cateid,
      }).then((res) => {
        // 三级分类
        this.thirdGoodsList = res.data.list;
      });
    },

    // 取消
    cancel() {
      this.info.isshow = false;
    },
    // 弹框消失完成
    close() {
      // 如果是添加开的弹框什么也不做，如果是编辑开的弹框，就清除form
      if (!this.info.isAdd) {
        this.empty();
      }
    },
    // 数据重置
    empty() {
      this.value1 = [];
      this.form = {
        title: "",
        begintime: "",
        endtime: "",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      };
      //二级分类的list
      this.secondCateLis = [];
      //商品属性list
      this.goodsAttrList = [];
      // 三级
      this.thirdGoodsList = [];
    },
    // 点击添加按钮
    add() {
      console.log(this.form);
      // 时间
      this.form.begintime = this.value1[0].getTime();
      this.form.endtime = this.value1[1].getTime();
      reqSeckillAdd(this.form).then((res) => {
        if (res.data.code == 200) {
          // 成功
          successAlert(res.data.msg);
          this.from = res.data.list;
          // 数据重置
          this.empty();
          // 弹框消失
          this.cancel();
          //要渲染的seckill页面的list数据要刷新
          this.reqSeckillListAction();
        } else {
          warningAlert(res.data.msg);
        }
      });
    },
    //获取菜单详情 （1条）
    look(id) {
      //发请求
      reqSeckillDetail(id).then((res) => {
        if (res.data.code == 200) {
          //这个时候form是没有id的
          this.form = res.data.list;
          // 补个id
          this.form.id = id;
          this.value1 = [
            // 查看时间
            new Date(parseInt(this.form.begintime)),
            new Date(parseInt(this.form.endtime)),
          ];
          // 需要请求一下二级分类的list
          this.getSecondList();
          this.changeSecond();
        } else {
          warningAlert(res.data.msg);
        }
      });
    },
    // 修改
    update() {
      reqSeckillUpdate(this.form).then((res) => {
        if (res.data.code == 200) {
          successAlert(res.data.msg);
          this.empty();
          this.cancel();
          this.reqListAction();
        } else {
          warningAlert(res.data.msg);
        }
      });
    },
  },
  mounted() {
    // 如果商品一级分类没有请求过  就请求一次
    if (this.cateList.length == 0) {
      this.reqCateList();
    }
  },
};
</script>
<style scoped>
.my-upload {
  width: 150px;
  height: 150px;
  border: 1px dashed #ccc;
  position: relative;
}

.my-upload h3 {
  text-align: center;
  line-height: 150px;
  font-size: 30px;
  font-weight: 400;
}

.my-upload .my-input {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
  cursor: pointer;
  opacity: 0;
}

.my-upload .img {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
}
</style>