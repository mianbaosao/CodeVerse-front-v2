<template>
  <div class="min-h-screen bg-gray-50">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <!-- 成功提示优化 -->
      <div v-if="showSuccess" 
        class="fixed top-4 right-4 bg-green-100 border-l-4 border-green-500 text-green-700 px-6 py-4 rounded-r shadow-lg flex items-center space-x-3 z-50 animate-fade-in"
      >
        <i class="fas fa-check-circle text-xl"></i>
        <span class="font-medium">{{ successMessage }}</span>
      </div>

      <div class="flex space-x-6">
        <!-- 左侧导航和信息区域 -->
        <div class="w-72 flex-shrink-0 space-y-6">
          <!-- 圈子列表 -->
          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
            <div class="p-6 border-b border-gray-100">
              <div class="flex justify-between items-center">
                <h2 class="text-xl font-semibold text-gray-800">圈子列表</h2>
                <button 
                  @click="showAddCircleModal = true"
                  class="p-2 text-indigo-600 hover:bg-indigo-50 rounded-lg transition-all duration-200 transform hover:scale-110"
                  title="新增圈子"
                >
                  <i class="fas fa-plus"></i>
                </button>
              </div>
            </div>
            
            <!-- 圈子列表样式优化 -->
            <div class="p-4 space-y-3">
              <template v-for="circle in circles" :key="circle.id">
                <!-- 主圈子 -->
                <div class="rounded-lg bg-gray-50 p-3">
                  <div class="font-medium text-gray-800 flex items-center justify-between mb-2">
                    <span class="flex items-center">
                      <i class="fas fa-folder-open text-indigo-500 mr-2"></i>
                      {{ circle.circleName }}
                    </span>
                    <div class="flex items-center space-x-2">
                      <button 
                        @click="showAddSubCircle(circle.id)"
                        class="p-1.5 text-xs text-gray-500 hover:text-indigo-600 hover:bg-white rounded transition-all"
                      >
                        <i class="fas fa-plus-circle"></i>
                      </button>
                      <button 
                        @click.stop="showEditCircle(circle)"
                        class="p-1.5 text-xs text-gray-500 hover:text-blue-600 hover:bg-white rounded transition-all"
                      >
                        <i class="fas fa-edit"></i>
                      </button>
                      <button 
                        @click.stop="confirmDeleteCircle(circle.id)"
                        class="p-1.5 text-xs text-gray-500 hover:text-red-600 hover:bg-white rounded transition-all"
                      >
                        <i class="fas fa-trash"></i>
                      </button>
                    </div>
                  </div>
                  
                  <!-- 子圈子 -->
                  <div class="space-y-1">
                    <div v-for="subCircle in circle.children" 
                      :key="subCircle.id"
                      class="group flex items-center justify-between p-2 rounded-lg cursor-pointer transition-all hover:bg-white"
                    >
                      <div class="flex items-center flex-1" @click="selectCircle(subCircle.id)">
                        <div class="w-8 h-8 rounded-lg bg-white shadow-sm p-1 mr-3">
                          <img v-if="subCircle.icon" :src="subCircle.icon" class="w-full h-full object-cover rounded">
                          <i v-else class="fas fa-circle text-gray-400"></i>
                        </div>
                        <span class="text-gray-700 group-hover:text-indigo-600">{{ subCircle.circleName }}</span>
                      </div>
                      <div class="flex items-center space-x-2 opacity-0 group-hover:opacity-100 transition-opacity">
                        <button 
                          @click.stop="showEditCircle(subCircle)"
                          class="p-1.5 text-xs text-gray-400 hover:text-blue-600 transition-all"
                        >
                          <i class="fas fa-edit"></i>
                        </button>
                        <button 
                          @click.stop="confirmDeleteCircle(subCircle.id)"
                          class="p-1.5 text-xs text-gray-400 hover:text-red-600 transition-all"
                        >
                          <i class="fas fa-trash"></i>
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </template>
            </div>
          </div>

          <!-- 圈子信息卡片 -->
          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
            <h3 class="text-lg font-semibold text-gray-800 mb-4">圈子信息</h3>
            <div class="space-y-4">
              <div class="flex items-center space-x-3">
                <div class="w-12 h-12 rounded-lg bg-indigo-50 flex items-center justify-center">
                  <i class="fas fa-users text-indigo-500 text-xl"></i>
                </div>
                <div>
                  <div class="text-sm text-gray-500">成员数</div>
                  <div class="text-lg font-medium text-gray-800">{{ memberCount || 0 }}</div>
                </div>
              </div>
              <div class="flex items-center space-x-3">
                <div class="w-12 h-12 rounded-lg bg-green-50 flex items-center justify-center">
                  <i class="fas fa-comment-dots text-green-500 text-xl"></i>
                </div>
                <div>
                  <div class="text-sm text-gray-500">今日动态</div>
                  <div class="text-lg font-medium text-gray-800">{{ todayPosts || 0 }}</div>
                </div>
              </div>
            </div>
          </div>

          <!-- 热门话题 -->
          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
            <h3 class="text-lg font-semibold text-gray-800 mb-4">热门话题</h3>
            <div class="space-y-3">
              <div v-for="(topic, index) in hotTopics" :key="index" 
                class="flex items-start space-x-3 p-3 rounded-lg hover:bg-gray-50 transition-colors cursor-pointer"
              >
                <div class="text-lg font-medium text-gray-400">#{{ index + 1 }}</div>
                <div>
                  <div class="text-gray-800 font-medium">{{ topic.title }}</div>
                  <div class="text-sm text-gray-500 mt-1">{{ topic.count }}个讨论</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 中间内容区域 -->
        <div class="flex-1">
          <!-- 发布框优化 -->
          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 mb-6">
            <textarea
              v-model="newMomentContent"
              rows="3"
              class="w-full px-4 py-3 text-gray-700 border border-gray-200 rounded-xl resize-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition-all"
              placeholder="分享你的想法..."
            ></textarea>
            
            <!-- 图片上传和发布按钮 -->
            <div class="mt-4 flex justify-between items-center">
              <div class="flex items-center space-x-4">
                <input
                  type="file"
                  ref="momentFileInput"
                  class="hidden"
                  accept="image/*"
                  @change="handleMomentImageUpload"
                >
                <button
                  @click="triggerMomentFileInput"
                  class="flex items-center space-x-2 px-4 py-2 text-gray-600 hover:text-indigo-600 hover:bg-indigo-50 rounded-lg transition-all"
                >
                  <i class="fas fa-image"></i>
                  <span>添加图片</span>
                </button>
              </div>
              <button
                @click="publishMoment"
                class="px-6 py-2.5 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-all disabled:opacity-50 disabled:cursor-not-allowed transform hover:scale-105"
                :disabled="!newMomentContent.trim() && !momentImages.length"
              >
                发布
              </button>
            </div>

            <!-- 图片预览优化 -->
            <div v-if="momentImages.length" class="mt-4 grid grid-cols-4 gap-4">
              <div v-for="(img, index) in momentImages" :key="index" class="relative group">
                <img :src="img" class="w-full h-24 object-cover rounded-lg shadow-sm">
                <div class="absolute inset-0 bg-black bg-opacity-40 opacity-0 group-hover:opacity-100 transition-opacity rounded-lg flex items-center justify-center">
                  <button
                    @click="removeMomentImage(index)"
                    class="p-2 text-white hover:text-red-400 transition-colors"
                  >
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- 内容列表优化 -->
          <div class="space-y-6">
            <div v-for="moment in moments" :key="moment.id" 
              class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden hover:shadow-md transition-shadow"
            >
              <div class="p-4">
                <div class="flex items-start justify-between">
                  <div class="flex-1">
                    <!-- 内容 -->
                    <p class="text-gray-700 mb-2">{{ moment.content }}</p>
                    <!-- 图片列表 -->
                    <div v-if="moment.picUrlList?.length" class="grid grid-cols-3 gap-2 mb-2">
                      <img v-for="(pic, index) in moment.picUrlList" 
                        :key="index"
                        :src="pic"
                        class="w-full h-32 object-cover rounded cursor-pointer hover:opacity-90"
                        @click="previewImage(pic)"
                      >
                    </div>
                    
                    <!-- 操作栏 -->
                    <div class="flex items-center space-x-6 text-gray-500 text-sm mt-4">
                      <!-- 点赞 -->
                      <button 
                        @click="toggleLike(moment)"
                        class="flex items-center space-x-1 hover:text-indigo-600 transition-colors"
                        :class="{ 'text-indigo-600': moment.isLiked }"
                      >
                        <i :class="[
                          'fas', 
                          moment.isLiked ? 'fa-heart' : 'fa-heart',
                          'transition-transform hover:scale-110'
                        ]"></i>
                        <span>{{ moment.likeCount || 0 }}</span>
                      </button>
                      
                      <!-- 评论按钮 -->
                      <button 
                        @click="showCommentInput(moment.id)"
                        class="flex items-center space-x-1 hover:text-indigo-600 transition-colors"
                      >
                        <i class="far fa-comment"></i>
                        <span>{{ moment.replyCount || 0 }}</span>
                      </button>
                    </div>

                    <!-- 评论输入框 -->
                    <div v-if="activeCommentId === moment.id" class="mt-4">
                      <div class="flex space-x-2">
                        <input 
                          v-model="commentContent"
                          type="text"
                          class="flex-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
                          placeholder="写下你的评论..."
                        >
                        <!-- 图片上传 -->
                        <input
                          type="file"
                          :ref="el => { if (el) fileInputs[moment.id] = el }"
                          class="hidden"
                          accept="image/*"
                          @change="handleImageUpload"
                        >
                        <button
                          @click="() => triggerFileInput(moment.id)"
                          class="px-3 py-2 text-gray-500 hover:text-indigo-600"
                          title="上传图片"
                        >
                          <i class="fas fa-image"></i>
                        </button>
                        <button
                          @click="submitComment(moment.id)"
                          class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors"
                          :disabled="!commentContent.trim()"
                        >
                          发送
                        </button>
                      </div>
                      <!-- 预览图片 -->
                      <div v-if="uploadedImages.length" class="mt-2 flex space-x-2">
                        <div v-for="(img, index) in uploadedImages" :key="index" class="relative">
                          <img :src="img" class="w-16 h-16 object-cover rounded">
                          <button
                            @click="removeImage(index)"
                            class="absolute -top-2 -right-2 w-5 h-5 bg-red-500 text-white rounded-full flex items-center justify-center text-xs"
                          >
                            ×
                          </button>
                        </div>
                      </div>
                    </div>

                    <!-- 评论列表 -->
                    <div v-if="moment.comments?.length" class="mt-4 space-y-3 bg-gray-50 p-4 rounded-lg">
                      <div v-for="comment in moment.comments" :key="comment.id" class="text-sm">
                        <div class="flex justify-between">
                          <div class="font-medium text-gray-900">{{ comment.userName }}</div>
                          <div class="text-gray-500">{{ formatDate(comment.createdTime) }}</div>
                        </div>
                        <div class="mt-1 text-gray-700">{{ comment.content }}</div>
                        <!-- 评论的图片 -->
                        <div v-if="comment.picUrlList?.length" class="mt-2 flex space-x-2">
                          <img 
                            v-for="(pic, index) in comment.picUrlList" 
                            :key="index"
                            :src="pic"
                            class="w-16 h-16 object-cover rounded cursor-pointer"
                            @click="previewImage(pic)"
                          >
                        </div>
                        <!-- 回复按钮 -->
                        <div class="mt-2 flex justify-end">
                          <button
                            @click="showReplyInput(comment.id)"
                            class="text-gray-500 hover:text-indigo-600 transition-colors text-xs"
                          >
                            回复
                          </button>
                        </div>
                        <!-- 回复输入框 -->
                        <div v-if="activeReplyId === comment.id" class="mt-2">
                          <div class="flex space-x-2">
                            <input 
                              v-model="replyContent"
                              type="text"
                              class="flex-1 px-3 py-2 border rounded-lg text-sm"
                              placeholder="回复..."
                            >
                            <button
                              @click="submitReply(moment.id, comment.id)"
                              class="px-3 py-1 bg-indigo-600 text-white rounded-lg text-sm"
                              :disabled="!replyContent.trim()"
                            >
                              回复
                            </button>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <!-- 管理员删除按钮 -->
                  <button 
                    v-if="isAdmin"
                    @click="confirmDeleteMoment(moment.id)"
                    class="p-2 text-gray-400 hover:text-red-600 transition-colors"
                    title="删除内容"
                  >
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- 分页 -->
          <div class="flex justify-center p-4">
            <div class="flex space-x-2">
              <button 
                v-for="page in totalPages" 
                :key="page"
                @click="changePage(page)"
                class="px-3 py-1 rounded"
                :class="[
                  currentPage === page 
                    ? 'bg-indigo-600 text-white' 
                    : 'text-gray-600 hover:bg-gray-100'
                ]"
              >
                {{ page }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 新增圈子弹窗 -->
    <div v-if="showAddCircleModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg w-96 p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-medium">{{ isSubCircle ? '新增子圈子' : '新增圈子' }}</h3>
          <button @click="closeAddCircleModal" class="text-gray-400 hover:text-gray-600">
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="space-y-4">
          <!-- 圈子图标上传 -->
          <div class="flex flex-col items-center">
            <div 
              class="w-20 h-20 rounded-full border-2 border-dashed border-gray-300 flex items-center justify-center cursor-pointer hover:border-indigo-500 transition-colors relative overflow-hidden"
              @click="triggerCircleIconUpload"
            >
              <img v-if="newCircleIcon" :src="newCircleIcon" class="w-full h-full object-cover">
              <div v-else class="text-gray-400">
                <i class="fas fa-camera text-xl"></i>
              </div>
              <input
                type="file"
                ref="circleIconInput"
                class="hidden"
                accept="image/*"
                @change="handleCircleIconUpload"
              >
            </div>
            <span class="text-sm text-gray-500 mt-2">点击上传图标</span>
          </div>

          <!-- 圈子名称 -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">圈子名称</label>
            <input
              v-model="newCircleName"
              type="text"
              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
              placeholder="请输入圈子名称"
            >
          </div>

          <!-- 保存按钮 -->
          <div class="flex justify-end">
            <button
              @click="saveCircle"
              class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors disabled:opacity-50"
              :disabled="!newCircleName.trim() || saving"
            >
              {{ saving ? '保存中...' : '保存' }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 编辑圈子弹窗 -->
    <div v-if="showEditCircleModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg w-96 p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-medium">编辑圈子</h3>
          <button @click="closeEditCircleModal" class="text-gray-400 hover:text-gray-600">
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="space-y-4">
          <!-- 圈子图标上传 -->
          <div class="flex flex-col items-center">
            <div 
              class="w-20 h-20 rounded-full border-2 border-dashed border-gray-300 flex items-center justify-center cursor-pointer hover:border-indigo-500 transition-colors relative overflow-hidden"
              @click="triggerEditCircleIconUpload"
            >
              <img v-if="editCircleForm.icon" :src="editCircleForm.icon" class="w-full h-full object-cover">
              <div v-else class="text-gray-400">
                <i class="fas fa-camera text-xl"></i>
              </div>
              <input
                type="file"
                ref="editCircleIconInput"
                class="hidden"
                accept="image/*"
                @change="handleEditCircleIconUpload"
              >
            </div>
            <span class="text-sm text-gray-500 mt-2">点击更换图标</span>
          </div>

          <!-- 圈子名称 -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">圈子名称</label>
            <input
              v-model="editCircleForm.circleName"
              type="text"
              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
              placeholder="请输入圈子名称"
            >
          </div>

          <!-- 保存按钮 -->
          <div class="flex justify-end">
            <button
              @click="updateCircle"
              class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors disabled:opacity-50"
              :disabled="!editCircleForm.circleName.trim() || saving"
            >
              {{ saving ? '保存中...' : '保存' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Circle {
  id: number
  circleName: string
  icon?: string
  children: Circle[]
}

interface Moment {
  id: number
  circleId: number
  content: string
  picUrlList: string[]
  replyCount: number
  createdTime: number
}

const circles = ref<Circle[]>([])
const moments = ref<Moment[]>([])
const currentCircleId = ref<number>(0)
const currentPage = ref(1)
const totalPages = ref(1)
const pageSize = 10

// 新增的状态和方法
const activeCommentId = ref<number | null>(null)
const activeReplyId = ref<number | null>(null)
const commentContent = ref('')
const replyContent = ref('')
const uploadedImages = ref<string[]>([])
const fileInputs = ref<{ [key: number]: HTMLInputElement | null }>({})

// 新增圈子相关的状态
const showAddCircleModal = ref(false)
const newCircleName = ref('')
const newCircleIcon = ref('')
const selectedParentId = ref<number | null>(null)
const isSubCircle = ref(false)
const saving = ref(false)
const circleIconInput = ref<HTMLInputElement | null>(null)

// 添加成功提示相关的状态
const showSuccess = ref(false)
const successMessage = ref('')

// 添加发布动态相关的状态
const newMomentContent = ref('')
const momentImages = ref<string[]>([])
const momentFileInput = ref<HTMLInputElement | null>(null)

// 添加右侧信息栏的数据
const memberCount = ref(128)
const todayPosts = ref(25)

const hotTopics = ref([
  { title: '面试技巧分享', count: 58 },
  { title: 'Java开发日常', count: 45 },
  { title: '算法刷题打卡', count: 32 },
  { title: '校招经验交流', count: 28 },
  { title: '技术架构讨论', count: 20 }
])

// 添加编辑圈子相关的状态
const showEditCircleModal = ref(false)
const editCircleForm = ref({
  id: 0,
  circleName: '',
  icon: '',
  parentId: 1
})
const editCircleIconInput = ref<HTMLInputElement | null>(null)

// 添加管理员状态判断
const isAdmin = ref(false)

// 获取圈子列表
const fetchCircles = async () => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/circle/list')
    const data = await response.json()
    if (data.success) {
      circles.value = data.data
      // 默认选择第一个子圈子
      if (data.data[0]?.children[0]) {
        selectCircle(data.data[0].children[0].id)
      }
    }
  } catch (error) {
    console.error('获取圈子列表失败:', error)
  }
}

// 获取圈子内容
const fetchMoments = async () => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/moment/getMoments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        circleId: currentCircleId.value,
        pageInfo: {
          pageNo: currentPage.value,
          pageSize: pageSize
        }
      })
    })
    const data = await response.json()
    if (data.success) {
      moments.value = data.data.result
      totalPages.value = data.data.totalPages
    }
  } catch (error) {
    console.error('获取圈子内容失败:', error)
  }
}

