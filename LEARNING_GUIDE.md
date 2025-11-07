# RuoYi-Vue-Plus 前端学习指南

## 📚 学习路线

### 第一阶段：基础环境搭建和项目运行 (1-2天)

#### 1. 环境准备
```bash
# 确保安装了 Node.js >= 18.18.0
node -v

# 安装依赖
npm install --registry=https://registry.npmmirror.com

# 启动开发服务器
npm run dev
```

#### 2. 项目结构熟悉
浏览以下关键文件，了解项目骨架：
- `src/main.ts` - 应用入口，了解插件注册顺序
- `src/App.vue` - 根组件
- `src/router/index.ts` - 路由配置
- `src/permission.ts` - 权限控制逻辑
- `.env.development` - 开发环境配置

**学习任务：**
- [ ] 成功启动项目
- [ ] 修改 `src/views/index.vue` 首页内容，看到实时更新
- [ ] 在浏览器开发者工具中查看 Vue DevTools
- [ ] 查看网络请求，了解 API 调用

---

### 第二阶段：Vue 3 核心概念 (3-5天)

#### 1. Composition API
学习文件：`src/views/` 下的各个页面组件

**关键概念：**
```vue
<script setup lang="ts">
// 1. 响应式数据
const count = ref(0)
const user = reactive({ name: '张三', age: 20 })

// 2. 计算属性
const doubleCount = computed(() => count.value * 2)

// 3. 监听器
watch(count, (newVal, oldVal) => {
  console.log(`count changed from ${oldVal} to ${newVal}`)
})

// 4. 生命周期
onMounted(() => {
  console.log('组件已挂载')
})

// 5. 方法定义
const increment = () => {
  count.value++
}
</script>
```

**实践任务：**
- [ ] 阅读 `src/views/index.vue` 首页组件，理解其结构
- [ ] 创建一个简单的计数器组件，使用 ref、computed、watch
- [ ] 在计数器中添加生命周期钩子，观察执行时机

#### 2. 组件通信
学习文件：`src/components/` 和 `src/layout/components/`

**常用方式：**
- Props（父传子）
- Emits（子传父）
- Provide/Inject（跨层级）
- Pinia Store（全局状态）

**实践任务：**
- [ ] 创建父子组件，实现 props 传递
- [ ] 子组件通过 emit 向父组件发送事件
- [ ] 查看 `src/layout/components/Sidebar/` 如何使用 Pinia

---

### 第三阶段：TypeScript 集成 (2-3天)

#### 1. 类型定义
学习文件：`src/types/`, `src/api/types.ts`

**关键概念：**
```typescript
// 1. 接口定义
interface User {
  userId: number
  userName: string
  email?: string  // 可选属性
}

// 2. 类型别名
type Status = 'success' | 'error' | 'warning'

// 3. 泛型
interface ApiResponse<T> {
  code: number
  msg: string
  data: T
}

// 4. Props 类型
interface Props {
  title: string
  count?: number
}
const props = defineProps<Props>()
```

**实践任��：**
- [ ] 为你的计数器组件添加 TypeScript 类型
- [ ] 创建一个 `interface` 定义用户数据结构
- [ ] 理解 `src/api/types.ts` 中的类型定义

---

### 第四阶段：状态管理 Pinia (2-3天)

#### 1. Store 结构
学习文件：`src/store/modules/`

**核心文件：**
- `user.ts` - 用户信息、登录登出
- `permission.ts` - 权限和动态路由
- `settings.ts` - 应用设置
- `dict.ts` - 字典数据

**Store 基本结构：**
```typescript
export const useExampleStore = defineStore('example', () => {
  // 1. State (响应式数据)
  const count = ref(0)
  const user = ref<User | null>(null)

  // 2. Getters (计算属性)
  const doubleCount = computed(() => count.value * 2)

  // 3. Actions (方法)
  const increment = () => {
    count.value++
  }

  const fetchUser = async () => {
    const res = await getUserInfo()
    user.value = res.data
  }

  return { count, user, doubleCount, increment, fetchUser }
})
```

**实践任务：**
- [ ] 阅读 `src/store/modules/user.ts`，理解登录流程
- [ ] 创建一个自己的 Store（如 TodoStore）
- [ ] 在组件中使用你的 Store

---

### 第五阶段：路由和权限 (3-4天)

#### 1. 路由配置
学习文件：`src/router/index.ts`, `src/permission.ts`

