<template>
  <!-- 新增商品 -->
  <div class="addprod">
    <h4>新增商品</h4>
    <el-form ref="addprod" :rules="prodrules" :model="addprod" label-width="80px">
      <el-form-item label="商品名" prop="name">
        <el-input v-model="addprod.name" placeholder="请输入商品名"></el-input>
      </el-form-item>
      <el-form-item label="价格" prop="price">
        <el-input v-model.number="addprod.price" placeholder="请输入商品价格"></el-input>
      </el-form-item>
      <el-form-item label="商品主图" prop="image">
        <el-upload
          class="prod-image"
          action="/learn/upload"
          :show-file-list="false"
          :on-success="handleSuccess"
          :before-upload="beforeUpload">
          <img v-if="imageUrl" :src="imageUrl" class="cur-image">
          <i v-else class="el-icon-plus prod-uploader-icon"></i>
        </el-upload>
      </el-form-item>
      <el-form-item label="商品类别" prop="typename">
        <el-select v-model="addprod.typename" placeholder="请选择商品类别">
          <template v-for="item in products">
            <el-option :label="item.name" :value="item.name" :key="item.type"></el-option>
          </template>
        </el-select>
      </el-form-item>

      <el-form-item label="商品类名">
        <el-input v-model="addprod.type" disabled></el-input>
      </el-form-item>
      
      <el-form-item label="是否上架">
        <el-switch v-model="addprod.selling"></el-switch>
      </el-form-item>
      
      <el-form-item label="售卖时间" prop="selltime">
        <el-input v-model="addprod.selltime" placeholder="请请输入商品售卖时间，月份或全年"></el-input>
      </el-form-item>
      
      <el-form-item label="商品简介" prop="desc">
        <el-input type="textarea" v-model="addprod.desc" placeholder="请请输入商品简介"></el-input>
      </el-form-item>

      <el-form-item label="商品详情" prop="info">
        <mavon-editor  ref="md" @imgAdd="$imgAdd" @imgDel="$imgDel" v-model="addprod.info"></mavon-editor>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="newprod">立即添加</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
import {UploadFile} from '../../../api/api'
import {GetProducts, NewProd} from '../../../api/adminApi'
export default {
  data () {
    return {
      imageUrl: '',
      products: [],
      addprod: {
        name: '',
        price: '',
        type: '',
        typename: '',
        selling: '',
        desc: '',
        info: ''
      },
      prodrules: {
        name: [
          {
            required: true,
            message: '请输入商品名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 15,
            message: '长度在 3 到 15 个字',
            trigger: 'blur'
          }
        ],
        price: [
          {
            required: true,
            message: '请输入商品价格',
            trigger: 'blur'
          },
          {
            type: 'number',
            message: '价格必须是数字',
            trigger: 'blur'
          }
        ],
        type: [
          {
            required: true,
            message: '商品必须选择一个类别',
            trigger: 'change'
          }
        ],
        desc: [
          {
            required: true,
            message: '请输入商品简介',
            trigger: 'blur'
          }
        ],
        selltime: [
          {
            required: true,
            message: '商品售卖时间段是必须的！',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  watch: {
    'addprod.typename': function (val) {
      // console.log(val)
      // console.log(this.products)
      const CurProduct = this.products.filter(p => {
        return p.name === this.addprod.typename
      })
      // console.log(CurProduct)
      this.addprod.type = CurProduct[0].type
      // console.log(this.addprod.type)
    },
    deep: true
  },
  methods: {
    // 获取所有商品分类
    getproducts () {
      let getproductspar = {
        type: 'all'
      }
      GetProducts(getproductspar).then(res => {
        this.products = res.data.products
      })
    },
    // 新增一个商品
    newprod () {
      this.$refs.addprod.validate(valid => {
        if (valid) {
          // const prodFd = new FormData()
          // prodFd.append('name', this.addprod.name)
          let addprodpar = {
            // 一种写法
            // id: this.addprod.type,
            // prod: {
            //   name: this.addprod.name,
            //   price: this.addprod.price,
            //   image: this.imageUrl,
            //   desc: this.addprod.desc,
            //   selling: this.addprod.selling,
            //   info: this.addprod.info
            // }
            // 另一种写法
            name: this.addprod.name,
            price: this.addprod.price,
            image: this.imageUrl,
            desc: this.addprod.desc,
            type: this.addprod.type,
            typename: this.addprod.typename,
            selling: this.addprod.selling,
            info: this.addprod.info,
            selltime: this.addprod.selltime
          }
          NewProd(addprodpar).then(res => {
            // console.log(res)
            this.$notify({
              title: '成功',
              type: 'success',
              message: res.data.msg,
              offset: 200
            })
            this.$refs.addprod.resetFields()
            this.$router.push('/admin/mangeprods/allprods')
          })
        } else {
          this.$notify({
            title: '警告!',
            type: 'warning',
            message: '请先完成验证',
            offset: 200
          })
        }
      })
    },
    // mavoneditor图片上传并替换地址
    // 绑定@imgAdd event
    $imgAdd (pos, $file) {
      // 第一步.将图片上传到服务器.
      let formdata = new FormData()
      formdata.append('file', $file)
      UploadFile(formdata)
      .then(url => {
        // console.log(url)
        // console.log(this.addprod.info)
        // 第二步.将返回的url替换到文本原位置![...](./0) -> ![...](url)
        this.$refs.md.$img2Url(pos, url.data)
      })
    },
    $imgDel (pos) {
      delete this.img_file[pos]
    },
    // 获取商品主图上传成功后返回的图片
    handleSuccess (res, file) {
      // console.log('res', res)
      // console.log('file', file)
      // this.imageUrl = URL.createObjectURL(file.raw)
      this.imageUrl = res
    },
    // 商品主图再上传前对文件进行判断
    beforeUpload (file) {
      const isPIC = file.type === 'image/jpeg' || 'image/png'
      const isLt5M = file.size / 1024 / 1024 < 5

      if (!isPIC) {
        this.$message.error('上传图片只能是 JPG或PNG 格式!')
      }
      if (!isLt5M) {
        this.$message.error('上传图片大小不能超过 5MB!')
      }
      return isPIC && isLt5M
    }
  },
  mounted () {
    this.getproducts()
  }
}
</script>
<style lang="less" scoped>
@import '../../../common/less/index.less';
.addprod {
  .learncontent;
  .el-form {
    text-align: left;
    .el-select {
      width: 100%;
    }
    .el-switch {
      margin: 10px 0 0 0;
    }
    .prod-image {
      width: 200px;
      height: 200px;
      border: 1px dashed #d9d9d9;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
      .cur-image {
        width: 100%;
      }
      .prod-uploader-icon {
        font-size: 45px;
        color: #8c939d;
        width: 200px;
        height: 200px;
        line-height: 200px;
        text-align: center;
      }
    }
  }
}
</style>