// 选择圈子
const selectCircle = (circleId: number) => {
  currentCircleId.value = circleId
  currentPage.value = 1
  fetchMoments()
}

// 切换页面
const changePage = (page: number) => {
  currentPage.value = page
  fetchMoments()
}

// 格式化日期
const formatDate = (timestamp: number) => {
  return new Date(timestamp).toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

// 显示评论输入框
const showCommentInput = (momentId: number) => {
  activeCommentId.value = momentId
  activeReplyId.value = null
  commentContent.value = ''
  uploadedImages.value = []
}

// 显示回复输入框
const showReplyInput = (commentId: number) => {
  activeReplyId.value = commentId
  replyContent.value = ''
}

// 提交评论
const submitComment = async (momentId: number) => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/comment/save', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        momentId,
        replyType: 1,  // 1表示评论
        content: commentContent.value,
        picUrlList: uploadedImages.value  // 使用上传的图片URL数组
      })
    })

    const data = await response.json()
    if (data.success) {
      await fetchMoments()  // 重新获取动态列表
      // 清空输入
      commentContent.value = ''
      uploadedImages.value = []
      activeCommentId.value = null
      showSuccessMessage('评论成功')  // 添加成功提示
    } else {
      throw new Error(data.message || '评论失败')
    }
  } catch (error) {
    console.error('提交评论失败:', error)
    alert('评论失败：' + (error instanceof Error ? error.message : '请重试'))
  }
}