**路由结构：**
```typescript
{
  path: '/system/user',
  component: Layout,
  meta: { title: '用户管理', icon: 'user' },
  children: [
    {
      path: '',
      component: () => import('@/views/system/user/index.vue'),
      name: 'User',
      meta: { title: '用户管理' }
    }
  ]
}
```

**权限控制流程：**
1. 用户访问路由
2. `permission.ts` 中的 `router.beforeEach` 拦截
3. 检查 token
4. 获取用户信息和权限
5. 动态生成路由（`usePermissionStore().generateRoutes()`）
6. 添加到路由表
7. 放行访问

**实践任务：**
- [ ] 理解静态路由和动态路由的区别
- [ ] 在 `permission.ts` 中添加 console.log，观察路由守卫执行流程
- [ ] 使用 `v-hasPermi` 指令控制按钮显示
- [ ] 创建一个新的路由（目前通过后端配置，但可以理解结构）

---

### 第六阶段：HTTP 请求和 API (3-4天)

#### 1. Axios 封装
学习文件：`src/utils/request.ts`, `src/api/`

**请求拦截器功能：**
- 添加 token
- 添加 clientId（多租户）
- 防重复提交
- 请求加密
- 语言头设置

**响应拦截器功能：**
- 响应解密
- 统一错误处理
- 401 自动登出
- 业务错误提示

**API 定义示例：**
```typescript
// src/api/system/user.ts
import request from '@/utils/request'

// 查询用户列表
export function listUser(query: UserQuery) {
  return request({
    url: '/system/user/list',
    method: 'get',
    params: query
  })
}

// 新增用户
export function addUser(data: UserForm) {
  return request({
    url: '/system/user',
    method: 'post',
    data: data,
    headers: { isEncrypt: 'true' }  // 启用加密
  })
}
```

**实践任务：**
- [ ] 阅读 `src/utils/request.ts`，理解拦截器逻辑
- [ ] 查看 `src/api/system/user.ts`，了解 API 定义规范
- [ ] 在组件中调用 API，处理 loading 和错误
- [ ] 使用浏览器开发者工具查看请求头和响应

---

### 第七阶段：UI 组件库 Element Plus (2-3天)

#### 1. 常用组件
本项目使用自动导入，无需手动 import

**表单组件：**
- `<el-form>` - 表单容器
- `<el-input>` - 输入框
- `<el-select>` - 下拉选择
- `<el-date-picker>` - 日期选择
- `<el-upload>` - 文件上传

**数据展示：**
- `<el-table>` - 基础表格
- `<vxe-table>` - 高级表格（大数据量）
- `<el-pagination>` - 分页
- `<el-dialog>` - 对话框

**反馈组件：**
```typescript
// 消息提示（自动导入，直接使用）
ElMessage.success('操作成功')
ElMessage.error('操作失败')

// 确认对话框
ElMessageBox.confirm('确定要删除吗?', '提示').then(() => {
  // 确认操作
})
```

**实践任务：**
- [ ] 查看 `src/views/system/user/index.vue` 的表单和表格实现
- [ ] 创建一个包含表单验证的页面
- [ ] 实现一个带分页的表格
- [ ] 使用 `ElMessage` 和 `ElMessageBox`

---

### 第八阶段：项目特色功能 (4-5天)

#### 1. 字典管理
学习文件：`src/utils/dict.ts`, `src/store/modules/dict.ts`

**使用方式：**
```vue
<script setup>
// 加载字典
const { sys_user_sex } = useDict('sys_user_sex')

// 模板中使用
</script>

<template>
  <!-- 显示字典标签 -->
  <dict-tag :options="sys_user_sex" :value="user.sex" />

  <!-- 下拉选择 -->
  <el-select v-model="form.sex">
    <el-option
      v-for="dict in sys_user_sex"
      :key="dict.value"
      :label="dict.label"
      :value="dict.value"
    />
  </el-select>
</template>
```

#### 2. 文件上传下载
学习文件：`src/components/FileUpload/`, `src/components/ImageUpload/`

```typescript
// 下载文件
import { download } from '@/utils/request'

download('/system/user/export', params, '用户数据.xlsx')
```

#### 3. 权限指令和函数
```vue
<template>
  <!-- 指令方式 -->
  <el-button v-hasPermi="['system:user:add']">新增</el-button>
  <el-button v-hasRole="['admin']">管理员功能</el-button>
</template>

<script setup>
// 函数方式
import { useUserStore } from '@/store/modules/user'

const userStore = useUserStore()
const hasAddPermission = userStore.permissions.includes('system:user:add')
</script>
```

#### 4. 多语言国际化
学习文件：`src/lang/`

