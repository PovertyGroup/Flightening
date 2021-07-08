<template lang="pug">
  a-layout.root-layout
    a-layout-sider.desktop-only(width="200" v-model:collapsed="collapsed" style="background: #fff")
      Logo(style="margin: 25%")
      a-menu(mode="inline" v-model:selectedKeys="menuSelectedKey")
        a-menu-item(key="1")
          client-only
            TeamOutlined
          span 用户管理
        a-menu-item(key="2")
          client-only
            ContainerOutlined
          span 订单管理
        a-menu-item(key="3")
          client-only
            MonitorOutlined
          span 航线管理
        a-menu-item(key="4")
          client-only
            NodeExpandOutlined
          span 航班管理
        a-menu-item(key="5")
          client-only
            GoldOutlined
          span 省份管理
        a-menu-item(key="6")
          client-only
            BlockOutlined
          span 城市管理
    a-layout
      a-drawer.nav-drawer.mobile-only(placement="left" :closable="false" :visible="drawerVisible" @close="() => drawerVisible = false")
        template(#title)
          Logo(style="margin: 25%")
        a-menu(mode="inline" v-model:selectedKeys="menuSelectedKey" @select="() => drawerVisible = false")
          a-menu-item(key="1")
            client-only
              SettingTwoTone
            span 账号信息
          a-menu-item(key="2")
            client-only
              SnippetsTwoTone
            span 问卷列表
          a-menu-item(key="3")
            client-only
              BlockOutlined
            span 帮助
      a-layout-header
        client-only
          menu-unfold-outlined.desktop-only(v-if="collapsed" class="trigger" @click="() => (collapsed = !collapsed)")
          menu-fold-outlined.desktop-only(v-else class="trigger" @click="() => (collapsed = !collapsed)")
          MenuOutlined.mobile-only(@click="() => drawerVisible = true")
        a-dropdown(@click.prevent)
          .user-info(style="float: right;")
            span(style="font-weight: 600; margin-right: 10px") {{ this.$auth.loggedIn ? this.$auth.user.username : "" }}
            a-avatar(src="https://z3.ax1x.com/2021/07/08/RqYMiq.jpg")
          template(#overlay)
            a-menu
              a-menu-item(@click="logout")
                client-only
                  CloseCircleOutlined
                span 退出
      a-layout-content
        .content-wrap(v-if="menuSelectedKey == '1'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 用户管理
              a-button(type="link" @click="fetchUserData()")
                ReloadOutlined
            a-table(:columns="userColumns" :data-source="userData" rowKey="username")
              template(slot="email" slot-scope="text")
                a(:href="'mailto:' + text") {{ text }}
              template(slot="boolRender" slot-scope="value")
                div(style="padding-left: 23px;")
                  CheckOutlined(v-if="value")
                  CloseOutlined(v-else="value")
              template(slot="phone" slot-scope="text")
                code {{ text }}
              template(slot="action" slot-scope="record")
                a(v-if="record.blocked" @click="unblockUser(record.id)") 取消封禁
                a(v-else @click="blockUser(record.id)") 封禁用户
        .content-wrap(v-if="menuSelectedKey == '2'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 订单查看
              a-button(type="link" @click="fetchOrderData()")
                ReloadOutlined
            a-table(:columns="orderColumns" :data-source="orderData")
              template(slot="idRender" slot-scope="text")
                code {{ text }}
              template(slot="priceRender" slot-scope="text")
                span ￥{{ text }}.00
              template(slot="timeRender" slot-scope="text")
                span {{ new Date(text).toLocaleString() }}
              template(slot="affiliateRender" slot-scope="record")
                img(style="width: 32px; height: 32px;" :src="record.airline.logo")
                span(style="padding-left: 5px") {{ record.airline.affiliate }}
              template(slot="uidRender" slot-scope="text")
                a-tooltip
                  template(#title) {{ text }}
                  span {{ text.substring(0, 5) }}...
        .content-wrap(v-if="menuSelectedKey == '3'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 航线查看
              a-button(type="link" @click="fetchAirlineData()")
                ReloadOutlined
            a-table(:columns="airlineColumns" :data-source="airlineData")
              template(slot="idRender" slot-scope="text")
                code {{ text }}
              template(slot="affiliateRender" slot-scope="record")
                img(style="width: 32px; height: 32px;" :src="record.logo")
                span(style="padding-left: 5px") {{ record.affiliate }}
        .content-wrap(v-if="menuSelectedKey == '4'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 航班查看
              a-button(type="link" @click="fetchShiftData()")
                ReloadOutlined
            a-table(:columns="shiftColumns" :data-source="shiftData")
              template(slot="idRender" slot-scope="text")
                code {{ text }}
              template(slot="timeRender" slot-scope="text")
                span {{ new Date(text).toLocaleString() }}
              template(slot="priceRender" slot-scope="text")
                span ￥{{ text }}.00
        .content-wrap(v-if="menuSelectedKey == '5'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 省份管理
              div
                a-button(type="link" @click="addProvince()")
                  PlusOutlined
                a-button(type="link" @click="fetchProvinceData()")
                  ReloadOutlined
            a-table(:columns="provinceColumns" :data-source="provinceData")
              template(slot="idRender" slot-scope="text")
                code {{ text }}
              template(slot="actionRender" slot-scope="record")
                a(@click="showModal(record.id, record.name)") 修改
            a-modal(v-model:visible="modalVisible" title="修改省份" @ok="handleOk")
              a-form(layout="horizontal" v-bind="formLayout")
                a-form-item(label="id")
                  a-input(disabled v-model:value="selectedProvinceId")
                a-form-item(label="名称")
                  a-input(v-model:value="selectedProvinceName")
          a-modal(v-model:visible="addingProvinceModal" title="添加省份" @ok="addingProvinceHandleOk")
            a-form(layout="horizontal" v-bind="formLayout")
              a-form-item(label="名称")
                a-input(v-model:value="addingProvinceName")
        .content-wrap(v-if="menuSelectedKey == '6'")
          a-card.card.shadow
            div(style="display: flex; width: 100%; justify-content: space-between;")
              h1(style="font-weight: bold; margin-bottom: 20px;") 城市管理
              div
                a-button(type="link" @click="addCity()")
                  PlusOutlined
                a-button(type="link" @click="fetchCityData()")
                  ReloadOutlined
            a-table(:columns="cityColumns" :data-source="cityData")
              template(slot="idRender" slot-scope="text")
                code {{ text }}
              template(slot="actionRender" slot-scope="record")
                a(@click="showCityModal(record.id, record.code, record.name)") 修改
            a-modal(v-model:visible="cityModalVisible" title="修改城市" @ok="handleCityOk")
              a-form(layout="horizontal" v-bind="formLayout")
                a-form-item(label="id")
                  a-input(disabled v-model:value="selectedcityId")
                a-form-item(label="名称")
                  a-input(v-model:value="selectedcityName")
                a-form-item(label="代码")
                  a-input(v-model:value="selectedcityCode")
          a-modal(v-model:visible="addingCityModal" title="添加城市" @ok="addingCityHandleOk")
            a-form(layout="horizontal" v-bind="formLayout" :rules="cityRules" :model="addingCityForm")
              a-form-item(label="名称" name="addingCityName")
                a-input(v-model:value="addingCityForm.addingCityName")
              a-form-item(label="省份" name="addingCityProvince")
                a-select(v-model:value="addingCityForm.addingCityProvince")
                  a-select-option(v-for="province in provinceData" :key="province.id" :value="province.id") {{ province.name }}
              a-form-item(label="代码" name="addingCityCode")
                a-input(v-model:value="addingCityForm.addingCityCode")
</template>

<script>
import {
  TeamOutlined,
  ContainerOutlined,
  MonitorOutlined,
  NodeExpandOutlined,
  GoldOutlined,
  BlockOutlined,
  MenuUnfoldOutlined,
  MenuFoldOutlined,
  CloseCircleOutlined,
  CarryOutTwoTone,
  CheckCircleTwoTone,
  MenuOutlined,
  CheckOutlined,
  CloseOutlined,
  ReloadOutlined,
  PlusOutlined,
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'

const userColumns = [
  {
    title: 'ID',
    dataIndex: 'id',
  },
  {
    title: '用户名',
    dataIndex: 'username',
  },
  {
    title: '邮箱',
    key: 'email',
    dataIndex: 'email',
    scopedSlots: { customRender: 'email' },
  },
  {
    title: '电话号码',
    dataIndex: 'phone',
    key: 'phone',
    scopedSlots: { customRender: 'phone' },
  },
  {
    title: '通过验证',
    dataIndex: 'validated',
    key: 'validated',
    scopedSlots: { customRender: 'boolRender' },
  },
  {
    title: '是否封禁',
    dataIndex: 'blocked',
    key: 'blocked',
    scopedSlots: { customRender: 'boolRender' },
  },
  {
    title: '操作',
    key: 'action',
    scopedSlots: { customRender: 'action' },
  },
]

const orderColumns = [
  {
    title: '订单号',
    dataIndex: 'uid',
    key: 'uid',
    scopedSlots: { customRender: 'uidRender' },
  },
  {
    title: '航线',
    dataIndex: 'airline.id',
    key: 'airline.id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '航司',
    key: 'airline.affiliate',
    scopedSlots: { customRender: 'affiliateRender' },
  },
  {
    title: '起点',
    key: 'airline.origin',
    dataIndex: 'airline.origin',
  },
  {
    title: '终点',
    key: 'airline.dest',
    dataIndex: 'airline.dest',
  },
  {
    title: '型号',
    key: 'airline.model',
    dataIndex: 'airline.model',
  },
  {
    title: '起飞时间',
    key: 'shift.takeoff',
    dataIndex: 'shift.takeoff',
    scopedSlots: { customRender: 'timeRender' },
  },
  {
    title: '降落时间',
    key: 'shift.landing',
    dataIndex: 'shift.landing',
    scopedSlots: { customRender: 'timeRender' },
  },
  {
    title: '下单时间',
    key: 'time',
    dataIndex: 'time',
    scopedSlots: { customRender: 'timeRender' },
  },
  {
    title: '价格',
    key: 'price',
    dataIndex: 'price',
    scopedSlots: { customRender: 'priceRender' },
  },
]

const airlineColumns = [
  {
    title: '航线',
    dataIndex: 'id',
    key: 'id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '航司',
    key: 'affiliate',
    scopedSlots: { customRender: 'affiliateRender' },
  },
  {
    title: '起点',
    key: 'origin',
    dataIndex: 'origin',
  },
  {
    title: '终点',
    key: 'dest',
    dataIndex: 'dest',
  },
  {
    title: '型号',
    key: 'airline.model',
    dataIndex: 'model',
  },
]

const shiftColumns = [
  {
    title: '航班号',
    dataIndex: 'id',
    key: 'id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '隶属航线',
    key: 'airline_id',
    dataIndex: 'airline_id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '起飞时间',
    key: 'takeoff',
    dataIndex: 'takeoff',
    scopedSlots: { customRender: 'timeRender' },
  },
  {
    title: '降落时间',
    key: 'landing',
    dataIndex: 'landing',
    scopedSlots: { customRender: 'timeRender' },
  },
  {
    title: '经济舱价格',
    key: 'economic_price',
    dataIndex: 'economic_price',
    scopedSlots: { customRender: 'priceRender' },
  },
  {
    title: '商务舱价格',
    key: 'business_price',
    dataIndex: 'business_price',
    scopedSlots: { customRender: 'priceRender' },
  },
  {
    title: '高等舱价格',
    key: 'premium_price',
    dataIndex: 'premium_price',
    scopedSlots: { customRender: 'priceRender' },
  },
  {
    title: '头等舱价格',
    key: 'first_price',
    dataIndex: 'first_price',
    scopedSlots: { customRender: 'priceRender' },
  },
]

const provinceColumns = [
  {
    title: '省份ID',
    dataIndex: 'id',
    key: 'id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '省份',
    key: 'name',
    dataIndex: 'name',
  },
  {
    title: '操作',
    key: 'action',
    scopedSlots: { customRender: 'actionRender' },
  },
]

const cityColumns = [
  {
    title: '城市ID',
    dataIndex: 'id',
    key: 'id',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '城市代码',
    dataIndex: 'code',
    key: 'code',
    scopedSlots: { customRender: 'idRender' },
  },
  {
    title: '城市',
    key: 'name',
    dataIndex: 'name',
  },
  {
    title: '省份',
    key: 'province',
    dataIndex: 'province',
  },
  {
    title: '操作',
    key: 'action',
    scopedSlots: { customRender: 'actionRender' },
  },
]

export default {
  components: {
    TeamOutlined,
    ContainerOutlined,
    MonitorOutlined,
    NodeExpandOutlined,
    GoldOutlined,
    BlockOutlined,
    MenuUnfoldOutlined,
    MenuFoldOutlined,
    CloseCircleOutlined,
    CarryOutTwoTone,
    CheckCircleTwoTone,
    MenuOutlined,
    CheckOutlined,
    CloseOutlined,
    ReloadOutlined,
    PlusOutlined,
  },
  data() {
    return {
      collapsed: false,
      menuSelectedKey: ['1'],
      drawerVisible: false,
      userColumns,
      userData: [],
      orderColumns,
      orderData: [],
      airlineColumns,
      airlineData: [],
      shiftColumns,
      shiftData: [],
      provinceColumns,
      provinceData: [],
      modalVisible: false,
      selectedProvinceId: 0,
      selectedProvinceName: '',
      cityColumns,
      cityData: [],
      cityModalVisible: false,
      selectedcityId: 0,
      selectedcityName: '',
      selectedcityCode: '',

      addingProvinceName: '',
      addingCityForm: {
        addingCityName: '',
        addingCityCode: '',
        addingCityProvince: 1,
      },

      addingProvinceModal: false,
      addingCityModal: false,

      formLayout: {
        labelCol: { span: 4 },
        wrapperCol: { span: 14 },
      },
      cityRules: {
        addingCityCode: [
          { required: true, message: '请输入城市代码', trigger: 'blur' },
          { min: 3, max: 3, message: '代码长度为3', trigger: 'blur' },
        ],
        addingCityName: [
          { required: true, message: '请输入城市名', trigger: 'blur' },
        ],
        addingCityProvince: [
          { required: true, message: '请选择城市', trigger: 'blur' },
        ],
      },
    }
  },
  mounted() {
    this.fetchUserData()
    this.fetchOrderData()
    this.fetchAirlineData()
    this.fetchShiftData()
    this.fetchProvinceData()
    this.fetchCityData()
  },
  methods: {
    logout() {
      this.$auth.logout()
      this.$router.push('/login')
    },
    fetchUserData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/admin/users').then((res) => {
        this.userData = res.data.data
      })
    },
    fetchOrderData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/admin/orders').then((res) => {
        this.orderData = res.data.data
      })
    },
    fetchAirlineData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/admin/airlines').then((res) => {
        this.airlineData = res.data.data
      })
    },
    fetchShiftData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/admin/shifts').then((res) => {
        this.shiftData = res.data.data
      })
    },
    fetchProvinceData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/admin/province').then((res) => {
        this.provinceData = res.data.data
      })
    },
    fetchCityData() {
      // fetch userdata
      this.$axios.get('http://localhost:8080/v1/airline/cities').then((res) => {
        this.cityData = res.data.result
      })
    },
    blockUser(uid) {
      this.$axios
        .post(`http://localhost:8080/v1/admin/users/${uid}/block`)
        .then((res) => {
          this.fetchUserData()
        })
        .catch(() => message.error('失败'))
    },
    unblockUser(uid) {
      this.$axios
        .post(`http://localhost:8080/v1/admin/users/${uid}/unblock`)
        .then((res) => {
          this.fetchUserData()
        })
        .catch(() => message.error('失败'))
    },
    showModal(id, name) {
      this.selectedProvinceId = id
      this.selectedProvinceName = name
      this.modalVisible = true
    },
    handleOk() {
      this.$axios
        .put('http://localhost:8080/v1/admin/province', {
          id: this.selectedProvinceId,
          name: this.selectedProvinceName,
        })
        .catch((e) => {
          message.error(e.response.data.msg)
        })
        .then(() => {
          this.fetchProvinceData()
        })
      this.modalVisible = false
    },
    showCityModal(id, code, name) {
      this.selectedcityId = id
      this.selectedcityCode = code
      this.selectedcityName = name
      this.cityModalVisible = true
    },
    handleCityOk() {
      this.$axios
        .put('http://localhost:8080/v1/admin/city', {
          id: this.selectedcityId,
          name: this.selectedcityName,
          code: this.selectedcityCode,
        })
        .catch((e) => {
          message.error(e.response.data.msg)
        })
        .then(() => {
          this.fetchCityData()
        })
      this.cityModalVisible = false
    },
    addProvince() {
      this.addingProvinceModal = true
    },
    addProvinceHandleOk() {
      this.$axios
        .post('http://localhost:8080/v1/admin/province', {
          name: this.addingProvinceName,
        })
        .catch((e) => {
          message.error(e.response.data.msg)
        })
        .then(() => {
          this.fetchProvinceData()
        })
      this.addingProvinceModal = false
    },
    addCity() {
      this.addingCityModal = true
    },
    addingCityHandleOk() {
      this.$axios
        .post('http://localhost:8080/v1/admin/city', {
          name: this.addingCityForm.addingCityName,
          code: this.addingCityForm.addingCityCode,
          province: this.addingCityForm.addingCityProvince,
        })
        .catch((e) => {
          message.error(e.response.data.msg)
        })
        .then(() => {
          this.fetchCityData()
        })
      this.addingCityModal = false
    },
  },
}
</script>

