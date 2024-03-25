<template>
<div class="vxeGridComponent">
  <div :style="{minHeight: '60px', height: noPage ? '100%' : 'calc(100% - 36px)'}">
    <vxe-grid
      ref="VxeTableGrid"
      :size="formSize"
      height="100%"
      :row-id="rowId"
      :border="true"
      :round="true"
      :show-header="showHeader"
      show-header-overflow 
      :auto-resize="true"
      :resizable="true"
      :data="dataList"
      :loading="tableLoading"
      :column-key='true'
      :stripe="true"
      :keep-source="true"
      :show-overflow="true"
      :highlight-hover-row="true"
      :highlight-current-row="true"
      :columns="tableColumn || []"
      :row-style="setRowStyle"
      :cell-style="cellStyle"
      :export-config="{}" 
      :seq-config="tableSeqConfig"
      :edit-config="tableEditConfig"  
      :tree-config="treeConfig"
      :edit-rules="editRules"
      :checkbox-config="checkboxConfig"
      @cell-click="cellClickEvent"
      @edit-actived="rowEditActived"
      @sort-change="sortChange"
      @edit-closed="rowEditClosedEvent"
      @current-change="currentChange"
      @checkbox-change="selectedChange"
      @radio-change="radioChange"
      @checkbox-all="selectedChange"
      @checkbox-range-start="checkboxRangeStart"
      @checkbox-range-change="checkboxRangeChange"
    >
      <template v-if="$slots.toolbar" v-slot:toolbar>
        <slot name="toolbar"></slot>
      </template> 
    </vxe-grid>
  </div>

  <!-- 分页控件 -->
  <div class="pager-container" v-if="!noPage">
    <vxe-pager v-if="!isSeparation"
      size="mini"
      :loading="tableLoading"
      :align="pagerConfig.align"
      :total="Number(pagerConfig.total)"
      :layouts="pagerConfig.layouts"
      :page-size="pagerConfig.pageSize"
      :page-sizes="pagerConfig.pageSizes"
      :current-page="pagerConfig.pageNum || pagerConfig.currentPage"
      @page-change="handlePageChange"
    >
      <template v-slot:left v-if="isShowColumnLayout"> 
        <el-button size="mini" type="primary" icon="el-icon-setting" @click="setTableColumnLayout">列表设置</el-button>
      </template>
      <template v-slot:right  v-if="isShowColumnLayout && !noShrink">  
        <el-button size="mini" class="putAwayBtn" v-if="!shrinkStatus"
          icon="el-icon-caret-bottom" @click="handlePutAway">默认收缩</el-button>
        <el-button size="mini" class="putAwayBtn" v-if="shrinkStatus"
          icon="el-icon-caret-top" @click="handlePutAway">默认展开</el-button> 
      </template>
    </vxe-pager>
    <vxe-pager v-else
      size="mini"
      :loading="tableLoading"
      :align="pagerConfig.align"
      :total="totalNums"
      :layouts="pagerConfig.layouts"
      :page-size="pagerConfig.pageSize"
      :current-page="pagerConfig.currentPage"
      @page-change="handlePageChange"
    >
      <template v-slot:right>
        <span>共</span> 
        <div class="pageCount pageOne" 
          @click="countHandle"
          v-loading="loadingPage" 
          element-loading-spinner="el-icon-loadi"
          element-loading-background="rgba(170, 170, 170, 0.8)"
        > 
          <span>{{countNums}}</span>
          <i class="el-icon-refresh refreshBox"></i>
        </div>  
        <span>条记录，共</span>
        <div class="pageCount pageTwo"
          v-loading="loadingPage"
          element-loading-spinner="el-icon-loadi"
          element-loading-background="rgba(170, 170, 170, 0.8)"
        >
          <span>{{pageNums}}</span>
        </div>
        <span>页</span>
      </template>
    </vxe-pager>
  </div>

  <vxe-modal v-model="showColumnModel" width="630" height="440" show-footer>
    <template v-slot:title>
      <span>表格列设置（支持上下拖拽排序）</span>
    </template>
    <template v-slot:footer>
      <el-button size="mini" type="primary" @click="saveCurrentLayout">保&nbsp;存</el-button>
      <el-button size="mini" @click="ColumnModelResetDefault">恢复系统默认布局</el-button>
      <el-button size="mini" @click="ColumnModelCancel">取&nbsp;消</el-button>
    </template>
    <template v-slot>
      <vxe-grid
        :border="true"
        :resizable="true"
        height="auto"
        :auto-resize="true"
        :data="fieldColumnData"
        :stripe="true"
        :keep-source="true"
        :show-overflow="true"
        :highlight-hover-row="true"
        :highlight-current-row="true"
        :columns="tableFieldColumn || []"
        :row-style="setRowStyle"> 
      </vxe-grid>
    </template>
  </vxe-modal>