// 提交回复
const submitReply = async (momentId: number, commentId: number) => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/comment/save', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        momentId,
        replyType: 2,  // 2表示回复
        content: replyContent.value,
        picUrlList: []  // 回复暂不支持图片
      })
    })

    const data = await response.json()
    if (data.success) {
      await fetchMoments()
      replyContent.value = ''
      activeReplyId.value = null
      showSuccessMessage('回复成功')  // 添加成功提示
    } else {
      throw new Error(data.message || '回复失败')
    }
  } catch (error) {
    console.error('提交回复失败:', error)
    alert('回复失败：' + (error instanceof Error ? error.message : '请重试'))
  }
}

// 处理图片上传
const handleImageUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files && input.files[0]) {
    const file = input.files[0]
    const formData = new FormData()
    formData.append('uploadFile', file)
    formData.append('bucket', 'user')
    formData.append('objectName', 'icon')

    try {
      const authInfo = localStorage.getItem('userAuthInfo')
      const { tokenValue = '' } = authInfo ? JSON.parse(authInfo) : {}

      const response = await fetch('http://localhost:4000/upload', {
        method: 'POST',
        headers: {
          'satoken': 'mianbao ' + tokenValue
        },
        body: formData
      })

      const data = await response.json()
      if (data.success) {
        uploadedImages.value.push(data.data)
        // 清空input以允许重复上传
        input.value = ''
        showSuccessMessage('图片上传成功')
      } else {
        throw new Error(data.message || '上传失败')
      }
    } catch (error) {
      console.error('上传图片失败:', error)
      alert('上传图片失败，请重试')
    }
  }
}