```typescript
// 使用 i18n
const { t } = useI18n()

// 模板中
{{ t('login.username') }}
```

**实践任务：**
- [ ] 在表单中使用字典组件
- [ ] 实现文件上传功能
- [ ] 使用权限指令控制按钮显示
- [ ] 切换系统语言（中文/英文）

---

### 第九阶段：样式和主题 (2-3天)

#### 1. UnoCSS 原子化 CSS
学习文件：`uno.config.ts`

**常用类名：**
```html
<!-- 布局 -->
<div class="flex items-center justify-between">
<div class="w-full h-100px p-4 m-2">

<!-- 文本 -->
<span class="text-lg font-bold text-red-500">

<!-- 响应式 -->
<div class="md:w-1/2 lg:w-1/3">
```

#### 2. SCSS 样式
学习文件：`src/assets/styles/`

**全局样式：**
- `index.scss` - 主样式入口
- `variables.module.scss` - SCSS 变量
- `transition.scss` - 过渡动画

#### 3. 主题切换
学习文件：`src/utils/theme.ts`, `src/store/modules/settings.ts`

**实践任务：**
- [ ] 使用 UnoCSS 创建一个响应式布局
- [ ] 修改主题色（在右上角设置面板）
- [ ] 切换暗黑模式
- [ ] 自定义一个 SCSS 样式文件

---

## 🛠️ 二次开发指南

### 开发新功能的标准流程

#### 1. 需求分析
- 确定功能模块（如：文章管理）
- 设计数据结构和 API 接口
- 规划页面布局和交互

#### 2. 创建 API 接口
```typescript
// src/api/article.ts
import request from '@/utils/request'

export interface Article {
  id?: number
  title: string
  content: string
  status: string
  createTime?: string
}

export interface ArticleQuery {
  title?: string
  status?: string
  pageNum: number
  pageSize: number
}

// 查询文章列表
export function listArticle(query: ArticleQuery) {
  return request({
    url: '/article/list',
    method: 'get',
    params: query
  })
}

// 新增文章
export function addArticle(data: Article) {
  return request({
    url: '/article',
    method: 'post',
    data: data
  })
}

// 修改文章
export function updateArticle(data: Article) {
  return request({
    url: '/article',
    method: 'put',
    data: data
  })
}

// 删除文章
export function delArticle(id: number) {
  return request({
    url: `/article/${id}`,
    method: 'delete'
  })
}

// 获取文章详情
export function getArticle(id: number) {
  return request({
    url: `/article/${id}`,
    method: 'get'
  })
}
```

#### 3. 创建页面组件

**目录结构：**
```
src/views/article/
├── index.vue          # 列表页
└── components/
    └── ArticleForm.vue # 表单弹窗
```