</div>
</template>

<script>
  import Sortable from 'sortablejs'
  export default {
    props:{
      formSize: {
        type: String,
        default(){
          return 'mini'
        }
      },
      rowId: {
        type: String,
        default(){
          return 'id'
        }
      },
      showHeader: {
        type: Boolean,
        default: true,
      },
      tableLoading: {
        type: Boolean,
        default(){
          return false;
        },
      },
      shrinkTab: {
        type: Boolean,
        default(){
          return false;
        },
      },
      defaultColumnArr: {
        type: Array,
        default(){
          return [];
        }
      },
      tableColumn: {
        type: Array,
        default(){
          return [];
        },
        required: true,
      },
      pagerConfig: {
        type: Object,
        default(){
          return {};
        },
      },
      dataList: {
        type: Array,
        required: true,
        default(){
          return [];
        },
      },
      isShowColumnLayout:{
        type: Boolean,
        default(){
          return false;
        },
      },
      noShrink:{
        type: Boolean,
        default(){
          return false;
        },
      },
      layoutPath:{
        type:String,
        default(){
          return "";
        }
      },
      noPage:{
        type:Boolean,
        default() {
          return false;
        }
      },
      countNums: {
        type:String,
        default(){
          return "";
        }
      },
      isSeparation:{
        type:Boolean,
        default() {
          return false;
        }
      }, 
      setRowStyle: [Object, Function],
      cellStyle: [Object, Function],
      treeConfig: {
        type: Object,
        default(){
          return {};
        },
      },
      checkboxConfig: {
        type: Object,
        default: () => ({})
      },
      editRules: {
        type: Object,
        default(){
          return {};
        },
      }
    },
    computed:{
      tableSeqConfig() {
        let { currentPage, pageNum, pageSize } = this.pagerConfig
        return { startIndex: ((pageNum || currentPage) - 1) * pageSize }
      }
    },
    data(){
      return {
        loading: false,
        loadingPage:false,
        totalNums:1000000000,
        pageNums:'',
        tableData: [],
        shrinkStatus:'',
        tableEditConfig:{
          mode: 'cell',
          trigger: 'dblclick',
        },
        selectedStart: null,
        selectedEnd: null,
        keyCodeSelected: [],
        ctrlKeySelected: {},
        fieldColumnData:[],
        showColumnModel: false,
        tableFieldColumn: [
          { field: "title", title: "类别名称", width: "200" },
          { field: "showStatus", title: "显示", width: "100", cellRender: { name: '$switch' }},
          { field: "lockStatus", title: "锁定", width: "100",
            slots: { default: ({ row }, h) => {
                return [
                  h('label', { class: ['toggle-lock', row.lockStatus ? 'on' : 'off'], on: {
                    click: ()=>{ this.ColumnLockTaggle(row) } }},[
                     h('span', { on: { click: () => {} }}, '')
                  ]),
                ];
              }
          }},
          { field: "operation", title: "操作", width: "200",
            slots: { default: ({ row }, h) => {
              return [
                  h('a', {  class: "el-icon-top operationAction", attrs:{ title:'往上一行' } , on: { click: () => {this.locationChange(row,'previousLine')} }}),
                  h('a', {  class: "el-icon-upload2 operationAction", attrs:{ title:'置顶' }, on: { click: () => {this.locationChange(row,'top')} }}),
                  h('a', {  class: "el-icon-bottom operationAction", attrs:{ title:'往下一行' }, on: { click: () => {this.locationChange(row,'nextStep')} }}),
                  h('a', {  class: "el-icon-download operationAction", attrs:{ title:'置底' }, on: { click: () => {this.locationChange(row,'bottom')} }}),
               ];
            }
          }},
        ]
      }
    },
    watch:{
      shrinkTab: {
  　　　　immediate: true,
  　　　　handler: function(oldVal,newVal) {
            this.shrinkStatus = oldVal;
  　　　　}
  　　},
      countNums(val){  
        this.pageNums = val ? Math.ceil(val / this.pagerConfig.pageSize) : ''; 
        this.loadingPage = false; 
      }
    },
    created(){ 
      if(this.layoutPath){  //判断是否含有路径名称
          this.columnDrop2()
      } 
    },
    mounted() {
      // 获取表格组件实例
      const tableRef = this.$refs.VxeTableGrid
      for (const key in tableRef) {
        // 将 vxe-grid 组件中的方法添加到当前封装的表格方法中
        // 让调用当前表格组件的父组件可以通过实例直接访问 vxe-grid 组件中的方法
        if (
          Object.hasOwnProperty.call(tableRef, key) && 
          typeof tableRef[key] === 'function' && 
          !Object.hasOwnProperty.call(this, key)
        ) {
          this[key] = tableRef[key]
        }
      }
    },
    beforeDestroy () {
      if (this.sortable2 && this.layoutPath) {
        this.sortable2.destroy()
      }
    },
    methods:{
      //数据总和点击
      countHandle(){
        this.loadingPage = true;
        this.$emit('countTotal');
      },
      columnDrop2  () {
        this.$nextTick(() => {
          const $table = this.$refs.VxeTableGrid
          this.sortable2 = Sortable.create($table.$el.querySelector('.body--wrapper>.vxe-table--header .vxe-header--row'), {
            handle: '.vxe-header--column:not(.col--fixed)',
            onEnd: ({ item, newIndex, oldIndex }) => { 
              const { fullColumn, tableColumn } = $table.getTableColumn() 
              const targetThElem = item
              const wrapperElem = targetThElem.parentNode
              const newColumn = fullColumn[newIndex]
              if (newColumn.fixed) {
                // 错误的移动
                if (newIndex > oldIndex) {
                  wrapperElem.insertBefore(targetThElem, wrapperElem.children[oldIndex])
                } else {
                  wrapperElem.insertBefore(wrapperElem.children[oldIndex], targetThElem)
                }
                return this.$XModal.message({ content: '固定列不允许拖动！', status: 'error' })
              }
              // 转换真实索引
              const oldColumnIndex = $table.getColumnIndex(tableColumn[oldIndex])
              const newColumnIndex = $table.getColumnIndex(tableColumn[newIndex])
              // 移动到目标列
              const currRow = fullColumn.splice(oldColumnIndex, 1)[0]
              fullColumn.splice(newColumnIndex, 0, currRow)
              $table.loadColumn(fullColumn)

              //处理拖动后的请求
              let defaultColumn = [];
              this.$XEUtils.arrayEach(fullColumn, (contents)=>{ //拖动后的表头数据
                this.$XEUtils.arrayEach(this.defaultColumnArr, (item)=>{  //表格表头现有数据
                  if(item['field'] === contents['property']){ 
                      defaultColumn.push({
                        field: item.field, showStatus:item.showStatus,
                        lockStatus: item.lockStatus, title: item.title
                      });
                  }
                });
              }); 
              this.$emit('tableColumnStatue', defaultColumn)
            }
          })
        }) 
      },
      checkboxRangeStart(){},
      checkboxRangeChange(){},
      cellClickEvent({ row, rowIndex , $event }){
        let { altKey, ctrlKey, shiftKey } = $event;
        if(ctrlKey || shiftKey){
          if(this.selectedStart === null){ this.selectedStart = rowIndex; }
          let betweenArray = [ this.selectedStart , rowIndex ];
          let { tableData } = this.$refs['VxeTableGrid'].getTableData();
          if(ctrlKey && !shiftKey){ ///按住ctrl键是选中
            $XEUtils.set(row, 'RowIndex', rowIndex);
            let findKey = $XEUtils.findKey(this.keyCodeSelected, item => {
               return item.RowIndex === rowIndex;
            });
            if(findKey === undefined || findKey === null){
              this.keyCodeSelected.push(row);
            } else {
              $XEUtils.remove(this.keyCodeSelected, item => item.RowIndex === rowIndex);
            }
          }
          ///按住shift时选中，以及同时按住ctrl键和shift键时选中
          if(!ctrlKey && shiftKey || (ctrlKey && shiftKey)){
            this.keyCodeSelected = $XEUtils.filter(tableData,(item , index) =>{
              return (index >= Math.min.apply(Math, betweenArray)
                && index <= Math.max.apply(Math, betweenArray));
            });
          }
          this.$refs['VxeTableGrid'].clearCheckboxRow();
          this.$refs['VxeTableGrid'].setCheckboxRow(this.keyCodeSelected, true);
          this.selectedChange({ records: this.keyCodeSelected });
        }
      },
      handlePageChange({ currentPage, pageSize }) { // 分页
        // this.$emit('pageChange' , {pageNum: pageEvent.currentPage, ...pageEvent});
        let obj = { currentPage, pageSize, pageNum: currentPage }
        if(
          (this.pagerConfig.pageSize === this.dataList.length && this.pagerConfig.pageSize === pageSize) 
          || currentPage <= this.pagerConfig.currentPage
        ) {
          this.$emit('pageChange' , obj);
        }  
        if(this.pagerConfig.pageSize !== pageSize){ 
          obj.currentPage = 1; 
          obj.pageNum = 1; 
          this.$emit('pageChange' , obj);
        } 
      },
      currentChange(Data) {
        this.$refs.VxeTableGrid.setRadioRow(Data.row) 
        let { row , $event , rowIndex } = Data;
        let { altKey, ctrlKey, shiftKey } = $event;
        ///当有有按住ctrl键或者shift键时选中不处理逻辑
        if(ctrlKey || shiftKey){ return false;}
        this.selectedStart = rowIndex;
        $XEUtils.set(row, 'RowIndex', rowIndex);
        this.keyCodeSelected = [row];
        this.$refs['VxeTableGrid'].clearCheckboxRow()
        this.$refs['VxeTableGrid'].toggleCheckboxRow(row);
        this.selectedChange({ records: [row] });
      },
      selectedChange({ records }){
        this.$emit('selectedChange' , {
          selectedList: records,
          currentItem: records[records.length - 1] || {}
        });
      },
      radioChange({ row }) { 
        this.$emit('selectedChange' , {
          selectedList: [row],
          currentItem: row
        });
      },
      sortChange({column, property, order, sortBy, sortList, $event}){
        this.$emit('sortChange',property,order); 
      },
      saveCurrentLayout(){  //保存
        this.showColumnModel = false; 
        this.$emit('tableColumnStatue',this.fieldColumnData)
      },
      ColumnModelCancel(){
        this.showColumnModel = false;
      },
      ColumnModelResetDefault(){  //恢复系统默认布局
        this.$XModal.confirm('列布局是否重置为系统默认？').then(type => {
          this.showColumnModel = false; let defaultColumn = [];
          this.$XEUtils.arrayEach(this.defaultColumnArr, (item)=>{
            if(item['field']){
                defaultColumn.push({
                  field: item.field, showStatus:true,
                  lockStatus: false, title: item.title
                });
            }
          });
          this.$emit('tableColumnStatue')
        })
      },
      setTableColumnLayout(){ //读取默认列表排序
        this.showColumnModel = true;
        this.fieldColumnData = [];
        let defaultColumn = this.defaultColumnArr ? this.defaultColumnArr : this.tableColumn;
          this.$XEUtils.arrayEach(defaultColumn, (item)=>{
            if(item['field']){
                this.fieldColumnData.push({
                  field: item.field, showStatus:item.showStatus,
                  lockStatus: item.lockStatus, title: item.title,
                  width:item.width
                });
            }
          });
      },
      locationChange(row,changeName){  //位置改变：上一步：previousLine 下一步：nextStep
        let newFieldColumnData = [];
        let newNextItem = {};
        let newNextIndex = '';
        this.$XEUtils.arrayEach(this.fieldColumnData, (item,index)=>{
          switch(changeName){
            case "previousLine":
                if(row.field === item['field'] && index !== 0){
                  newFieldColumnData.splice(index-1,0,item);
                } else {
                  newFieldColumnData.push(item);
                }
              break;
            case "nextStep":
                if(row.field === item['field'] && index !== this.fieldColumnData.length){
                  newNextItem = item
                  newNextIndex = index+1;
                } else {
                  newFieldColumnData.push(item);
                }
              break;
            case "top":
                if(row.field === item['field'] && index !== 0){
                  newFieldColumnData.splice(0,0,item);
                } else {
                  newFieldColumnData.push(item);
                }
              break;
            case "bottom":
                if(row.field === item['field'] && index !== this.fieldColumnData.length){
                  newFieldColumnData.push(item);
                } else {
                  newFieldColumnData.splice(newFieldColumnData.length - 1,0,item);
                }
              break;
          }
        });
        if(changeName === 'nextStep'){ newFieldColumnData.splice(newNextIndex,0,newNextItem); }
        this.fieldColumnData = newFieldColumnData;
      },
      rowEditClosedEvent(editRow){
        let { row , column } = editRow;
        if (this.$refs.VxeTableGrid.isUpdateByRow(row, column.property)) {
          this.$emit('editClosedEvent' , editRow);
        }
      },
      handlePutAway(){ //收缩
        this.shrinkStatus = !this.shrinkStatus;
        this.$emit('shrinkstat',this.shrinkStatus);
      },
      rowEditActived(){

      },
      ColumnLockTaggle(row){
        row.lockStatus = !row.lockStatus;
      }
    }
  }