// 图片预览
const previewImage = (url: string) => {
  // 在新窗口打开图片
  window.open(url, '_blank')
}

// 移除已上传的图片
const removeImage = (index: number) => {
  uploadedImages.value.splice(index, 1)
}

// 触发文件选择
const triggerFileInput = (momentId: number) => {
  const input = fileInputs.value[momentId]
  if (input && typeof input.click === 'function') {
    input.click()
  }
}

// 点赞功能
const toggleLike = async (moment: any) => {
  // 实现点赞/取消点赞的逻辑
  moment.isLiked = !moment.isCall
  moment.likeCount += moment.isLiked ? 1 : -1
}

// 显示添加子圈子弹窗
const showAddSubCircle = (parentId: number) => {
  selectedParentId.value = parentId
  isSubCircle.value = true
  showAddCircleModal.value = true
}

// 关闭弹窗
const closeAddCircleModal = () => {
  showAddCircleModal.value = false
  newCircleName.value = ''
  newCircleIcon.value = ''
  selectedParentId.value = null
  isSubCircle.value = false
}

// 触发圈子图标上传
const triggerCircleIconUpload = () => {
  circleIconInput.value?.click()
}

// 处理圈子图标上传
const handleCircleIconUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files && input.files[0]) {
    const file = input.files[0]
    const formData = new FormData()
    formData.append('uploadFile', file)
    formData.append('bucket', 'user')
    formData.append('objectName', 'icon')

    try {
      const authInfo = localStorage.getItem('userAuthInfo')
      const { tokenValue = '' } = authInfo ? JSON.parse(authInfo) : {}

      const response = await fetch('http://localhost:4000/upload', {
        method: 'POST',
        headers: {
          'satoken': 'mianbao ' + tokenValue
        },
        body: formData
      })

      const data = await response.json()
      if (data.success) {
        newCircleIcon.value = data.data
      } else {
        throw new Error(data.message || '上传失败')
      }
    } catch (error) {
      console.error('上传图标失败:', error)
      alert('上传图标失败，请重试')
    }
  }
}

