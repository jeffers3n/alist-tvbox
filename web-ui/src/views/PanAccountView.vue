<template>
  <div class="list">
    <h1>网盘账号列表</h1>
    <el-row justify="end">
      <el-button @click="load">刷新</el-button>
      <el-button type="primary" @click="handleAdd">添加</el-button>
    </el-row>
    <div class="space"></div>

    <el-table :data="accounts" border style="width: 100%">
      <el-table-column prop="id" label="ID" sortable width="70">
        <template #default="scope">
          {{ scope.row.id + 4000 }}
        </template>
      </el-table-column>
      <el-table-column prop="type" label="类型" sortable width="150">
        <template #default="scope">
          <span v-if="scope.row.type=='QUARK'">夸克网盘</span>
          <span v-else-if="scope.row.type=='UC'">UC网盘</span>
          <span v-else-if="scope.row.type=='QUARK_TV'">夸克TV</span>
          <span v-else-if="scope.row.type=='UC_TV'">UC TV</span>
          <span v-else-if="scope.row.type=='PAN115'">115云盘</span>
          <span v-else-if="scope.row.type=='OPEN115'">115 Open</span>
          <span v-else-if="scope.row.type=='THUNDER'">迅雷云盘</span>
          <span v-else-if="scope.row.type=='CLOUD189'">天翼云盘</span>
          <span v-else-if="scope.row.type=='PAN139'">移动云盘</span>
          <span v-else-if="scope.row.type=='PAN123'">123网盘</span>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="名称" sortable width="200"/>
      <el-table-column label="路径">
        <template #default="scope">
          {{ fullPath(scope.row) }}
        </template>
      </el-table-column>
      <el-table-column prop="master" label="主账号？" width="120">
        <template #default="scope">
          <el-icon v-if="scope.row.master">
            <Check/>
          </el-icon>
          <el-icon v-else>
            <Close/>
          </el-icon>
        </template>
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="200">
        <template #default="scope">
          <el-button link type="primary" size="small" @click="handleEdit(scope.row)">编辑</el-button>
          <el-button link type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog v-model="formVisible" :title="dialogTitle" width="60%">
      <el-form :model="form" label-width="140">
        <el-form-item label="名称" label-width="140" required>
          <el-input v-model="form.name" autocomplete="off"/>
        </el-form-item>
        <el-form-item label="类型" label-width="120" required>
          <el-radio-group v-model="form.type" class="ml-4">
            <el-radio label="QUARK" size="large">夸克网盘</el-radio>
            <el-radio label="UC" size="large">UC网盘</el-radio>
            <el-radio label="QUARK_TV" size="large">夸克TV</el-radio>
            <el-radio label="UC_TV" size="large">UC TV</el-radio>
            <el-radio label="PAN115" size="large">115云盘</el-radio>
            <el-radio label="OPEN115" size="large">115 Open</el-radio>
            <el-radio label="THUNDER" size="large">迅雷云盘</el-radio>
            <el-radio label="CLOUD189" size="large">天翼云盘</el-radio>
            <el-radio label="PAN139" size="large">移动云盘</el-radio>
            <el-radio label="PAN123" size="large">123网盘</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="Cookie" required v-if="form.type=='QUARK'||form.type=='UC'||form.type=='PAN115'">
          <el-input v-model="form.cookie" type="textarea" :rows="5"/>
          <a href="https://pan.quark.cn/" target="_blank" v-if="form.type=='QUARK'">夸克网盘</a>
          <a href="https://drive.uc.cn/" target="_blank" v-if="form.type=='UC'">UC网盘</a>
          <a href="https://115.com/" target="_blank" v-if="form.type=='PAN115'">115云盘</a>
        </el-form-item>
        <el-form-item label="Token" v-if="form.type=='PAN139'" required>
          <el-input v-model="form.token" type="textarea" :rows="5"/>
          <a href="https://yun.139.com/" target="_blank">移动云盘</a>
        </el-form-item>
        <el-form-item label="Refresh Token" required v-if="form.type=='OPEN115'">
          <el-input v-model="form.token"/>
          <a href="https://alist.nn.ci/zh/tool/115/token" target="_blank">获取刷新令牌</a>
        </el-form-item>
        <el-form-item label="Token" v-if="form.type=='PAN115'">
          <el-input v-model="form.token"/>
        </el-form-item>
        <el-form-item label="Token" v-if="form.type=='QUARK_TV'||form.type=='UC_TV'" required>
          <el-input v-model="form.token" type="textarea" :rows="3"/>
        </el-form-item>
        <el-form-item v-if="form.type=='QUARK_TV'||form.type=='UC_TV'" required>
          <el-button type="primary" @click="showQrCode">扫码获取</el-button>
        </el-form-item>
        <el-form-item label="用户名" v-if="form.type=='THUNDER'||form.type=='CLOUD189'||form.type=='PAN123'" required>
          <el-input v-model="form.username" :placeholder="form.type=='THUNDER'?'+86 12345678900':''" />
        </el-form-item>
        <el-form-item label="密码" v-if="form.type=='THUNDER'||form.type=='CLOUD189'||form.type=='PAN123'" required>
          <el-input type="password" show-password v-model="form.password"/>
        </el-form-item>
        <el-form-item label="验证码" v-if="form.type=='THUNDER'||form.type=='CLOUD189'">
          <el-input v-model="form.token"/>
        </el-form-item>
        <el-form-item label="保险箱密码" v-if="form.type=='THUNDER'">
          <el-input type="password" show-password v-model="form.safePassword"/>
        </el-form-item>
        <el-form-item label="文件夹ID">
          <el-input v-model="form.folder"/>
        </el-form-item>
        <el-form-item v-if="form.type=='PAN115'" label="本地代理">
          <el-switch
            v-model="form.useProxy"
            inline-prompt
            active-text="开启"
            inactive-text="关闭"
          />
        </el-form-item>
        <el-form-item label="主账号" v-if="form.type!='OPEN115'&&form.type!='QUARK_TV'&&form.type!='UC_TV'">
          <el-switch
            v-model="form.master"
            inline-prompt
            active-text="是"
            inactive-text="否"
          />
          <span class="hint">主账号用来观看分享。</span>
        </el-form-item>
        <span v-if="form.name">完整路径： {{ fullPath(form) }}</span>
      </el-form>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="handleCancel">取消</el-button>
        <el-button type="primary" @click="handleConfirm">{{ updateAction ? '更新' : '添加' }}</el-button>
      </span>
      </template>
    </el-dialog>

    <el-dialog v-model="dialogVisible" title="删除网盘账号" width="30%">
      <p>是否删除网盘账号 - {{ form.id + 4000 }}</p>
      <p> {{ getTypeName(form.type) }} ： {{ form.name }}</p>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="danger" @click="deleteAccount">删除</el-button>
      </span>
      </template>
    </el-dialog>

    <el-dialog v-model="qrModel" title="扫码登陆" width="40%">
      <img alt="qr" :src="'data:image/jpeg;base64,' + qr.qr_data" />
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="qrModel=false">取消</el-button>
        <el-button @click="showQrCode">刷新二维码</el-button>
        <el-button type="primary" @click="getRefreshToken">我已扫码</el-button>
      </span>
      </template>
    </el-dialog>

  </div>
