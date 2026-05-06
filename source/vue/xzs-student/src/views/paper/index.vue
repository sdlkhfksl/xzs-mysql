<template>
  <div style="margin-top: 10px" class="app-contain">
    <el-tabs tab-position="left" v-model="tabId" @tab-click="subjectChange">
      <el-tab-pane :label="item.name" :key="item.id" :name="item.id" v-for="item in subjectList" style="margin-left: 20px;">
        <el-row style="float: right">
          <el-radio-group v-model="queryParam.paperType" size="mini" @change="paperTypeChange">
            <el-radio v-for="item in paperTypeEnum" size="mini" :key="item.key" :label="item.key">{{item.value}}</el-radio>
          </el-radio-group>
        </el-row>
        <el-table v-loading="listLoading" :data="sortedTableData" fit highlight-current-row style="width: 100%">
          <el-table-column prop="id" label="序号" width="90px"/>
          <el-table-column prop="name" label="名称"/>
          <el-table-column label="收藏" width="60px" align="center">
            <template slot-scope="{row}">
              <el-button type="text" size="small" @click="handleToggleFavorite(row)" :class="{'is-favorite': isPaperFavorite(row.id)}">
                <svg-icon :icon-class="isPaperFavorite(row.id) ? 'star' : 'star'" :class="{'favorite-icon': true, 'active': isPaperFavorite(row.id)}"/>
              </el-button>
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template slot-scope="{row}">
              <router-link target="_blank" :to="{path:'/do',query:{id:row.id}}">
                <el-button type="text" size="small">开始答题</el-button>
              </router-link>
            </template>
          </el-table-column>
        </el-table>
        <pagination v-show="total>0" :total="total" :background="false" :page.sync="queryParam.pageIndex" :limit.sync="queryParam.pageSize"
                    @pagination="search" style="margin-top: 20px"/>
      </el-tab-pane>
    </el-tabs>

  </div>
</template>

<script>
import { mapState } from 'vuex'
import Pagination from '@/components/Pagination'
import examPaperApi from '@/api/examPaper'
import subjectApi from '@/api/subject'
import { isFavorite, toggleFavorite, sortByFavorite } from '@/utils/favorite'

export default {
  components: { Pagination },
  data () {
    return {
      queryParam: {
        paperType: 1,
        subjectId: 0,
        pageIndex: 1,
        pageSize: 10
      },
      tabId: '',
      listLoading: true,
      subjectList: [],
      tableData: [],
      total: 0
    }
  },
  computed: {
    ...mapState('enumItem', {
      paperTypeEnum: state => state.exam.examPaper.paperTypeEnum
    }),
    sortedTableData () {
      return sortByFavorite(this.tableData)
    }
  },
  created () {
    this.initSubject()
  },
  methods: {
    initSubject () {
      let _this = this
      subjectApi.list().then(re => {
        _this.subjectList = re.response
        let subjectId = _this.subjectList[0].id
        _this.queryParam.subjectId = subjectId
        _this.tabId = subjectId.toString()
        _this.search()
      })
    },
    search () {
      this.listLoading = true
      examPaperApi.pageList(this.queryParam).then(data => {
        const re = data.response
        this.tableData = re.list
        this.total = re.total
        this.queryParam.pageIndex = re.pageNum
        this.listLoading = false
      })
    },
    paperTypeChange (val) {
      this.search()
    },
    subjectChange (tab, event) {
      this.queryParam.subjectId = Number(this.tabId)
      this.search()
    },
    isPaperFavorite (paperId) {
      return isFavorite(paperId)
    },
    handleToggleFavorite (row) {
      const result = toggleFavorite(row.id, {
        name: row.name,
        subjectId: row.subjectId
      })
      if (result) {
        this.$message({
          message: this.isPaperFavorite(row.id) ? '收藏成功' : '已取消收藏',
          type: 'success',
          duration: 1500
        })
      } else {
        this.$message.error('操作失败，请稍后重试')
      }
    }
  }
}
</script>

<style scoped>
.favorite-icon {
  width: 18px;
  height: 18px;
  fill: #c0c4cc;
  transition: all 0.3s;
  cursor: pointer;
}

.favorite-icon:hover {
  fill: #409eff;
  transform: scale(1.2);
}

.favorite-icon.active {
  fill: #f7ba2a;
}

.is-favorite .favorite-icon {
  fill: #f7ba2a;
}
</style>