// 保存圈子
const saveCircle = async () => {
  if (!newCircleName.value.trim()) return

  try {
    saving.value = true
    const response = await fetch('http://127.0.0.1:3014/share/circle/save', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        circleName: newCircleName.value,
        icon: newCircleIcon.value,
        parentId: selectedParentId.value || null
      })
    })

    const data = await response.json()
    if (data.success) {
      await fetchCircles()
      closeAddCircleModal()
      showSuccessMessage('创建圈子成功！')
    } else {
      throw new Error(data.message || '创建失败')
    }
  } catch (error) {
    console.error('保存圈子失败:', error)
    alert('创建圈子失败：' + (error instanceof Error ? error.message : '请重试'))
  } finally {
    saving.value = false
  }
}

// 显示成功提示
const showSuccessMessage = (message: string) => {
  successMessage.value = message
  showSuccess.value = true
  setTimeout(() => {
    showSuccess.value = false
  }, 3000) // 3秒后自动消失
}

// 删除圈子
const deleteCircle = async (id: number) => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/circle/remove', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ id })
    })

    const data = await response.json()
    if (data.success) {
      showSuccessMessage('删除成功')
      await fetchCircles() // 重新获取圈子列表
    } else {
      throw new Error(data.message || '删除失败')
    }
  } catch (error) {
    console.error('删除失败:', error)
    alert('删除失败：' + (error instanceof Error ? error.message : '请重试'))
  }
}