</template>

<script setup lang="ts">
import {onMounted, ref} from 'vue'
import {Check, Close} from '@element-plus/icons-vue'
import axios from "axios"
import {ElMessage} from "element-plus";

const updateAction = ref(false)
const dialogTitle = ref('')
const accounts = ref([])
const formVisible = ref(false)
const dialogVisible = ref(false)
const qrModel = ref(false)
const form = ref({
  id: 0,
  type: 'QUARK',
  name: '',
  cookie: '',
  token: '',
  username: '',
  password: '',
  safePassword: '',
  folder: '',
  useProxy: false,
  master: false,
})
const qr = ref({
  qr_data: '',
  query_token: '',
})

const handleAdd = () => {
  dialogTitle.value = '添加网盘账号'
  updateAction.value = false
  form.value = {
    id: 0,
    type: 'QUARK',
    name: '',
    cookie: '',
    token: '',
    username: '',
    password: '',
    safePassword: '',
    folder: '',
    useProxy: false,
    master: false,
  }
  formVisible.value = true
}

const getTypeName = (type: string) => {
  if (type == 'QUARK') {
    return '夸克网盘'
  }
  if (type == 'UC') {
    return 'UC网盘'
  }
  if (type == 'QUARK_TV') {
    return '夸克TV网盘'
  }
  if (type == 'UC_TV') {
    return 'UC TV网盘'
  }
  if (type == 'PAN115') {
    return '115云盘'
  }
  if (type == 'OPEN115') {
    return '115 Open'
  }
  if (type == 'THUNDER') {
    return '迅雷云盘'
  }
  if (type == 'CLOUD189') {
    return '天翼云盘'
  }
  if (type == 'PAN139') {
    return '移动云盘'
  }
  if (type == 'PAN123') {
    return '123网盘'
  }
  return '未知'
}

