<template>
  <div class="knowledge-container">
    <!-- 左侧知识库列表 -->
    <div class="library-list">
      <div
        v-for="library in libraries"
        :key="library.id"
        class="library-item"
        :class="{ 'active': selectedLibrary === library.id }"
        @click="selectLibrary(library.id)"
      >
        {{ library.name }}
        <div class="file-count">{{ library.files.length }} files</div>
      </div>

      <el-button
        type="primary"
        class="new-library-btn"
        @click="showDialog = true"
      >
        新建知识库
      </el-button>
    </div>

    <!-- 右侧文件管理 -->
    <div class="file-management">
      <template v-if="selectedLibrary">
        <div class="upload-section">
          <input
            ref="fileInput"
            type="file"
            multiple
            style="display: none"
            @change="handleFileUpload"
          >
          <el-button
            type="primary"
            @click="$refs.fileInput.click()"
          >
            上传文件
          </el-button>

          <el-checkbox v-model="readImage" class="read-image-checkbox">
            读取图片
          </el-checkbox>
        </div>

        <div class="file-list">
          <div
            v-for="file in currentLibrary.files"
            :key="file.id"
            class="file-item"
          >
            <div class="file-name">{{ file.name }}</div>
            <div class="file-meta">
              {{ formatDate(file.uploadTime) }} -
              {{ formatFileSize(file.size) }}
            </div>
          </div>
        </div>
      </template>

      <div v-else class="empty-state">
        请选择或创建一个知识库
      </div>
    </div>

    <!-- 新建知识库对话框 -->
    <el-dialog
      title="新建知识库"
      :visible.sync="showDialog"
      width="30%"
    >
      <el-form>
        <el-form-item label="知识库名称">
          <el-input v-model="newLibraryName" />
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="showDialog = false">取消</el-button>
        <el-button type="primary" @click="createLibrary">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      libraries: [
        {
          id: 1,
          name: '默认知识库',
          files: [
            { id: 1, name: '文档1.pdf', uploadTime: new Date(), size: 1024 },
            { id: 2, name: '图片1.jpg', uploadTime: new Date(), size: 2048 }
          ]
        }
      ],
      selectedLibrary: null,
      showDialog: false,
      newLibraryName: '',
      readImage: false
    }
  },

  computed: {
    currentLibrary() {
      return this.libraries.find(lib => lib.id === this.selectedLibrary)
    }
  },

  methods: {
    selectLibrary(id) {
      this.selectedLibrary = id
    },

    createLibrary() {
      if (!this.newLibraryName.trim()) return

      this.libraries.push({
        id: Date.now(),
        name: this.newLibraryName,
        files: []
      })

      this.newLibraryName = ''
      this.showDialog = false
    },

    async handleFileUpload(e) {
      const files = Array.from(e.target.files)

      for (const file of files) {
        await this.uploadFile(file)
      }
    },

    uploadFile(file) {
      return new Promise(resolve => {
        // 模拟上传过程
        setTimeout(() => {
          this.currentLibrary.files.push({
            id: Date.now(),
            name: file.name,
            uploadTime: new Date(),
            size: file.size
          })
          resolve()
        }, 1000)
      })
    },

    formatDate(date) {
      return date.toLocaleDateString()
    },

    formatFileSize(bytes) {
      if (bytes === 0) return '0 B'
      const k = 1024
      const sizes = ['B', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }
  }
}
</script>

  <style scoped>
  .knowledge-container {
    display: flex;
    height: 100vh;
  }

  .library-list {
    width: 250px;
    border-right: 1px solid #ebeef5;
    padding: 20px;
    overflow-y: auto;
  }

  .library-item {
    padding: 12px;
    margin-bottom: 8px;
    cursor: pointer;
    border-radius: 4px;
    transition: background-color 0.3s;
  }

  .library-item:hover {
    background-color: #f5f7fa;
  }

  .library-item.active {
    background-color: #409eff;
    color: white;
  }

  .file-count {
    font-size: 0.8em;
    opacity: 0.7;
  }

  .new-library-btn {
    width: 100%;
    margin-top: 20px;
  }

  .file-management {
    flex: 1;
    padding: 20px;
  }

  .upload-section {
    margin-bottom: 20px;
    display: flex;
    gap: 15px;
    align-items: center;
  }

  .file-list {
    max-height: calc(100vh - 160px);
    overflow-y: auto;
  }

  .file-item {
    padding: 12px;
    border-bottom: 1px solid #ebeef5;
  }

  .file-name {
    font-weight: 500;
  }

  .file-meta {
    font-size: 0.8em;
    color: #909399;
  }

  .empty-state {
    display: flex;
    height: 100%;
    align-items: center;
    justify-content: center;
    color: #909399;
  }
  </style>