// 确认删除
const confirmDeleteCircle = (id: number) => {
  if (confirm('确定要删除吗？')) {
    deleteCircle(id)
  }
}

// 触发动态图片上传
const triggerMomentFileInput = () => {
  momentFileInput.value?.click()
}

// 处理动态图片上传
const handleMomentImageUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files && input.files[0]) {
    const file = input.files[0]
    const formData = new FormData()
    formData.append('uploadFile', file)
    formData.append('bucket', 'user')
    formData.append('objectName', 'icon')

    try {
      const authInfo = localStorage.getItem('userAuthInfo')
      const { tokenValue = '' } = authInfo ? JSON.parse(authInfo) : {}

      const response = await fetch('http://localhost:4000/upload', {
        method: 'POST',
        headers: {
          'satoken': 'mianbao ' + tokenValue
        },
        body: formData
      })

      const data = await response.json()
      if (data.success) {
        momentImages.value.push(data.data)
        if (input) {
          input.value = ''
        }
        showSuccessMessage('图片上传成功')
      } else {
        throw new Error(data.message || '上传失败')
      }
    } catch (error) {
      console.error('上传图片失败:', error)
      alert('上传图片失败，请重试')
    }
  }
}

// 移除动态图片
const removeMomentImage = (index: number) => {
  momentImages.value.splice(index, 1)
}