</script>
<style>
  .toggle-lock{
    margin-top: 0;
    margin-bottom: 0;
    width: 18px;
    height: 18px;
    border-radius: 0;
    background: url("~@/assets/icon-lock.png") 0 0 no-repeat;
    -webkit-transition: background .1s linear .1s;
    transition: background .1s linear .1s;
    display: inline-block;
    cursor: pointer;
    text-indent: -200px;
  }
  .toggle-lock.off {
    background-position: 0 -18px;
  }
  .vxeGridComponent .mytable-style .vxe-table--fixed-left-wrapper
  .vxeGridComponent .vxe-table--body-wrapper.fixed-left--wrapper
  .vxeGridComponent .vxe-table--body tbody .col--ellipsis{
    background-color: white !important;
  }
  .vxeGridComponent .self_el-cascader-panel .el-cascader-panel.is-bordered{
    background-color: white !important;
    position: fixed !important;
    margin:4px 0 0 10px;
  }
  .operationAction {
    font-size: 20px;
    color: #4cb2ff;
    margin: 0 1px;
    font-weight: bold;
    cursor: pointer;
  }
  .vxeGridComponent .pageOne{
    min-width:60px
  }
  .vxeGridComponent .pageOne span{
    min-width:34px;
    display:inline-block
  }
  .vxeGridComponent .pageTwo span{
    min-width:20px;
    display:inline-block
  }
</style>
<style scoped>
  .vxeGridComponent {
    width: 100%;
    height: 100%;
  }
  .pager-container {
    height: 36px;
  }
  .putAwayBtn{
    color: #fff;
    margin-left: 20px;
    background-color: #67c23a;
    border-color: #67c23a;
  }
  .pageCount{
    display: inline-block;
    border: 1px solid #e4e4e4; 
    padding: 0 6px;
    min-width: 10px;
    border-radius: 4px;
    margin: 0 6px;
  }
</style>
