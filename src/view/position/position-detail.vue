<template>
  <div class="container">
    <div class="title">
      <span>职位详细信息</span>
      <span class="back" @click="back">
        <i class="iconfont icon-fanhui"></i> 返回
      </span>
    </div>
    <el-divider></el-divider>
    <div class="wrap">
      <el-row>
        <el-col :lg="16" :md="20" :sm="24" :xs="24">
          <el-form :model="form" status-icon ref="form" label-width="100px" v-loading="loading" @submit.native.prevent>
            <el-form-item label="职位名称" prop="title">
              <span>{{ form.title }}</span>
            </el-form-item>
            <el-form-item label="所属分类" prop="category_name">
              <span>{{ form.category_name }}</span>
            </el-form-item>
            <el-form-item label="职位浏览量" prop="hits">
              <span>{{ form.hits }}</span>
            </el-form-item>
            <el-form-item label="招聘人数" prop="quantity">
              <span>{{ form.quantity }}</span>
            </el-form-item>
            <el-form-item label="薪资" prop="salary_up">
              <span>{{ form.salary_down }}~{{ form.salary_up }}</span>
            </el-form-item>
            <el-form-item label="工作地点" prop="city">
              <span>{{ form.city }}</span>
            </el-form-item>
            <el-form-item label="职位要求" prop="requirement">
              <span>{{ form.requirement }}</span>
            </el-form-item>
            <el-form-item label="公司名称" prop="company_name">
              <span>{{ form.company_name }}</span>
            </el-form-item>
            <el-form-item label="公司简介" prop="company_desc">
              <span>{{ form.company_desc }}</span>
            </el-form-item>
            <el-form-item class="submit">
              <!--通过职位的状态属性state来控制通过按钮和不通过按钮是否显示-->
              <el-button type="primary" @click="handlePass('form')" v-if="form.state == 0 && flag">通 过</el-button>
              <el-button @click="handleFail('form')" v-if="form.state == 0 && flag">不通过</el-button>
              <el-button type="primary" @click="handleDelivery('form')" v-if="temp == 1">投递简历</el-button>
              <el-button @click="handleCollect('form')" v-if="temp == 1">收藏职位</el-button>
            </el-form-item>
          </el-form>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import { Position } from '@/model/position'
import { Application } from '@/model/application'
import { Resume } from '@/model/resume'
import { Favor } from '@/model/favor'
import router from '../../router'

export default {
  // 定义变量
  props: {
    // 子组件定义的变量通过父组件赋值
    detailPositionID: {
      type: Number,
    },
    flag: {
      type: Boolean,
      default: true,
    },
    temp: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      loading: false,
      form: {
      },
      list: {
        user_id: '',
        position_id: '',
        resume_id: '',
        hr_id: '',
        company_id: '',
      },
      collect: {
      },
    }
  },
  async mounted() {
    this.loading = true
    const res = await Position.getById(this.detailPositionID)
    // 更新职位浏览量(每进入一次职位详情页面，职位浏览量加一)
    await Position.updateHits(this.detailPositionID)
    this.form = res
    this.getInfo()
    this.loading = false
  },
  methods: {
    async handlePass(val) {
      const res = await Position.updateState(this.detailPositionID, 1)
      if (res.code < window.MAX_SUCCESS_CODE) {
        this.$message.success(`${res.message}`)
        // 返回查询所有未审核职位的页面
        this.$emit('detailClose')
      }
    },
    async handleFail(val) {
      const res = await Position.updateState(this.detailPositionID, 2)
      if (res.code < window.MAX_SUCCESS_CODE) {
        this.$message.success(`${res.message}`)
        // 返回查询所有未审核职位的页面
        this.$emit('detailClose')
      }
    },
    // 投递简历
    async handleDelivery(val) {
      if (this.$store.getters.user === null) {
        this.$message.error("请注册登录后投递简历")
        return
      }
      // 根据用户id和职位id判断用户是否已经投递过该职位，避免重复投递
      const flag = await Application.getByPositionIdAndUserId(this.detailPositionID, this.$store.getters.user.id)
      if (flag === false) {
        // 不能重复投递
        this.$message.error("你已投递过该职位，不可重复投递")
      } else {
        // 获取当前登录求职者的用户id
        this.list.user_id = this.$store.getters.user.id
        // 获取当前投递简历的职位id
        this.list.position_id = this.detailPositionID
        // 根据用户id查询绑定的简历id
        const temp = await Resume.getByUserId(this.$store.getters.user.id)
        this.list.resume_id = temp.resume_id
        // 根据职位id查询该职位所对应的hr_id
        const result = await Position.getById(this.detailPositionID)
        this.list.hr_id = result.hr_id
        this.list.company_id = result.company_id
        // 调用创建申请的方法
        const res = await Application.create(this.list)
        if (res.code < window.MAX_SUCCESS_CODE) {
          this.$message.success(`${res.message}`)
        }
      }
    },
    async getInfo() {
      // 获取当前登录求职者的用户id
      this.list.user_id = this.$store.getters.user.id
      // 获取当前投递简历的职位id
      this.list.position_id = this.detailPositionID
      // 根据用户id查询绑定的简历id
      const temp = await Resume.getByUserId(this.$store.getters.user.id)
      this.list.resume_id = temp.id
      // 根据职位id查询该职位所对应的hr_id
      const result = await Position.getById(this.detailPositionID)
      this.list.hr_id = result.hr_id
      this.list.company_id = result.company_id
    },
    // 收藏职位
    async handleCollect(val) {
      if (this.$store.getters.user === null) {
        this.$message.error("请注册登录后收藏职位")
        return
      }
      // 获取当前登录求职者的用户id
      this.collect.user_id = this.$store.getters.user.id
      // 获取当前收藏的职位id
      this.collect.position_id = this.detailPositionID
      // 调用添加收藏的方法
      const res = await Favor.create(this.collect)
      if (res.code < window.MAX_SUCCESS_CODE) {
        this.$message.success(`${res.message}`)
      }
    },
    back() {
      // this.$router.go(-1)
      // 由于showEdit这个变量是在父组件定义的，因此不能在子组件改变该变量的值，可以使用事件传递的方式，当点击了返回按钮，触发的事件处理机制交给父组件去处理
      this.$emit('detailClose')
    },
  },
}
</script>

<style lang="scss" scoped>
.el-divider--horizontal {
  margin: 0;
}

.container {
  text-align: left;

  .title {
    height: 59px;
    line-height: 59px;
    color: $parent-title-color;
    font-size: 16px;
    font-weight: 500;
    text-indent: 40px;

    .back {
      float: right;
      margin-right: 40px;
      cursor: pointer;
    }
  }

  .wrap {
    padding: 20px;
  }

  .submit {
    float: left;
  }
}
</style>
