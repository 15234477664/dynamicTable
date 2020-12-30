<template>
  <div class="testQuestions">
    <!--搜索-->
    <div class="searchBox">
      <searchForm :config="config" @submit="searchList" ref="form"></searchForm>
    </div>
    <!--搜索-->
    <!--列表-->
    <div class="conBox">
      <!--按钮-->
      <div class="conBtnBox">
        <el-button type="primary" size="mini" class="operateBtn" icon="el-icon-plus" @click="addBtn">新建</el-button>
      </div>
      <!--按钮-->
      <div class="conListBox">
        <searchTable
          ref="tableList"
          :tableColumnList="tableColumnList"
          :optionalColumnList="optionalColumnList"
          :dataList="dataList"
          :btnList="btnList"
          :tableWidth="tableWidth"
          :isHandle="isHandle"
          :isDynamic="isDynamic"
          :apiName="apiName">
        </searchTable>
      </div>
    </div>
    <!--列表-->
  </div>
</template>

<script>
  import searchTable from '@/components/dynamicTable'
  import searchForm from '@/components/dynamicForm'
  const payList = [
    {
      value: 1,
      label: '是'
    }, {
      value: 2,
      label: '否'
    }
  ]
  const stateList = [
    {
      value: 1,
      label: '未开始'
    }, {
      value: 2,
      label: '已开始'
    }, {
      value: 3,
      label: '已结束'
    }
  ]
  export default {
    name: "testQuestions",
    data() {
      return {
        // 表单搜索条件配置
        config: {
          columns: [
            {
              prop: "liveName",
              label: "直播名称",
              is: "text"
            },
            {
              prop: "isPay",
              label: "是否付费",
              is: "select",
              list: payList,
            },
            {
              prop: "lecturer",
              label: "讲师",
              is: "text"
            },
            {
              prop: 'liveTime',
              label: "直播时间",
              is: 'daterange',
            },
            {
              prop: 'creationTime',
              label: "创建日期",
              is: 'daterange',
            },
            {
              prop: "state",
              label: "状态",
              is: "select",
              list: stateList,
            }
          ],
          rowSize: 4,   //一行可以展示几列表单，默认为3列
        },
        // 注释 在配置数据时需要注意  selectStatus  100否添加筛选样式  sortable  针对排序筛选  filterMethod  传递筛选方法  isDisabled  100否可配置表头
        // 这里为了简便我就没有调用后台接口获取数据，直接写的假数据  你要用的话可以调用后台接口获取tableColumnList，注意数据格式
        tableColumnList: [
          {prop: 'liveCover', propName: '直播封面', isCoverImg: true},
          {prop: 'liveName', propName: '直播名称'},
          {prop: 'lecturer', propName: '讲师'},
          {prop: 'charge', propName: '收费', selectStatus: true,
            filters: [
              { text: '是', value: '是'},
              { text: '否', value: '否'},
            ],
            isText: '是',
            isEnd: '',
            filterMethod: this.filterTopic},
          {prop: 'state', propName: '状态', selectStatus: true,
            filters: [
              { text: '未开始', value: '未开始'},
              { text: '已开始', value: '已开始'},
              { text: '已结束', value: '已结束'},
            ],
            isText: '未开始',
            isEnd: '已结束',
            filterMethod: this.filterTopic},
          {prop: 'startTime', propName: '开始时间', sortable: true},
          {prop: 'creationTime', propName: '创建时间', sortable: true},
        ],
        // 动态设置想要的
        optionalColumnList: [
          {prop: 'liveCover', propName: '直播封面', isDisabled: true},
          {prop: 'liveName', propName: '直播名称', isDisabled: true},
          {prop: 'lecturer', propName: '讲师', isDisabled: true},
          {prop: 'charge', propName: '收费', isDisabled: true},
          {prop: 'state', propName: '状态', isDisabled: true},
          {prop: 'startTime', propName: '开始时间', isDisabled: true},
          {prop: 'creationTime', propName: '创建时间', isDisabled: true},
          {prop: 'price', propName: '价格（元）', isDisabled: false},
          {prop: 'endTime', propName: '结束时间', isDisabled: false},
          {prop: 'liveDescribe', propName: '直播描述', isDisabled: false},
        ],
        isHandle: true, // 是否展示操作  true 为展示 false 为不展示
        isDynamic: true, // 是否展示动态配置表头 true 为展示 false 为不展示
        // 这里为了简便我就没有调用后台接口获取数据，直接写的假数据
        dataList: [
          {
            id: '100001',
            liveName: '马克思主义基本原理概论',
            lecturer: '欧阳智贤',
            charge: '是',
            price: '1999',
            state: '未开始',
            startTime: '2020-01-04  09:41:00',
            endTime: '2020-01-04 09:41:00',
            liveDescribe: '全国教育系统劳动全国教育系统劳动全国教育系统劳动',
            creationTime: '2019-09-21 08:50:08',
          },
          {
            id: '100001',
            liveName: '马克思主义基本原理概论',
            lecturer: '欧阳智贤',
            charge: '否',
            price: '1999',
            state: '已开始',
            startTime: '2020-01-04  09:41:00',
            endTime: '2020-01-04 09:41:00',
            liveDescribe: '全国教育系统劳动全国教育系统劳动全国教育系统劳动',
            creationTime: '2019-09-21 08:50:08',
          },
          {
            id: '100001',
            liveName: '马克思主义基本原理概论',
            lecturer: '欧阳智贤',
            charge: '是',
            price: '1999',
            state: '已结束',
            startTime: '2020-01-04  09:41:00',
            endTime: '2020-01-04 09:41:00',
            liveDescribe: '全国教育系统劳动全国教育系统劳动全国教育系统劳动',
            creationTime: '2019-09-21 08:50:08',
          }
        ],
        // 页面按钮列表
        btnList: [
          {
            btnName: '编辑',
            isShow: true,
            handleFun: this.editBtn
          },
          {
            btnName: '查看',
            isShow: true,
            handleFun: this.detailBtn
          }
        ],
        // 表格操作按钮宽度
        tableWidth: '120',
        apiName: 'liveList' // 列表接口名字
      }
    },
    components: {
      searchTable,
      searchForm
    },
    methods: {
      // 搜索
      searchList (res){
        this.$refs.tableList.initList(res) // 搜索时调用子组件列表参数且把搜索参数传过去
      },
      // 筛选方法
      filterTopic (value, row, column) {
        const property = column['property'];
        return row[property] === value;
      },
      // 删除
      deleteBtn (scope) {
        this.$confirm('确认删除吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          customClass: 'myClass'
        }).then(() => {
          console.log(scope.row)
          // 注 自己操作方法
        }).catch(() => {
          return false
        })
      },
      // 添加按钮触发弹框
      addBtn (){
        this.$router.push({path: '/live/add'})
      },
    },
    watch: {},
    mounted() {}
  }
</script>

<style lang="scss" scoped>
</style>