const fullPath = (share: any) => {
  const path = share.name;
  if (path.startsWith('/')) {
    return path
  }
  if (share.type == 'QUARK') {
    return '/🌞我的夸克网盘/' + path
  } else if (share.type == 'UC') {
    return '/🌞我的UC网盘/' + path
  } else if (share.type == 'QUARK_TV') {
    return '/我的夸克网盘/' + path
  } else if (share.type == 'UC_TV') {
    return '/我的UC网盘/' + path
  } else if (share.type == 'PAN115') {
    return '/115云盘/' + path
  }  else if (share.type == 'OPEN115') {
    return '/115网盘/' + path
  } else if (share.type == 'THUNDER') {
    return '/我的迅雷云盘/' + path
  } else if (share.type == 'CLOUD189') {
    return '/我的天翼云盘/' + path
  } else if (share.type == 'PAN139') {
    return '/我的移动云盘/' + path
  } else if (share.type == 'PAN123') {
    return '/我的123网盘/' + path
  } else {
    return '/网盘/' + path
  }
}

const handleEdit = (data: any) => {
  dialogTitle.value = '更新网盘账号 - ' + data.name
  updateAction.value = true
  form.value = Object.assign({}, data)
  formVisible.value = true
}

const handleDelete = (data: any) => {
  form.value = data
  dialogVisible.value = true
}

const deleteAccount = () => {
  dialogVisible.value = false
  axios.delete('/api/pan/accounts/' + form.value.id).then(() => {
    load()
  })
}

const handleCancel = () => {
  formVisible.value = false
}

const handleConfirm = () => {
  const url = updateAction.value ? '/api/pan/accounts/' + form.value.id : '/api/pan/accounts'
  axios.post(url, form.value).then(() => {
    formVisible.value = false
    if (accounts.value.length === 0) {
      ElMessage.success('添加成功')
    } else {
      ElMessage.success('更新成功')
    }
    load()
  })
}

const showQrCode = () => {
  axios.post('/api/pan/accounts/-/qr?type=' + form.value.type).then(({data}) => {
    qr.value = data
    qrModel.value = true
  })
}

const getRefreshToken = () => {
  axios.post('/api/pan/accounts/-/token?type=' + form.value.type + '&queryToken=' + qr.value.query_token).then(({data}) => {
    form.value.token = data
    qrModel.value = false
  })
}

const load = () => {
  axios.get('/api/pan/accounts').then(({data}) => {
    accounts.value = data
  })
}

onMounted(() => {
  load()
})
</script>

<style scoped>
.space {
  margin-bottom: 6px;
}

.json pre {
  height: 600px;
  overflow: scroll;
}
</style>
