<template>
  <div>
    <vxe-toolbar
      custom
      :loading="loading"
      :refresh="{query: loadList}">
      <template v-slot:buttons>
        <vxe-input clearable class="search-input" v-model="filterName" type="search" :placeholder="`vxe-${apiName} ${$t('app.api.apiSearch')}`" @keyup="searchEvent" @clear="searchEvent"></vxe-input>
      </template>
    </vxe-toolbar>

    <vxe-table
      resizable
      auto-resize
      highlight-current-row
      highlight-hover-row
      highlight-current-column
      ref="xTable"
      id="document_api"
      class="api-table"
      row-id="id"
      :loading="loading"
      :cell-class-name="cellClassNameFunc"
      :data="apiList"
      :custom-config="{storage: true, checkMethod: checkColumnMethod}"
      :tree-config="{children: 'list', expandRowKeys: defaultExpandRowKeys, trigger: 'cell'}"
      :context-menu="{header: {options: headerMenus}, body: {options: bodyMenus}, visibleMethod: menuVisibleMethod}"
      @header-cell-context-menu="headerCellContextMenuEvent"
      @cell-context-menu="cellContextMenuEvent"
      @context-menu-click="contextMenuClickEvent">
      <vxe-table-column field="name" title="app.api.title.prop" type="html" min-width="280" :filters="nameFilters" tree-node></vxe-table-column>
      <vxe-table-column field="desc" title="app.api.title.desc" type="html" min-width="200"></vxe-table-column>
      <vxe-table-column field="type" title="app.api.title.type" type="html" min-width="140"></vxe-table-column>
      <vxe-table-column field="enum" :title="$t('app.api.title.enum')" type="html" min-width="150"></vxe-table-column>
      <vxe-table-column field="defVal" :title="$t('app.api.title.defVal')" type="html" min-width="160"></vxe-table-column>
      <vxe-table-column field="version" :title="$t('app.api.title.version')" width="120">
        <template v-slot="{ row }">
          <span v-show="row.version" class="compatibility">v{{  row.version }}</span>
        </template>
      </vxe-table-column>
      <template v-slot:empty>
        <span class="red">找不对应 API，请输入正确的关键字！</span>
      </template>
    </vxe-table>
  </div>
</template>

<script>
import XEUtils from 'xe-utils'
import pack from '../../../package.json'
import XEClipboard from 'xe-clipboard'
import tableAPI from '../../api/table'
import tableColumnAPI from '../../api/column'
import toolbarAPI from '../../api/toolbar'
import gridAPI from '../../api/grid'
import virtualTreeAPI from '../../api/virtual-tree'
import excelAPI from '../../api/excel'
import pagerAPI from '../../api/pager'
import radioAPI from '../../api/radio'
import checkboxAPI from '../../api/checkbox'
import inputAPI from '../../api/input'
import selectAPI from '../../api/select'
import optgroupAPI from '../../api/optgroup'
import optionAPI from '../../api/option'
import textareaAPI from '../../api/textarea'
import buttonAPI from '../../api/button'
import tooltipAPI from '../../api/tooltip'
import modalAPI from '../../api/modal'
import formAPI from '../../api/form'
import formItemAPI from '../../api/form-item'
import switchAPI from '../../api/switch'
import listAPI from '../../api/list'
import pulldownAPI from '../../api/pulldown'

// import i18n from '../../i18n'
// const attributes = window.attributes = {}
// const tags = window.tags = {}

// const tagMaps = [
//   ['vxe-table', tableAPI, { subtags: ['vxe-table-column'], description: '基础表格' }],
//   ['vxe-table-column', tableColumnAPI, { description: '基础表格 - 列' }],
//   ['vxe-grid', gridAPI, { description: '高级表格' }],
//   ['vxe-toolbar', toolbarAPI, { description: '工具栏' }],
//   ['vxe-pager', pagerAPI, { description: '分页' }],
//   ['vxe-radio', radioAPI, { description: '单选框' }],
//   ['vxe-checkbox', checkboxAPI, { description: '复选框' }],
//   ['vxe-switch', switchAPI, { description: '开关按钮' }],
//   ['vxe-input', inputAPI, { description: '输入框' }],
//   ['vxe-select', selectAPI, { subtags: ['vxe-optgroup', 'vxe-option'], description: '下拉框' }],
//   ['vxe-optgroup', optgroupAPI, { subtags: ['vxe-option'], description: '下拉框 - 分组' }],
//   ['vxe-option', optionAPI, { description: '下拉框 - 选项' }],
//   ['vxe-button', buttonAPI, { description: '按钮' }],
//   ['vxe-tooltip', tooltipAPI, { description: '工具提示' }],
//   ['vxe-modal', modalAPI, { description: '模态窗口' }],
//   ['vxe-form', formAPI, { subtags: ['vxe-form-item'], description: '表单' }],
//   ['vxe-form-item', formItemAPI, { description: '表单 - 项' }],
//   ['vxe-list', listAPI, { description: '列表' }],
//   ['vxe-pulldown', pulldownAPI, { description: '下拉容器' }]
// ]