// 发布动态
const publishMoment = async () => {
  if (!newMomentContent.value.trim() && !momentImages.value.length) return

  try {
    const response = await fetch('http://127.0.0.1:3014/share/moment/save', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        circleId: currentCircleId.value,
        content: newMomentContent.value,
        picUrlList: momentImages.value
      })
    })

    const data = await response.json()
    if (data.success) {
      // 清空输入
      newMomentContent.value = ''
      momentImages.value = []
      // 重新获取动态列表
      await fetchMoments()
      showSuccessMessage('发布成功')
    } else {
      throw new Error(data.message || '发布失败')
    }
  } catch (error) {
    console.error('发布失败:', error)
    alert('发布失败：' + (error instanceof Error ? error.message : '请重试'))
  }
}

// 显示编辑圈子弹窗
const showEditCircle = (circle: any) => {
  editCircleForm.value = {
    id: circle.id,
    circleName: circle.circleName,
    icon: circle.icon || '',
    parentId: circle.parentId || -1
  }
  showEditCircleModal.value = true
}

// 关闭编辑弹窗
const closeEditCircleModal = () => {
  showEditCircleModal.value = false
  editCircleForm.value = {
    id: 0,
    circleName: '',
    icon: '',
    parentId: -1
  }
}

// 触发编辑圈子图标上传
const triggerEditCircleIconUpload = () => {
  editCircleIconInput.value?.click()
}