**列表页模板 (index.vue)：**
```vue
<template>
  <div class="app-container">
    <!-- 查询表单 -->
    <el-form :model="queryParams" :inline="true">
      <el-form-item label="标题">
        <el-input v-model="queryParams.title" placeholder="请输入标题" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleQuery">搜索</el-button>
        <el-button @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- 操作按钮 -->
    <el-row class="mb-2">
      <el-button type="primary" v-hasPermi="['article:add']" @click="handleAdd">
        新增
      </el-button>
    </el-row>

    <!-- 数据表格 -->
    <el-table v-loading="loading" :data="articleList">
      <el-table-column label="ID" prop="id" width="80" />
      <el-table-column label="标题" prop="title" />
      <el-table-column label="状态" prop="status">
        <template #default="{ row }">
          <dict-tag :options="article_status" :value="row.status" />
        </template>
      </el-table-column>
      <el-table-column label="创建时间" prop="createTime" width="180" />
      <el-table-column label="操作" width="200">
        <template #default="{ row }">
          <el-button
            text
            type="primary"
            v-hasPermi="['article:edit']"
            @click="handleUpdate(row)"
          >
            修改
          </el-button>
          <el-button
            text
            type="danger"
            v-hasPermi="['article:remove']"
            @click="handleDelete(row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <pagination
      v-show="total > 0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 表单弹窗 -->
    <el-dialog v-model="open" :title="title" width="800px">
      <el-form ref="formRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入标题" />
        </el-form-item>
        <el-form-item label="内容" prop="content">
          <el-input v-model="form.content" type="textarea" :rows="5" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="form.status">
            <el-option
              v-for="dict in article_status"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="cancel">取消</el-button>
        <el-button type="primary" @click="submitForm">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts" name="Article">
import { listArticle, addArticle, updateArticle, delArticle, getArticle } from '@/api/article'
import type { Article, ArticleQuery } from '@/api/article'

// 查询参数
const queryParams = ref<ArticleQuery>({
  title: undefined,
  status: undefined,
  pageNum: 1,
  pageSize: 10
})

// 表格数据
const articleList = ref<Article[]>([])
const loading = ref(false)
const total = ref(0)

// 表单相关
const open = ref(false)
const title = ref('')
const form = ref<Article>({
  title: '',
  content: '',
  status: '0'
})
const formRef = ref()

// 表单验证规则
const rules = {
  title: [{ required: true, message: '标题不能为空', trigger: 'blur' }],
  content: [{ required: true, message: '内容不能为空', trigger: 'blur' }]
}

// 字典
const { article_status } = useDict('article_status')

// 查询列表
const getList = async () => {
  loading.value = true
  try {
    const res = await listArticle(queryParams.value)
    articleList.value = res.rows
    total.value = res.total
  } finally {
    loading.value = false
  }
}

// 搜索
const handleQuery = () => {
  queryParams.value.pageNum = 1
  getList()
}

// 重置
const resetQuery = () => {
  queryParams.value = {
    title: undefined,
    status: undefined,
    pageNum: 1,
    pageSize: 10
  }
  getList()
}

// 新增
const handleAdd = () => {
  reset()
  open.value = true
  title.value = '添加文章'
}

// 修改
const handleUpdate = async (row: Article) => {
  reset()
  const res = await getArticle(row.id!)
  form.value = res.data
  open.value = true
  title.value = '修改文章'
}

// 删除
const handleDelete = async (row: Article) => {
  await ElMessageBox.confirm(`是否确认删除文章"${row.title}"?`, '警告', {
    type: 'warning'
  })
  await delArticle(row.id!)
  ElMessage.success('删除成功')
  await getList()
}

// 提交表单
const submitForm = async () => {
  await formRef.value.validate()
  if (form.value.id) {
    await updateArticle(form.value)
    ElMessage.success('修改成功')
  } else {
    await addArticle(form.value)
    ElMessage.success('新增成功')
  }
  open.value = false
  await getList()
}

// 取消
const cancel = () => {
  open.value = false
  reset()
}

// 重置表单
const reset = () => {
  form.value = {
    title: '',
    content: '',
    status: '0'
  }
  formRef.value?.resetFields()
}

// 初始化
onMounted(() => {
  getList()
})
</script>
```

#### 4. 后端配置路由
在后端系统管理-菜单管理中添加菜单配置：
- 菜单名称：文章管理
- 路由地址：article
- 组件路径：article/index
- 权限标识：article:list, article:add, article:edit, article:remove

#### 5. 测试功能
- 增删改查操作
- 权限控制
- 表单验证
- 错误处理
- 响应式布局

---

### 常见二次开发场景

#### 场景 1：添加新的表单字段
```typescript
// 1. 更新类型定义
interface UserForm {
  userName: string
  nickName: string
  phone: string  // 新增
}

// 2. 在表单中添加
<el-form-item label="手机号" prop="phone">
  <el-input v-model="form.phone" />
</el-form-item>

// 3. 添加验证规则
const rules = {
  phone: [
    { required: true, message: '手机号不能为空', trigger: 'blur' },
    { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
  ]
}
```

#### 场景 2：添加自定义工具函数
```typescript
// src/utils/myUtils.ts
/**
 * 格式化金额（分转元）
 */
export function formatMoney(amount: number): string {
  return (amount / 100).toFixed(2)
}

/**
 * 生成随机字符串
 */
export function randomString(length: number = 8): string {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
  let result = ''
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length))
  }
  return result
}

// 在组件中使用
import { formatMoney } from '@/utils/myUtils'

const price = formatMoney(12999) // "129.99"
```

#### 场景 3：封装通用组件
```vue
<!-- src/components/MyCard/index.vue -->
<template>
  <div class="my-card">
    <div class="my-card-header">
      <slot name="header">{{ title }}</slot>
    </div>
    <div class="my-card-body">
      <slot></slot>
    </div>
    <div class="my-card-footer" v-if="$slots.footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Props {
  title?: string
}

defineProps<Props>()
</script>

<style scoped lang="scss">
.my-card {
  border: 1px solid #eee;
  border-radius: 4px;

  &-header {
    padding: 12px 16px;
    border-bottom: 1px solid #eee;
    font-weight: bold;
  }

  &-body {
    padding: 16px;
  }

  &-footer {
    padding: 12px 16px;
    border-top: 1px solid #eee;
  }
}
</style>

<!-- 使用 -->
<MyCard title="用户信息">
  <p>这是卡片内容</p>
  <template #footer>
    <el-button>操作按钮</el-button>
  </template>
</MyCard>
```

