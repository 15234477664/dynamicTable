<template>
  <div class='tableFatherBox'>
    <div class='tableBox'>
      <el-table
        ref="multipleTable"
        :data="showList"
        v-loading="listLoading"
        header-cell-class-name="tableHeader"
        cell-class-name="cellClassName"
        height="450"
        @selection-change="handleSelectionChange">
        <el-table-column
          type="selection"
          width="60"
          align="center" v-if="isSelect">
        </el-table-column>
        <el-table-column
          prop="id"
          label="ID"
          width="80"
          align="center">
        </el-table-column>
        <el-table-column
          :label="item.propName"
          :property="item.prop"
          :sortable="item.sortable"
          :filters="item.filters"
          :filter-method="item.filterMethod"
          v-for="(item,index) in tableColumnListNew"
          :key="index"
          align="center"
          show-overflow-tooltip>
          <template slot-scope="scope">
            <span v-if="item.isCoverImg"><img :src="scope.row[scope.column.property]" alt="" style="vertical-align: top"></span>
            <span v-if="!item.selectStatus && !item.isCoverImg">{{scope.row[scope.column.property]}}</span>
            <el-tag
              v-if="item.selectStatus && !item.isCoverImg"
              :type="scope.row[scope.column.property] === item.isEnd ? 'info' : scope.row[scope.column.property] === item.isText ? 'primary' : 'success'"
              disable-transitions>{{scope.row[scope.column.property]}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          align="center"
          :width="tableWidth" v-if="isHandle">
          <template slot-scope="scope">
            <el-button type="text" size="small" v-for="(btn,index) in btnList" :key="index" v-if="btn.isShow" @click="btn.handleFun(scope)">{{btn.btnName}}</el-button>
          </template>
        </el-table-column>
        <el-table-column
          label=""
          align="center"
          width="60" :render-header="renderHeader" v-if="isDynamic">
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="pageSizeList"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>
  </div>
</template>

<script>

  export default {
    name: '',
    props: {
      tableColumnList: {
        type: Array,
        default: null
      },
      optionalColumnList: {
        type: Array,
        default: []
      },
      dataList: {
        type: Array,
        default: null
      },
      btnList: {
        type: Array,
        default: null
      },
      tableWidth: {
        type: String,
        default: null
      },
      isHandle:{
        type: Boolean,
        default: null
      },
      isDynamic:{
        type: Boolean,
        default: null
      },
      apiName:{
        type: String,
        default: null
      },
      isSelect: {
        type: Boolean,
        default: true
      }
    },
    data () {
      return {
        currentPage: 1, // 当前页
        pageNo: 1, // 页码
        total: 20, // 总条数
        pageSize: 8, // 每页展示条数
        pageSizeList: [8, 16, 34, 68], // 每页分别展示条数
        listLoading: false,
        showList: [],
        checkArr:[],
        tableColumnListNew: this.tableColumnList, // 避免页面表头导致报错
        multipleSelection: []
      }
    },
    methods: {
      // ***********************************************************分页*****************************************************
      handleSizeChange(val) {
        this.pageSize = val
        this.initList()
      },
      handleCurrentChange(val) {
        this.pageNo = val
        this.initList()
      },
      // **************************************************************初始化数据*************************************************
      initTable () {
        // 配置表头
        let selHeader = this.tableColumnListNew;
        let tableHeader = []; // 组合动态的表头
        selHeader.forEach(item => {
          let tabHeader = {};
          tabHeader.prop = item.prop;
          tabHeader.propName = item.propName;
          // 是否时间筛选
          if (item.sortable) {
            tabHeader.sortable = item.sortable;
          }
          // 是否精确筛选
          if (item.filters) {
            tabHeader.filters = item.filters;
          }
          // 精确筛选是否加样式
          if (item.selectStatus) {
            tabHeader.selectStatus = item.selectStatus;
          }
          // 精确筛选方法
          if (item.filterMethod) {
            tabHeader.filterMethod = item.filterMethod;
          }
          // 精确筛选标题
          if (item.isText) {
            tabHeader.isText = item.isText;
          }
          if (item.isEnd) { // 已结束状态
            tabHeader.isEnd = item.isEnd;
          }
          // 是否是封面
          if (item.isCoverImg) {
            tabHeader.isCoverImg = item.isCoverImg;
          }
          tableHeader.push(tabHeader);
        });
        this.tableColumnListNew = tableHeader;
        this.showList = this.dataList
      },
      // 配置表头
      menuChange(item){
        console.log(item)
        // 注意  我这里都用的假数据，要从后台获取tableColumnListNew和dataList的时候
        // 每一次调用menuChange都要重新获取tableColumnListNew和dataList，保证属性和数据是对应的
        let flag = true
        for(var i=0;i<this.checkArr.length;i++){
          if(this.checkArr[i] === item.propName){
            flag = false
            break
          }
        }
        if(!flag){
          this.tableColumnListNew.push(item)
        }
        if(flag){
          Array.prototype.contains = function(obj) {
            var j = this.length;
            while (j--) {
              if (this[j] === obj) {
                return j; // 返回的这个 i 就是元素的索引下标，
              }
            }
            return false
          }
          this.tableColumnListNew.splice(this.tableColumnListNew.contains(item),1)
        }
      },
      // 表头自定义方法
      renderHeader(h, obj) {
        if (obj.column.label === '') {
          return h('div', {}, [
            h('el-popover', {
                props: {
                  placement: "bottom",
                  width: "150",
                  trigger: "click",
                },
              },
              [
                h('span', {
                  slot: 'reference',
                  class: 'ces-table-require',
                  domProps: {
                    innerHTML: obj.column.label + '<i class="el-icon-more"/>'
                  }
                }),
                h('el-checkbox-group', {
                  class: '',
                  style: {
                    'margin-bottom': '10px',
                    'border-bottom': '1px solid #efefef',
                    'padding-bottom': '10px'
                  },
                  props: {
                    'value': this.checkArr,
                  },
                }, [
                  this.optionalColumnList.map((item, index) => {
                    return h('el-checkbox', {
                      props: {
                        label: item.propName,
                        key: index,
                        checked: this.checkArr[index] === item.propName ? true : false,
                        disabled: item.isDisabled
                      },
                      on: {
                        change: val => {
                          console.log(val)
                          if (val) {
                            this.checkArr.push(item.propName)
                            this.checkArr = [...new Set(this.checkArr)]
                            this.menuChange(item)
                          } else {
                            console.log(this.checkArr)
                            let i = this.checkArr.findIndex(row => row === item.propName)
                            this.checkArr.splice(i, 1)
                            this.menuChange(item)
                          }
                        }
                      },
                    })
                  })
                ])
              ])
          ])
        }
      },
      // 表格选择
      handleSelectionChange(val) {
        this.multipleSelection = val;
        console.log(this.multipleSelection)
      },
      // 列表
      initList (search) {},
    },
    beforeUpdate (){
      this.$nextTick(() => { //在数据加载完，重新渲染表格 ,避免动态配置表格抖动问题
        this.$refs['multipleTable'].doLayout();
      })
    },
    mounted () {
      this.initTable()
    }
  }
</script>

<style>
</style>