// 处理编辑圈子图标上传
const handleEditCircleIconUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files && input.files[0]) {
    const file = input.files[0]
    const formData = new FormData()
    formData.append('uploadFile', file)
    formData.append('bucket', 'user')
    formData.append('objectName', 'icon')

    try {
      const authInfo = localStorage.getItem('userAuthInfo')
      const { tokenValue = '' } = authInfo ? JSON.parse(authInfo) : {}

      const response = await fetch('http://localhost:4000/upload', {
        method: 'POST',
        headers: {
          'satoken': 'mianbao ' + tokenValue
        },
        body: formData
      })

      const data = await response.json()
      if (data.success) {
        editCircleForm.value.icon = data.data
      } else {
        throw new Error(data.message || '上传失败')
      }
    } catch (error) {
      console.error('上传图标失败:', error)
      alert('上传图标失败，请重试')
    }
  }
}

// 更新圈子
const updateCircle = async () => {
  if (!editCircleForm.value.circleName.trim()) return

  try {
    saving.value = true
    const response = await fetch('http://127.0.0.1:3014/share/circle/update', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(editCircleForm.value)
    })

    const data = await response.json()
    if (data.success) {
      await fetchCircles()
      closeEditCircleModal()
      showSuccessMessage('更新成功')
    } else {
      throw new Error(data.message || '更新失败')
    }
  } catch (error) {
    console.error('更新圈子失败:', error)
    alert('更新失败：' + (error instanceof Error ? error.message : '请重试'))
  } finally {
    saving.value = false
  }
}

// 在 onMounted 中检查用户权限
const checkAdminStatus = () => {
  const userInfo = localStorage.getItem('userInfo')
  if (userInfo) {
    const { role } = JSON.parse(userInfo)
    isAdmin.value = role === 'admin' // 根据实际的角色判断逻辑修改
  }
}

// 删除内容
const deleteMoment = async (id: number) => {
  try {
    const response = await fetch('http://127.0.0.1:3014/share/moment/remove', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ id })
    })

    const data = await response.json()
    if (data.success) {
      showSuccessMessage('删除成功')
      await fetchMoments() // 重新获取内容列表
    } else {
      throw new Error(data.message || '删除失败')
    }
  } catch (error) {
    console.error('删除失败:', error)
    alert('删除失败：' + (error instanceof Error ? error.message : '请重试'))
  }
}

// 确认删除
const confirmDeleteMoment = (id: number) => {
  if (confirm('确定要删除这条内容吗？')) {
    deleteMoment(id)
  }
}

onMounted(() => {
  fetchCircles()
  checkAdminStatus()
})
</script>

<style scoped>
/* 添加更多动画效果 */
.hover\:scale-105:hover {
  transform: scale(1.05);
}

.transition-all {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 优化滚动条 */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb {
  background: #cbd5e0;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #a0aec0;
}

/* 添加动画效果 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 点赞动画 */
@keyframes like-animation {
  0% { transform: scale(1); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

.liked {
  animation: like-animation 0.3s ease;
}

/* 添加圈子图标上传悬浮效果 */
.circle-icon-upload:hover {
  @apply border-indigo-500;
}

/* 添加弹窗动画 */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

/* 添加圈子列表项悬浮效果 */
.circle-item:hover {
  @apply transform translate-x-1;
}

/* 添加成功提示动画 */
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fade-in 0.3s ease-out;
}

/* 删除按钮悬浮效果 */
.group:hover .opacity-0 {
  opacity: 1;
}

/* 添加卡片悬浮效果 */
.hover\:shadow-lg:hover {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

/* 删除按钮动画 */
.hover\:text-red-600:hover {
  @apply transform scale-110;
}
</style> 