// tagMaps.forEach(confs => {
//   const props = confs[1].find(item => item.name === 'Props').list
//   const keys = []
//   props.forEach(item => {
//     const name = XEUtils.kebabCase(item.name)
//     attributes[`${confs[0]}/${name}`] = {
//       type: XEUtils.toString(item.type).toLowerCase(),
//       description: item.descKey ? i18n.t(item.descKey) : item.desc
//     }
//     keys.push(name)
//   })
//   tags[confs[0]] = Object.assign({ attributes: keys }, confs[2])
// })

export default {
  data () {
    return {
      filterName: this.$route.query.filterName ? decodeURIComponent(this.$route.query.filterName) : '',
      apiList: [],
      defaultExpandRowKeys: [],
      loading: false,
      tableData: [],
      nameFilters: [
        { label: 'Props', value: 'Props' },
        { label: 'Slots', value: 'Slots' },
        { label: 'Events', value: 'Events' },
        { label: 'Methods', value: 'Methods' }
      ],
      headerMenus: [
        [
          {
            code: 'hideColumn',
            name: '隐藏列',
            disabled: false
          },
          {
            code: 'showAllColumn',
            name: '取消所有隐藏列'
          },
          {
            code: 'resetColumn',
            name: '重置个性化数据'
          }
        ],
        [
          {
            code: 'exportXLSXAPI',
            name: '导出文档.xlsx',
            prefixIcon: 'fa fa-download'
          }
        ]
      ],
      bodyMenus: [
        [
          {
            code: 'copy',
            name: 'app.body.label.copy',
            prefixIcon: 'fa fa-copy'
          }
        ],
        [
          {
            code: 'resize',
            name: '重新加载'
          },
          {
            code: 'exportHTMLAPI',
            name: '导出文档.html',
            prefixIcon: 'fa fa-download'
          },
          {
            code: 'exportXLSXAPI',
            name: '导出文档.xlsx',
            prefixIcon: 'fa fa-download'
          }
        ],
        [
          {
            code: 'allExpand',
            name: '全部展开'
          },
          {
            code: 'allShrink',
            name: '全部收起'
          }
        ]
      ]
    }
  },
  computed: {
    apiName () {
      return this.$route.params.name
    }
  },
  watch: {
    apiName () {
      this.loadList()
    },
    '$i18n.locale' () {
      this.loadList()
    }
  },
  created () {
    this.loadList()
  },
  methods: {
    loadList () {
      this.loading = true
      return new Promise(resolve => {
        setTimeout(() => {
          let apis = []
          switch (this.$route.params.name) {
            case 'table':
              apis = tableAPI
              break
            case 'table-column':
              apis = tableColumnAPI
              break
            case 'toolbar':
              apis = toolbarAPI
              break
            case 'grid':
              apis = gridAPI
              break
            case 'virtual-tree':
              apis = virtualTreeAPI
              break
            case 'excel':
              apis = excelAPI
              break
            case 'pager':
              apis = pagerAPI
              break
            case 'radio':
              apis = radioAPI
              break
            case 'checkbox':
              apis = checkboxAPI
              break
            case 'input':
              apis = inputAPI
              break
            case 'textarea':
              apis = textareaAPI
              break
            case 'select':
              apis = selectAPI
              break
            case 'optgroup':
              apis = optgroupAPI
              break
            case 'option':
              apis = optionAPI
              break
            case 'button':
              apis = buttonAPI
              break
            case 'tooltip':
              apis = tooltipAPI
              break
            case 'modal':
              apis = modalAPI
              break
            case 'form':
              apis = formAPI
              break
            case 'form-item':
              apis = formItemAPI
              break
            case 'switch':
              apis = switchAPI
              break
            case 'list':
              apis = listAPI
              break
            case 'pulldown':
              apis = pulldownAPI
              break
          }
          // 生成唯一 id
          let index = 1
          const searchProps = ['name', 'desc', 'type', 'enum', 'defVal']
          this.tableData = XEUtils.clone(apis, true)
          XEUtils.eachTree(this.tableData, item => {
            item.id = index++
            item.desc = item.descKey ? this.$t(item.descKey) : item.desc
            searchProps.forEach(key => {
              item[key] = XEUtils.escape(item[key])
            })
          }, { children: 'list' })
          // 默认展开一级
          this.defaultExpandRows = this.tableData.filter(item => item.list && item.list.length)
          this.defaultExpandRowKeys = this.defaultExpandRows.map(item => item.id)
          this.loading = false
          this.handleSearch()
          resolve()
        }, 100)
      })
    },
    cellClassNameFunc ({ row, column }) {
      return {
        'api-disabled': row.disabled,
        'api-abandoned': row.abandoned,
        'disabled-line-through': (row.disabled || row.abandoned) && column.property === 'name'
      }
    },
    checkColumnMethod ({ column }) {
      if (['name', 'desc'].includes(column.property)) {
        return false
      }
      return true
    },
    headerCellContextMenuEvent ({ column }) {
      this.$refs.xTable.setCurrentColumn(column)
    },
    cellContextMenuEvent ({ row }) {
      this.$refs.xTable.setCurrentRow(row)
    },
    contextMenuClickEvent ({ menu, row, column }) {
      const xTable = this.$refs.xTable
      switch (menu.code) {
        case 'hideColumn':
          xTable.hideColumn(column)
          break
        case 'showAllColumn':
          xTable.resetColumn({ visible: true })
          break
        case 'resetColumn':
          xTable.resetColumn(true)
          break
        case 'exportHTMLAPI':
          xTable.exportData({
            type: 'html',
            data: XEUtils.toTreeArray(this.tableData, { children: 'list' }),
            filename: `vxe-${this.apiName}_v${pack.version}`
          })
          break
        case 'exportXLSXAPI':
          xTable.exportData({
            type: 'xlsx',
            data: XEUtils.toTreeArray(this.tableData, { children: 'list' }),
            filename: `vxe-${this.apiName}_v${pack.version}`
          })
          break
        case 'copy':
          if (row && column) {
            if (XEClipboard.copy(row[column.property])) {
              this.$XModal.message({ message: this.$t('app.body.msg.copyToClipboard'), status: 'success' })
            }
          }
          break
        case 'resize':
          this.filterName = ''
          this.tableData = []
          this.$nextTick(() => {
            xTable.clearAll()
            this.loadList()
          })
          break
        case 'exportAPI':
          xTable.exportData({
            filename: `vxe-${this.apiName}_v${pack.version}.csv`
          })
          break
        case 'allExpand':
          xTable.setAllTreeExpand(true)
          break
        case 'allShrink':
          xTable.clearTreeExpand()
          break
      }
    },
    menuVisibleMethod ({ options, column }) {
      const isDisabled = !this.checkColumnMethod({ column })
      options.forEach(list => {
        list.forEach(item => {
          if (['hideColumn'].includes(item.code)) {
            item.disabled = isDisabled
          }
        })
      })
      return true
    },
    handleSearch () {
      const filterName = XEUtils.toString(this.filterName).trim().toLowerCase()
      if (filterName) {
        const filterRE = new RegExp(filterName, 'gi')
        const options = { children: 'list' }
        const searchProps = ['name', 'desc', 'type', 'enum', 'defVal']
        const rest = XEUtils.searchTree(this.tableData, item => searchProps.some(key => item[key].toLowerCase().indexOf(filterName) > -1), options)
        XEUtils.eachTree(rest, item => {
          searchProps.forEach(key => {
            item[key] = item[key].replace(filterRE, match => `<span class="keyword-lighten">${match}</span>`)
          })
        }, options)
        this.apiList = rest
        this.$nextTick(() => {
          if (this.$refs.xTable) {
            this.$refs.xTable.setAllTreeExpand(true)
          }
        })
      } else {
        this.apiList = this.tableData
        this.$nextTick(() => {
          if (this.$refs.xTable) {
            this.$refs.xTable.setTreeExpand(this.defaultExpandRows, true)
          }
        })
      }
    },
    // 调用频率间隔 500 毫秒
    searchEvent: XEUtils.debounce(function () {
      this.handleSearch()
    }, 500, { leading: false, trailing: true })
  },
  beforeRouteUpdate (to, from, next) {
    next()
    this.filterName = ''
    if (this.$refs.xTable) {
      this.$refs.xTable.clearAll()
    }
    this.handleSearch()
  }
}
</script>