<style scoped>
.ant-menu-item {
  font-weight: 700;
}

.ant-menu-item > span:last-child {
  margin-left: 10px;
}

.ant-layout-header {
  padding-left: 20px;
}

.mobile-only {
  display: none;
}

.content-wrap {
  padding: 10px;
  min-height: 1vh;
}

.shadow:hover {
  border-color: rgba(0, 0, 0, 0.09);
  box-shadow: 0 2px 8px rgb(0 0 0 / 9%);
}

.card {
  width: calc(100% - 32px);
  display: inline-block;
  margin: 16px;
}

@media screen and (max-width: 500px) {
  .desktop-only {
    display: none;
  }
  .mobile-only {
    display: unset;
  }
  .ant-layout-header {
    padding-right: 20px;
  }
}

@media screen and (max-width: 500px) {
  .desktop-only {
    display: none;
  }
  .mobile-only {
    display: unset;
  }
}
</style>

<style>
.ant-layout-header {
  background: #fff;
}
.ant-menu-item {
  margin: 0 10px !important;
  width: calc(100% - 20px) !important;
  padding-left: 24px !important;
  border-radius: 40px;
}
.ant-menu-inline-collapsed > .ant-menu-item {
  padding: 0 22px !important;
}
.large-avatar > .ant-avatar-string {
  font-size: 32px;
}
.ant-menu-item::after {
  border-right: none !important;
}
.nav-drawer .ant-drawer-body {
  padding: 20px 0 0 0;
}
code {
  background: rgb(240, 240, 240);
  border-radius: 4px;
  padding: 0 6px;
}

.root-layout.ant-layout.ant-layout-has-sider {
  min-height: 100vh;
}

@media screen and (max-width: 700px) {
  .ant-modal {
    width: 100% !important;
  }
}
</style>