#### 场景 4：集成第三方库
```bash
# 安装 ECharts 图表库（项目已包含）
npm install echarts

# 安装 Day.js 日期库
npm install dayjs
```

```vue
<!-- 使用 ECharts -->
<script setup>
import * as echarts from 'echarts'

const chartRef = ref()

onMounted(() => {
  const chart = echarts.init(chartRef.value)
  chart.setOption({
    xAxis: { type: 'category', data: ['周一', '周二', '周三'] },
    yAxis: { type: 'value' },
    series: [{ data: [120, 200, 150], type: 'line' }]
  })
})
</script>

<template>
  <div ref="chartRef" style="width: 600px; height: 400px"></div>
</template>
```

---

## 🔧 开发工具推荐

### VS Code 插件
- Vue Language Features (Volar) - Vue 3 支持
- TypeScript Vue Plugin (Volar) - TypeScript 支持
- ESLint - 代码检查
- Prettier - 代码格式化
- UnoCSS - UnoCSS 智能提示
- Vue VSCode Snippets - Vue 代码片段

### 浏览器扩展
- Vue DevTools - Vue 调试工具
- Redux DevTools - Pinia 调试（兼容）

### 配置 VS Code
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": ["javascript", "typescript", "vue"]
}
```

---

## 📖 学习资源

### 官方文档
- [Vue 3 文档](https://cn.vuejs.org/)
- [TypeScript 文档](https://www.typescriptlang.org/zh/)
- [Element Plus 文档](https://element-plus.org/zh-CN/)
- [Vite 文档](https://cn.vitejs.dev/)
- [Pinia 文档](https://pinia.vuejs.org/zh/)
- [Vue Router 文档](https://router.vuejs.org/zh/)
- [UnoCSS 文档](https://unocss.dev/)

### 推荐学习顺序
1. Vue 3 基础 → Composition API
2. TypeScript 基础语法
3. Pinia 状态管理
4. Vue Router 路由
5. Element Plus 组件库
6. HTTP 请求和 Axios
7. 项目实战

---

## 💡 调试技巧

### 1. 使用 console.log
```typescript
// 查看响应式数据
console.log('count:', count.value)
console.log('user:', toRaw(user.value))

// 查看 API 响应
const res = await listUser(query)
console.log('API response:', res)
```

### 2. 使用 Vue DevTools
- 查看组件树
- 检查组件 props 和 state
- 查看 Pinia store 状态
- 时间旅行调试

### 3. 断点调试
在 VS Code 中设置断点，使用 F5 启动调试

### 4. 查看网络请求
- 打开浏览器开发者工具 → Network 标签
- 查看请求 URL、headers、payload
- 查看响应 status、data

---

## ⚠️ 常见问题

### 1. 端口被占用
```bash
# 修改 .env.development 中的端口
VITE_APP_PORT = 8080
```

### 2. 依赖安装失败
```bash
# 清除缓存重新安装
rm -rf node_modules package-lock.json
npm install --registry=https://registry.npmmirror.com
```

### 3. TypeScript 类型错误
```bash
# 重启 TypeScript 服务器（VS Code）
Ctrl+Shift+P → TypeScript: Restart TS Server
```

### 4. ESLint 报错
```bash
# 自动修复
npm run lint:eslint:fix
```

### 5. 后端接口 404
- 检查 `.env.development` 中的 `VITE_APP_BASE_API`
- 检查 `vite.config.ts` 中的 proxy 配置
- 确认后端服务已启动

---

## 🎯 学习建议

1. **循序渐进**：不要急于求成，按阶段学习
2. **多动手**：每个知识点都要自己写代码实践
3. **看源码**：遇到不懂的功能，查看项目中的实现
4. **写注释**：在学习过程中给代码添加注释，帮助理解
5. **做笔记**：记录重要概念和常用代码片段
6. **问问题**：遇到问题及时搜索或询问
7. **做项目**：学完基础后，尝试开发自己的功能模块

---

## 📝 练习项目建议

### 初级项目
- 待办事项管理（Todo List）
- 个人博客管理
- 通讯录管理

### 中级项目
- 商品管理系统
- 订单管理系统
- 客户关系管理（CRM）

### 高级项目
- 多租户 SaaS 系统
- 工作流审批系统
- 数据可视化大屏

---

祝学习顺利！有问题随时问我。🚀
