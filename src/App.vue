<template>
  <div class="app-container">
    <!-- 加载动画 -->
    <div v-if="isLoading" class="loading-screen">
      <div class="matrix-rain"></div>
      <div class="loading-content">
        <div class="terminal-window">
          <div class="terminal-header">
            <span class="terminal-title">DARKNET TERMINAL v2.7.3</span>
            <div class="terminal-controls">
              <span class="control-btn minimize"></span>
              <span class="control-btn maximize"></span>
              <span class="control-btn close"></span>
            </div>
          </div>
          <div class="terminal-body">
            <div class="terminal-line" v-for="(line, index) in terminalLines" :key="index" :class="{ 'typing': line.typing }">
              <span class="prompt">root@darknet:~$</span>
              <span class="command">{{ line.text }}</span>
              <span v-if="line.typing" class="cursor">_</span>
            </div>
          </div>
        </div>
        <div class="loading-progress">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: loadingProgress + '%' }"></div>
          </div>
          <div class="progress-text">
            <span class="glitch-text">{{ loadingText }}</span>
            <span class="percentage">{{ loadingProgress }}%</span>
          </div>
        </div>
        <div class="loading-warnings">
          <div class="warning-line">⚠️ 检测到监控活动</div>
          <div class="warning-line">🔒 启用匿名保护</div>
          <div class="warning-line">🌐 连接到TOR网络</div>
        </div>
      </div>
    </div>
    
    <!-- 节点路由可视化 -->
    <div v-if="showRouteVisualization" class="route-overlay" @click="closeRouteVisualization">
      <div class="route-visualization" @click.stop>
        <div class="route-header">
          <h3><i class="fas fa-route"></i> 洋葱路由传输路径</h3>
          <button @click="closeRouteVisualization" class="close-btn">×</button>
        </div>
      <div class="route-path">
        <div v-for="(node, index) in routeNodes" :key="index" 
             :class="['route-node', { 'active': node.active, 'failed': node.failed }]">
          <div class="node-icon">
            <i class="fas fa-server"></i>
          </div>
          <div class="node-info">
            <div class="node-id">{{ node.id }}</div>
            <div class="node-status">{{ node.status }}</div>
            <div class="node-location">{{ node.location }}</div>
            <div class="encryption-layer">{{ node.encryption }}</div>
            <div class="node-metrics">
              <span class="bandwidth">{{ node.bandwidth }}</span>
              <span class="uptime">运行率: {{ node.uptime }}</span>
            </div>
          </div>
          <div v-if="index < routeNodes.length - 1" class="route-arrow">
            <i class="fas fa-arrow-right"></i>
          </div>
        </div>
      </div>
      <div class="route-details">
        <div class="detail-row">
          <div class="detail-item">
            <span class="label">传输状态:</span>
            <span :class="['status', routeStatus.class]">{{ routeStatus.text }}</span>
          </div>
          <div class="detail-item">
            <span class="label">加密层数:</span>
            <span class="value">{{ encryptionLayers }}/5</span>
          </div>
        </div>
        <div class="detail-row">
          <div class="detail-item">
            <span class="label">网络延迟:</span>
            <span class="value">{{ latency }}ms</span>
          </div>
          <div class="detail-item">
            <span class="label">数据包大小:</span>
            <span class="value">{{ packetSize }}KB</span>
          </div>
        </div>
        <div class="detail-row">
          <div class="detail-item">
            <span class="label">跳跃次数:</span>
            <span class="value">{{ hopCount }}/{{ routeNodes.length }}</span>
          </div>
          <div class="detail-item">
            <span class="label">匿名等级:</span>
            <span class="value security-level">{{ anonymityLevel }}</span>
          </div>
        </div>
        <div class="technical-info">
          <div class="info-item">
            <i class="fas fa-shield-alt"></i>
            <span>TLS 1.3 + Tor v3 洋葱服务</span>
          </div>
          <div class="info-item">
            <i class="fas fa-clock"></i>
            <span>电路建立时间: {{ circuitTime }}s</span>
          </div>
          <div class="info-item">
            <i class="fas fa-fingerprint"></i>
            <span>指纹混淆: {{ fingerprintObfuscation }}</span>
          </div>
        </div>
      </div>
      </div>
    </div>

    <!-- 加密状态显示 -->
    <div v-if="showEncryptionStatus" class="encryption-overlay" @click="closeEncryptionStatus">
      <div class="encryption-status" @click.stop>
        <div class="encryption-header">
          <h3><i class="fas fa-lock"></i> 端到端加密状态</h3>
          <button @click="closeEncryptionStatus" class="close-btn">×</button>
        </div>
      <div class="encryption-details">
        <div class="encryption-step" v-for="(step, index) in encryptionSteps" :key="index"
             :class="{ 'active': step.active, 'completed': step.completed }">
          <div class="step-icon">
            <i :class="step.icon"></i>
          </div>
          <div class="step-info">
            <div class="step-title">{{ step.title }}</div>
            <div class="step-description">{{ step.description }}</div>
          </div>
          <div class="step-status">
            <span :class="['status-indicator', step.status]"></span>
          </div>
        </div>
      </div>
      </div>
    </div>

    <!-- 暗网黑市彩蛋入口 -->
    <div class="market-entry" @click="checkMarketAccess">
      <i class="fas fa-shopping-cart"></i>
    </div>

    <!-- 暗网黑市界面 -->
    <div v-if="showDarkMarket" class="market-overlay" @click="closeDarkMarket">
      <div class="dark-market" @click.stop>
        <div class="market-header">
          <h2><i class="fab fa-bitcoin"></i> 暗网黑市 - 违禁品交易所</h2>
          <button @click="closeDarkMarket" class="close-btn">×</button>
        </div>
        <div class="market-warning">
          <i class="fas fa-skull-crossbones"></i>
          警告: 此区域包含非法内容，仅供演示用途
        </div>
        <div class="market-items">
          <div v-for="item in marketItems" :key="item.id" class="market-item">
            <div class="item-icon">
              <i :class="item.icon"></i>
            </div>
            <div class="item-info">
              <div class="item-name">{{ item.name }}</div>
              <div class="item-description">{{ item.description }}</div>
            </div>
            <div class="item-price">
              <div class="btc-price">{{ item.btcPrice }} BTC</div>
              <div class="cny-price">≈ ¥{{ item.cnyPrice }}</div>
            </div>
            <button class="buy-btn" @click="showPurchaseWarning(item)">
              <i class="fas fa-shopping-cart"></i> 购买
            </button>
          </div>
        </div>
        <div class="market-footer">
          <div class="btc-rate">
            <i class="fab fa-bitcoin"></i> 
            当前汇率: 1 BTC = ¥{{ btcToRmb }} (实时汇率)
          </div>
        </div>
      </div>
    </div>

    <!-- 主要内容 -->
    <div v-if="!isLoading">
      <div class="glitch-effect"></div>
      <div class="container">
      <div class="header">
        <h1>暗网发帖退回通知</h1>
        <div class="timestamp">服务器时间: <span>{{ currentTime }}</span> UTC</div>
      </div>

      <div class="warning">
        <i class="fas fa-exclamation-triangle"></i>
        严重: 你的网络活动已被监控。该通知仅可访问一次，阅读后将自动销毁。
      </div>

      <div class="notification" v-if="!isDestroyed">
        <h3>系统通知
          <span>{{ notificationData.notificationId }}</span>
        </h3>

        <div class="typing-animation">发帖退回 - 安全协议违规</div>

        <div class="notification-content">
          <div class="notification-item">
            <div class="label">用户:</div>
            <div class="value"><b>{{ notificationData.username }}</b></div>
          </div>

          <div class="notification-item">
            <div class="label">发帖时间:</div>
            <div class="value"><b>{{ notificationData.postDate }}</b></div>
          </div>

          <div class="notification-item">
            <div class="label">退回时间:</div>
            <div class="value"><b>{{ notificationData.returnDate }}</b></div>
          </div>

          <div class="notification-item">
            <div class="label">节点ID:</div>
            <div class="value"><b>{{ notificationData.nodeId }}</b></div>
          </div>

          <div class="notification-item">
            <div class="label">退回等级:</div>
            <div class="value">
              <b :class="['severity', getSeverityClass(notificationData.severity)]">{{ getSeverityText(notificationData.severity) }}</b>
            </div>
          </div>

          <div class="notification-item">
            <div class="label">原因代码:</div>
            <div class="value">
              <b>{{ notificationData.reasonCode }}</b> - <span>{{ getReasonCodeDescription(notificationData.reasonCode) }}</span>
            </div>
          </div>

          <div class="notification-item">
            <div class="label">退回原因:</div>
          </div>
          <div class="value" style="margin-bottom: 15px; background: rgba(255,0,60,0.1); padding: 10px; border-radius: 4px;">
            {{ notificationData.reason }}
          </div>

          <div class="notification-item">
            <div class="label">帖子预览:</div>
          </div>
          <div class="post-preview">
            <div class="post-title">{{ notificationData.postTitle }}</div>
            <div class="post-content">{{ notificationData.postContent }}</div>
            <img v-if="notificationData.imageUrl" :src="notificationData.imageUrl" alt="帖子图片" class="post-image">
          </div>

          <div class="notification-item">
            <div class="label">建议操作:</div>
          </div>
          <div class="value" style="margin-bottom: 15px; padding-left: 5px;">
            <ul>
              <li>立即删除相关内容</li>
              <li>在72小时内修改并重新提交</li>
              <li>联系节点管理员申请审查</li>
            </ul>
          </div>
          <p class="status-bar">
            <span @click="showEncryptionDetails" class="clickable-status">
              传输加密: <i class="fas fa-lock"></i> <span class="status-text">{{ encryptionStatusText }}</span>
            </span>
            <span>追踪状态: <i class="fas fa-user-secret"></i> <span class="status-text">匿名</span></span>
            <span @click="showRouteDetails" class="clickable-status">
              节点状态: <i class="fas fa-circle" style="color: #00ff41; font-size: 10px;"></i> <span class="status-text">{{ nodeStatusText }}</span>
            </span>
            <span>记录ID: {{ recordId }}</span>
          </p>
        </div>
      </div>

      <!-- 销毁状态显示 -->
      <div v-if="isDestroyed" style="text-align: center; padding: 60px 0;">
        <i class="fas fa-skull" style="font-size: 80px; color: #ff003c; margin-bottom: 30px;"></i>
        <h3 style="margin-bottom: 20px; color: #ff003c;">此通知已被销毁</h3>
        <p style="margin-bottom: 40px; color: #ff003c;">所有痕迹已清除，连接已终止</p>
        <button class="btn" @click="createNewNotification">
          <i class="fas fa-redo"></i> 创建新通知
        </button>
      </div>

      <div v-if="!isDestroyed">
        <h3 style="margin: 25px 0 15px; color: #00ff99; border-bottom: 1px solid #005522; padding-bottom: 10px;">
          <i class="fas fa-cog"></i> 通知设置
        </h3>

        <div class="notification-controls">
          <div class="input-group">
            <label for="notification-id-input"><i class="fas fa-id-badge"></i> 通知编号</label>
            <div class="notification-id-control">
              <input type="text" v-model="formData.notificationId" placeholder="输入通知编号">
              <button @click="generateNotificationId"><i class="fas fa-sync-alt"></i></button>
            </div>
          </div>

          <div class="input-group">
            <label for="severity"><i class="fas fa-radiation"></i> 退回等级</label>
            <select v-model="formData.severity">
              <option value="critical">严重</option>
              <option value="high">高</option>
              <option value="medium">中</option>
              <option value="low">低</option>
            </select>
          </div>

          <div class="input-group">
            <label for="node-id"><i class="fas fa-server"></i> 节点ID</label>
            <select v-model="formData.nodeId">
              <option value="ONION#8FD7KX">ONION#8FD7KX</option>
              <option value="ONION#5GH32T">ONION#5GH32T</option>
              <option value="ONION#9PQR6F">ONION#9PQR6F</option>
              <option value="ONION#2LM89N">ONION#2LM89N</option>
              <option value="ONION#1ZX45V">ONION#1ZX45V</option>
            </select>
          </div>

          <div class="input-group">
            <label for="reason-code"><i class="fas fa-bug"></i> 原因代码</label>
            <select v-model="formData.reasonCode">
              <option value="12.7a">12.7a - 违反内容协议</option>
              <option value="3.2b">3.2b - 非法交易信息</option>
              <option value="8.5c">8.5c - 敏感关键词</option>
              <option value="10.1d">10.1d - 泄露个人隐私</option>
              <option value="9.4e">9.4e - 恶意软件链接</option>
              <option value="6.3f">6.3f - 平台规则违反</option>
              <option value="15.8g">15.8g - 加密签名失效</option>
              <option value="7.1h">7.1h - 节点拒绝转发</option>
              <option value="11.3i">11.3i - 量子密钥协商失败</option>
              <option value="4.9j">4.9j - 洋葱路由中断</option>
              <option value="13.2k">13.2k - AES-256解密错误</option>
              <option value="5.6l">5.6l - 匿名层级不足</option>
            </select>
          </div>
        </div>

        <div class="input-row">
          <div class="input-group">
            <label for="post-date"><i class="fas fa-calendar"></i> 发帖日期</label>
            <input type="datetime-local" v-model="formData.postDate">
          </div>

          <div class="input-group">
            <label for="return-date"><i class="fas fa-calendar-times"></i> 退回日期</label>
            <input type="datetime-local" v-model="formData.returnDate">
          </div>
        </div>

        <div class="input-group">
          <label for="username"><i class="fas fa-user-secret"></i> 用户名/身份</label>
          <input type="text" v-model="formData.username" placeholder="输入你的用户名或身份标识">
        </div>

        <div class="input-row">
          <div class="input-group">
            <label for="post-title"><i class="fas fa-heading"></i> 帖子标题</label>
            <input type="text" v-model="formData.postTitle" placeholder="输入帖子标题">
          </div>

          <div class="input-group">
            <label for="image-upload"><i class="fas fa-image"></i> 上传图片</label>
            <div class="file-upload-container">
              <input type="file" id="image-upload" @change="handleImageUpload" accept="image/*" class="file-input">
              <label for="image-upload" class="file-upload-label">
                <i class="fas fa-upload"></i> 选择图片文件
              </label>
              <span v-if="uploadedImageName" class="uploaded-file-name">{{ uploadedImageName }}</span>
              <button v-if="formData.imageUrl" @click="removeImage" class="remove-image-btn" type="button">
                <i class="fas fa-times"></i>
              </button>
            </div>
          </div>
        </div>

        <div class="input-group">
          <label for="post-content"><i class="fas fa-file-alt"></i> 帖子内容</label>
          <textarea v-model="formData.postContent" placeholder="输入帖子内容..."></textarea>
        </div>

        <div class="input-group">
          <label for="reason"><i class="fas fa-exclamation-circle"></i> 退回原因</label>
          <textarea v-model="formData.reason" placeholder="输入发帖被退回的详细原因..."></textarea>
        </div>

        <div class="btn-container">
          <button class="btn btn-generate" @click="generateNotification">
            <i class="fas fa-code"></i> 生成通知
          </button>
          <button class="btn btn-submit" @click="submitAction">
            <i class="fas fa-paper-plane"></i> 提交重新审核
          </button>
          <button class="btn btn-share" @click="shareNotification">
            <i class="fas fa-share-alt"></i> 分享链接
          </button>
          <button class="btn btn-download" @click="saveNotificationAsImage">
            <i class="fas fa-download"></i> 保存图片
          </button>
          <button class="btn" @click="clearData">
            <i class="fas fa-trash-alt"></i> 清除记录
          </button>
          <button class="btn" @click="showLoadingAnimation">
            <i class="fas fa-sync-alt"></i> 重新连接
          </button>
          <button class="btn btn-warning" @click="selfDestruct">
            <i class="fas fa-skull"></i> 销毁通知
          </button>
        </div>
      </div>

      <div class="footer" v-if="!isDestroyed">
        <p>节点信息: TORv3 #8FD7KX | 密钥有效期: <span>{{ countdown }}</span></p>
        <p>警告: 所有活动均被记录。安全级别: MAXIMUM</p>
      </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted, onUnmounted, computed } from 'vue'

export default {
  name: 'App',
  setup() {
    const currentTime = ref('')
    const countdown = ref('18:34:15')
    const isDestroyed = ref(false)
    const isLoading = ref(true)
    const loadingProgress = ref(0)
    const loadingText = ref('初始化暗网连接...')
    const terminalLines = ref([])
    
    // 新增功能状态
    const showRouteVisualization = ref(false)
    const showEncryptionStatus = ref(false)
    const showDarkMarket = ref(false)
    const encryptionStatusText = ref('量子密钥协商中...')
    const nodeStatusText = ref('在线')
    const encryptionLayers = ref(3)
    const latency = ref(847)
    const packetSize = ref(64)
    const hopCount = ref(3)
    const anonymityLevel = ref('高级')
    const circuitTime = ref(4.2)
    const fingerprintObfuscation = ref('启用')
    const btcToRmb = ref(0)
    const marketClickCount = ref(0)
    const uploadedImageName = ref('')
    
    // 路由节点数据
    const routeNodes = ref([
      {
        id: '用户设备',
        status: '发送中',
        encryption: '初始加密',
        active: true,
        failed: false,
        location: '本地',
        bandwidth: '1.2 MB/s',
        uptime: '99.8%'
      },
      {
        id: 'ONION#8FD7KX',
        status: '转发中',
        encryption: 'AES-256层',
        active: true,
        failed: false,
        location: '德国·法兰克福',
        bandwidth: '847 KB/s',
        uptime: '97.3%'
      },
      {
        id: 'ONION#5GH32T',
        status: '拒绝转发',
        encryption: 'RSA-4096层',
        active: false,
        failed: true,
        location: '荷兰·阿姆斯特丹',
        bandwidth: '0 KB/s',
        uptime: '0%'
      },
      {
        id: 'ONION#9K2L8M',
        status: '建立连接',
        encryption: '椭圆曲线层',
        active: true,
        failed: false,
        location: '瑞士·苏黎世',
        bandwidth: '2.1 MB/s',
        uptime: '99.9%'
      },
      {
        id: '暗网服务器',
        status: '等待中',
        encryption: '量子加密层',
        active: false,
        failed: false,
        location: '未知·深网',
        bandwidth: '??? KB/s',
        uptime: '???'
      }
    ])
    
    // 路由状态
    const routeStatus = ref({
      text: '节点#5GH32T拒绝转发',
      class: 'status-error'
    })
    
    // 加密步骤
    const encryptionSteps = ref([
      {
        title: '量子密钥协商',
        description: '使用Diffie-Hellman密钥交换协议',
        icon: 'fas fa-key',
        active: true,
        completed: false,
        status: 'processing'
      },
      {
        title: 'AES-256加密',
        description: '对称加密算法保护数据传输',
        icon: 'fas fa-shield-alt',
        active: false,
        completed: true,
        status: 'completed'
      },
      {
        title: 'RSA-4096签名',
        description: '数字签名验证消息完整性',
        icon: 'fas fa-certificate',
        active: false,
        completed: false,
        status: 'failed'
      },
      {
        title: '洋葱路由封装',
        description: '多层加密保护传输路径',
        icon: 'fas fa-layer-group',
        active: false,
        completed: true,
        status: 'completed'
      }
    ])
    
    // 暗网黑市商品
    const marketItems = ref([
      {
        id: 1,
        name: '敏感词过滤器',
        description: '绕过内容审查的高级工具',
        icon: 'fas fa-filter',
        btcPrice: 0.003,
        cnyPrice: 0
      },
      {
        id: 2,
        name: '匿名身份包',
        description: '完整的虚假身份信息套装',
        icon: 'fas fa-mask',
        btcPrice: 0.015,
        cnyPrice: 0
      },
      {
        id: 3,
        name: '加密通信软件',
        description: '军用级端到端加密聊天工具',
        icon: 'fas fa-comments',
        btcPrice: 0.008,
        cnyPrice: 0
      },
      {
        id: 4,
        name: '节点访问权限',
        description: '高级暗网节点永久访问权',
        icon: 'fas fa-server',
        btcPrice: 0.025,
        cnyPrice: 0
      },
      {
        id: 5,
        name: '数据销毁服务',
        description: '专业级数字痕迹清除',
        icon: 'fas fa-eraser',
        btcPrice: 0.012,
        cnyPrice: 0
      }
    ])
    
    // 表单数据
    const formData = reactive({
      notificationId: 'REF-7X3B9P',
      severity: 'critical',
      nodeId: 'ONION#8FD7KX',
      reasonCode: '12.7a',
      postDate: '',
      returnDate: '',
      username: '',
      postTitle: '',
      imageUrl: '',
      postContent: '',
      reason: ''
    })

    // 通知显示数据
    const notificationData = reactive({
      notificationId: 'REF-7X3B9P',
      username: '匿名用户',
      postDate: '2023-08-26 05:31:17 UTC',
      returnDate: '2023-08-27 08:15:32 UTC',
      nodeId: 'ONION#8FD7KX',
      severity: 'critical',
      reasonCode: '12.7a',
      reason: '你的帖子包含受限制的内容或关键词，违反了当前节点的安全规定。',
      postTitle: '未指定帖子标题',
      postContent: '没有可用的内容预览',
      imageUrl: ''
    })

    // 计算属性：记录ID
    const recordId = computed(() => {
      const now = new Date()
      const dateStr = now.toISOString().slice(0, 10).replace(/-/g, '')
      const timeStr = now.toTimeString().slice(0, 8).replace(/:/g, '')
      return `8FD7KX-${dateStr}-${timeStr}`
    })

    let timeInterval = null
    let countdownInterval = null

    // 更新时间
    const updateTime = () => {
      const now = new Date()
      currentTime.value = now.toTimeString().split(' ')[0]
    }

    // 倒计时
    const startCountdown = () => {
      let hours = 18, minutes = 34, seconds = 15
      
      countdownInterval = setInterval(() => {
        seconds--
        if (seconds < 0) {
          seconds = 59
          minutes--
          if (minutes < 0) {
            minutes = 59
            hours--
            if (hours < 0) {
              hours = 0
            }
          }
        }
        
        countdown.value = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
      }, 1000)
    }

    // 生成通知编号
    const generateNotificationId = () => {
      const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'
      let id = 'REF-'
      for (let i = 0; i < 6; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length))
      }
      formData.notificationId = id
      return id
    }

    // 格式化日期时间
    const formatDateTime = (dateStr) => {
      if (!dateStr) return ''
      
      const date = new Date(dateStr)
      const year = date.getFullYear()
      const month = (date.getMonth() + 1).toString().padStart(2, '0')
      const day = date.getDate().toString().padStart(2, '0')
      const hours = date.getHours().toString().padStart(2, '0')
      const minutes = date.getMinutes().toString().padStart(2, '0')
      
      return `${year}-${month}-${day} ${hours}:${minutes}:00 UTC`
    }

    // 获取严重性样式类
    const getSeverityClass = (severity) => {
      switch (severity) {
        case 'low': return 'severity-low'
        case 'medium': return 'severity-medium'
        case 'high': return 'severity-high'
        case 'critical': return 'severity-critical'
        default: return 'severity-critical'
      }
    }

    // 获取严重性文本
    const getSeverityText = (severity) => {
      switch (severity) {
        case 'low': return '低'
        case 'medium': return '中'
        case 'high': return '高'
        case 'critical': return '严重'
        default: return '严重'
      }
    }

    // 获取原因代码描述
    const getReasonCodeDescription = (code) => {
      const descriptions = {
        '12.7a': '违反内容协议',
        '3.2b': '非法交易信息',
        '8.5c': '敏感关键词',
        '10.1d': '泄露个人隐私',
        '9.4e': '恶意软件链接',
        '6.3f': '平台规则违反',
        '15.8g': '加密签名失效',
        '7.1h': '节点拒绝转发',
        '11.3i': '量子密钥协商失败',
        '4.9j': '洋葱路由中断',
        '13.2k': 'AES-256解密错误',
        '5.6l': '匿名层级不足'
      }
      return descriptions[code] || '协议违规'
    }

    // 生成通知
    const generateNotification = () => {
      notificationData.notificationId = formData.notificationId || generateNotificationId()
      notificationData.username = formData.username || '匿名用户'
      notificationData.reason = formData.reason || '你的帖子包含受限制的内容或关键词，违反了当前节点的安全规定。'
      notificationData.postTitle = formData.postTitle || '未指定帖子标题'
      notificationData.postContent = formData.postContent || '没有可用的内容预览'
      notificationData.imageUrl = formData.imageUrl
      notificationData.severity = formData.severity
      notificationData.nodeId = formData.nodeId
      notificationData.reasonCode = formData.reasonCode
      
      if (formData.postDate) {
        notificationData.postDate = formatDateTime(formData.postDate)
      }
      
      if (formData.returnDate) {
        notificationData.returnDate = formatDateTime(formData.returnDate)
      }
    }

    // 提交操作
    const submitAction = () => {
      const username = formData.username || '匿名用户'
      const postTitle = formData.postTitle || '未指定帖子标题'
      const reason = formData.reason || '未提供退回原因'
      const imageUrl = formData.imageUrl
      const severity = formData.severity
      const reasonCode = formData.reasonCode
      
      let msg = `警告: 重新提交申请已发送至节点管理员\n`
      msg += `用户: ${username}\n`
      msg += `标题: ${postTitle}\n`
      msg += `严重性: ${getSeverityText(severity)}\n`
      msg += `原因代码: ${reasonCode}\n`
      msg += `退回原因: ${reason.substring(0, 80)}${reason.length > 80 ? '...' : ''}\n`
      
      if (imageUrl) {
        if (imageUrl.startsWith('data:image/')) {
          msg += `包含图片: ${uploadedImageName.value || '已上传图片'}\n`
        } else {
          msg += `包含图片: ${imageUrl.substring(0, 50)}${imageUrl.length > 50 ? '...' : ''}\n`
        }
      }
      
      msg += "\n注意: 你的IP地址可能不再受到保护！操作已记录。"
      
      alert(msg)
    }

    // 处理图片上传
    const handleImageUpload = (event) => {
      const file = event.target.files[0]
      if (file) {
        // 检查文件大小 (限制为5MB)
        if (file.size > 5 * 1024 * 1024) {
          alert('图片文件大小不能超过5MB')
          return
        }
        
        // 检查文件类型
        if (!file.type.startsWith('image/')) {
          alert('请选择有效的图片文件')
          return
        }
        
        uploadedImageName.value = file.name
        
        // 将图片转换为base64格式存储在内存中
        const reader = new FileReader()
        reader.onload = (e) => {
          formData.imageUrl = e.target.result
        }
        reader.readAsDataURL(file)
      }
    }
    
    // 移除图片
    const removeImage = () => {
      formData.imageUrl = ''
      uploadedImageName.value = ''
      // 清除文件输入框
      const fileInput = document.getElementById('image-upload')
      if (fileInput) {
        fileInput.value = ''
      }
    }

    // 分享通知链接
    const shareNotification = () => {
      try {
        // 创建精简的分享数据对象（排除图片数据以减少URL长度）
        const shareData = {
          id: notificationData.notificationId,
          user: notificationData.username,
          pd: notificationData.postDate,
          rd: notificationData.returnDate,
          node: notificationData.nodeId,
          sev: notificationData.severity,
          code: notificationData.reasonCode,
          reason: notificationData.reason,
          title: notificationData.postTitle,
          content: notificationData.postContent
          // 注意：不包含imageUrl以减少数据大小
        }
        
        // 压缩JSON字符串（移除空格）
        const jsonString = JSON.stringify(shareData)
        
        // 使用UTF-8安全的Base64编码方式
        const encodedData = btoa(unescape(encodeURIComponent(jsonString)))
        
        // 生成分享链接
        const shareUrl = `${window.location.origin}${window.location.pathname}?d=${encodedData}`
        
        // 检查URL长度
        if (shareUrl.length > 2000) {
          alert('警告: 分享数据过大，可能导致链接无法正常工作。\n建议减少帖子内容长度后重试。')
          return
        }
        
        // 复制到剪贴板
        navigator.clipboard.writeText(shareUrl).then(() => {
          alert('分享链接已复制到剪贴板！\n\n链接包含通知数据（不含图片），其他人可以通过此链接查看退回通知。\n\n注意: 图片需要重新上传。')
        }).catch(() => {
          // 如果剪贴板API不可用，显示链接让用户手动复制
          prompt('分享链接已生成，请复制以下链接:', shareUrl)
        })
      } catch (error) {
        alert('生成分享链接失败: ' + error.message)
      }
    }

    // 保存通知为图片
    const saveNotificationAsImage = async () => {
      try {
        // 动态导入html2canvas
        const html2canvas = (await import('html2canvas')).default
        
        // 获取通知容器元素
        const containerElement = document.querySelector('.container')
        if (!containerElement) {
          alert('未找到通知内容，无法保存图片')
          return
        }
        
        // 截图配置
        const canvas = await html2canvas(containerElement, {
          backgroundColor: '#0a0a0a',
          scale: 3,
          useCORS: true,
          allowTaint: true,
          logging: false,
          width: containerElement.offsetWidth,
          height: containerElement.offsetHeight,
          scrollX: 0,
          scrollY: 0,
          windowWidth: window.innerWidth,
          windowHeight: window.innerHeight,
          onclone: (clonedDoc) => {
            // 在克隆的文档中应用样式优化
            const clonedContainer = clonedDoc.querySelector('.container')
            if (clonedContainer) {
              clonedContainer.style.filter = 'brightness(2.5) contrast(2.0) saturate(1.5)'
              
              // 强制所有文本为白色并添加发光效果
              const allText = clonedContainer.querySelectorAll('*')
              allText.forEach(el => {
                if (el.textContent && el.textContent.trim()) {
                  el.style.color = '#ffffff !important'
                  el.style.textShadow = '0 0 10px #ffffff, 0 0 20px #ffffff, 0 0 30px #ffffff'
                }
              })
              
              // 特别处理绿色文字
              const greenTexts = clonedContainer.querySelectorAll('.value, .status-text, h1, h3')
              greenTexts.forEach(el => {
                el.style.color = '#00ffaa !important'
                el.style.textShadow = '0 0 15px #00ffaa, 0 0 25px #00ffaa, 0 0 35px #00ffaa'
              })
              
              // 增强标题
              const titles = clonedContainer.querySelectorAll('h1, h3')
              titles.forEach(el => {
                el.style.color = '#00ffaa !important'
                el.style.textShadow = '0 0 20px #00ffaa, 0 0 30px #00ffaa, 0 0 40px #00ffaa'
                el.style.fontWeight = 'bold'
              })
              
              // 增强警告框
              const warnings = clonedContainer.querySelectorAll('.warning')
              warnings.forEach(el => {
                el.style.backgroundColor = 'rgba(255, 0, 60, 0.3)'
                el.style.border = '2px solid #ff003c'
                el.style.boxShadow = '0 0 20px rgba(255, 0, 60, 0.5)'
              })
            }
          }
        })
        
        // 创建下载链接
        const link = document.createElement('a')
        link.download = `暗网退回通知_${notificationData.notificationId}_${new Date().toISOString().slice(0, 10)}.png`
        link.href = canvas.toDataURL('image/png', 1.0)
        
        // 触发下载
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
        
        alert('通知图片已保存到下载文件夹！')
      } catch (error) {
        alert('保存图片失败: ' + error.message)
        console.error('保存图片错误:', error)
      }
    }

    // 从URL参数加载分享的通知数据
    const loadSharedNotification = () => {
      try {
        const urlParams = new URLSearchParams(window.location.search)
        const encodedData = urlParams.get('d') || urlParams.get('data') // 兼容旧格式
        
        if (encodedData) {
          console.log('原始编码数据:', encodedData)
          
          // 解码数据
          let decodedData
          let decodeMethod = ''
          
          try {
            // 尝试新格式（UTF-8安全的Base64）
            const decoded = atob(encodedData)
            const unescaped = decodeURIComponent(escape(decoded))
            decodedData = JSON.parse(unescaped)
            decodeMethod = 'UTF-8安全格式'
          } catch (error1) {
            console.log('UTF-8安全格式解码失败:', error1.message)
            try {
              // 回退到直接Base64格式
              const decoded = atob(encodedData)
              decodedData = JSON.parse(decoded)
              decodeMethod = '直接Base64格式'
            } catch (error2) {
              console.log('直接Base64格式解码失败:', error2.message)
              try {
                // 最后回退到旧格式（Base64 + URI编码）
                const decoded = atob(encodedData)
                const uriDecoded = decodeURIComponent(decoded)
                decodedData = JSON.parse(uriDecoded)
                decodeMethod = '旧URI编码格式'
              } catch (error3) {
                console.log('所有解码方式都失败:', error3.message)
                throw new Error('无法解码分享数据')
              }
            }
          }
          
          console.log('解码成功，使用方法:', decodeMethod)
          console.log('解码后数据:', decodedData)
          
          // 验证数据完整性
          if (!decodedData || typeof decodedData !== 'object') {
            throw new Error('解码后的数据格式无效')
          }
          
          // 映射精简字段名到完整字段名
          const mappedData = {
            notificationId: decodedData.id || decodedData.notificationId || 'REF-SHARED',
            username: decodedData.user || decodedData.username || '分享用户',
            postDate: decodedData.pd || decodedData.postDate || new Date().toISOString().slice(0, 16),
            returnDate: decodedData.rd || decodedData.returnDate || new Date().toISOString().slice(0, 16),
            nodeId: decodedData.node || decodedData.nodeId || 'SHARED#NODE',
            severity: decodedData.sev || decodedData.severity || 'warning',
            reasonCode: decodedData.code || decodedData.reasonCode || '99.9',
            reason: decodedData.reason || '分享的退回通知',
            postTitle: decodedData.title || decodedData.postTitle || '分享的帖子',
            postContent: decodedData.content || decodedData.postContent || '分享的内容',
            imageUrl: decodedData.imageUrl || '' // 图片数据不包含在分享中
          }
          
          console.log('映射后数据:', mappedData)
          
          // 更新通知数据
          Object.assign(notificationData, mappedData)
          Object.assign(formData, mappedData)
          
          // 显示加载成功提示
          setTimeout(() => {
            alert(`已成功加载分享的退回通知数据！\n\n解码方式: ${decodeMethod}\n\n注意: 图片未包含在分享链接中，如需要请重新上传。`)
          }, 2000)
        }
      } catch (error) {
        console.error('加载分享数据详细错误:', error)
        console.error('错误堆栈:', error.stack)
        alert(`分享链接解析失败！\n\n错误信息: ${error.message}\n\n请检查链接是否完整或联系分享者重新生成链接。`)
      }
    }

    // 清除数据
    const clearData = () => {
      if (confirm('此操作将清除所有输入内容和通知。继续吗?')) {
        // 重置表单数据
        Object.assign(formData, {
          notificationId: 'REF-7X3B9P',
          severity: 'critical',
          nodeId: 'ONION#8FD7KX',
          reasonCode: '12.7a',
          postDate: '',
          returnDate: '',
          username: '',
          postTitle: '',
          imageUrl: '',
          postContent: '',
          reason: ''
        })
        
        // 清除上传的图片
        removeImage()
        
        // 重置通知数据
        Object.assign(notificationData, {
          notificationId: 'REF-7X3B9P',
          username: '匿名用户',
          postDate: '2023-08-26 05:31:17 UTC',
          returnDate: '2023-08-27 08:15:32 UTC',
          nodeId: 'ONION#8FD7KX',
          severity: 'critical',
          reasonCode: '12.7a',
          reason: '你的帖子包含受限制的内容或关键词，违反了当前节点的安全规定。',
          postTitle: '未指定帖子标题',
          postContent: '没有可用的内容预览',
          imageUrl: ''
        })
        
        alert('所有数据已清除。连接已重置。')
      }
    }

    // 销毁通知
    const selfDestruct = () => {
      if (confirm('此操作将销毁此通知并清除所有痕迹！继续吗?')) {
        isDestroyed.value = true
      }
    }

    // 创建新通知
    const createNewNotification = () => {
      isDestroyed.value = false
      clearData()
    }

    // 重新显示加载动画
    const showLoadingAnimation = () => {
      isLoading.value = true
      loadingProgress.value = 0
      loadingText.value = '初始化暗网连接...'
      terminalLines.value = []
      startLoadingAnimation()
    }

    // 显示路由详情
    const showRouteDetails = () => {
      showRouteVisualization.value = true
      // 模拟路由动画
      setTimeout(() => {
        routeNodes.value[2].failed = true
        routeStatus.value = {
          text: '节点#5GH32T拒绝转发',
          class: 'status-error'
        }
      }, 1000)
    }

    // 关闭路由可视化
    const closeRouteVisualization = () => {
      showRouteVisualization.value = false
    }

    // 显示加密详情
    const showEncryptionDetails = () => {
      showEncryptionStatus.value = true
      // 模拟加密过程
      let step = 0
      const interval = setInterval(() => {
        if (step < encryptionSteps.value.length) {
          encryptionSteps.value[step].active = true
          if (step > 0) {
            encryptionSteps.value[step - 1].active = false
          }
          step++
        } else {
          clearInterval(interval)
          encryptionStatusText.value = 'AES-256加密生效'
        }
      }, 1500)
    }

    // 关闭加密状态
    const closeEncryptionStatus = () => {
      showEncryptionStatus.value = false
    }

    // 检查黑市访问
    const checkMarketAccess = () => {
      marketClickCount.value++
      if (marketClickCount.value >= 3) {
        showDarkMarket.value = true
        marketClickCount.value = 0
        // 获取实时比特币汇率
        fetchBitcoinRate()
      }
    }

    // 关闭暗网黑市
    const closeDarkMarket = () => {
      showDarkMarket.value = false
    }

    // 获取比特币汇率
    const fetchBitcoinRate = async () => {
      try {
        // 模拟实时汇率（实际项目中可以调用真实API）
        const mockRate = 680000 + Math.random() * 20000
        btcToRmb.value = Math.floor(mockRate)
        
        // 更新商品人民币价格
        marketItems.value.forEach(item => {
          item.cnyPrice = Math.floor(item.btcPrice * btcToRmb.value)
        })
      } catch (error) {
        btcToRmb.value = 690000 // 默认汇率
        marketItems.value.forEach(item => {
          item.cnyPrice = Math.floor(item.btcPrice * 690000)
        })
      }
    }

    // 显示购买警告
    const showPurchaseWarning = (item) => {
      alert(`警告: 这是演示内容！\n\n商品: ${item.name}\n价格: ${item.btcPrice} BTC (≈ ¥${item.cnyPrice})\n\n此功能仅供展示，不涉及任何真实交易。`)
    }

    // 更新加密状态文本 - 使用固定宽度
    const updateEncryptionStatus = () => {
      const statuses = [
        'AES-256加密生效　　　　　',
        'RSA-4096签名验证中...　　',
        '量子密钥协商中...　　　　',
        '椭圆曲线加密已建立　　　　',
        '洋葱路由多层加密中...　　'
      ]
      let index = 0
      setInterval(() => {
        encryptionStatusText.value = statuses[index]
        index = (index + 1) % statuses.length
      }, 3500)
    }

    // 更新节点状态 - 使用固定宽度
    const updateNodeStatus = () => {
      const statuses = [
        '同步中　　',
        '转发中　　', 
        '验证中　　',
        '在线　　　',
        '重连中　　'
      ]
      let index = 0
      setInterval(() => {
        nodeStatusText.value = statuses[index]
        index = (index + 1) % statuses.length
      }, 4200)
    }

    // 更新路由状态和技术参数
    const updateRouteStatus = () => {
      const statusList = [
        { text: '节点#5GH32T拒绝转发', class: 'status-error' },
        { text: '重新建立电路连接', class: 'status-warning' },
        { text: '通过备用节点转发', class: 'status-success' },
        { text: '加密隧道已建立', class: 'status-success' },
        { text: '检测到网络拥塞', class: 'status-warning' }
      ]
      
      let statusIndex = 0
      setInterval(() => {
        routeStatus.value = statusList[statusIndex]
        statusIndex = (statusIndex + 1) % statusList.length
        
        // 动态更新技术参数
        latency.value = 650 + Math.floor(Math.random() * 400)
        packetSize.value = 32 + Math.floor(Math.random() * 96)
        hopCount.value = 3 + Math.floor(Math.random() * 3)
        circuitTime.value = parseFloat((2.1 + Math.random() * 4.5).toFixed(1))
        
        // 更新节点状态
        routeNodes.value.forEach((node, index) => {
          if (index === 2) { // 失败节点
            node.failed = Math.random() > 0.3
            node.active = !node.failed
            node.status = node.failed ? '拒绝转发' : '重新连接'
          } else if (index < routeNodes.value.length - 1) {
            node.active = Math.random() > 0.2
            const activeStatuses = ['转发中', '验证中', '加密中']
            const inactiveStatuses = ['等待中', '重连中', '超时']
            node.status = node.active ? 
              activeStatuses[Math.floor(Math.random() * activeStatuses.length)] :
              inactiveStatuses[Math.floor(Math.random() * inactiveStatuses.length)]
          }
        })
      }, 6000)
    }

    // 加载动画逻辑
    const startLoadingAnimation = () => {
      const commands = [
        { text: 'connecting to tor network...', delay: 1200 },
        { text: 'establishing encrypted tunnel...', delay: 1500 },
        { text: 'verifying node authenticity...', delay: 1000 },
        { text: 'loading security protocols...', delay: 1300 },
        { text: 'initializing anonymous session...', delay: 1600 },
        { text: 'darknet connection established', delay: 800 }
      ]
      
      const loadingTexts = [
        '初始化暗网连接...',
        '建立加密隧道...',
        '验证节点身份...',
        '加载安全协议...',
        '启动匿名会话...',
        '连接完成'
      ]
      
      let commandIndex = 0
      let progressValue = 0
      
      const addCommand = () => {
        if (commandIndex < commands.length) {
          const command = commands[commandIndex]
          terminalLines.value.push({ text: command.text, typing: true })
          
          setTimeout(() => {
            if (terminalLines.value[commandIndex]) {
              terminalLines.value[commandIndex].typing = false
            }
            commandIndex++
            
            if (commandIndex < commands.length) {
              setTimeout(addCommand, 500)  // 增加命令间隔时间
            }
          }, command.delay)
        }
      }
      
      const updateProgress = () => {
        const interval = setInterval(() => {
          progressValue += Math.random() * 8 + 2  // 减慢进度条速度
          if (progressValue > 100) progressValue = 100
          
          loadingProgress.value = Math.floor(progressValue)
          
          const textIndex = Math.floor((progressValue / 100) * loadingTexts.length)
          if (textIndex < loadingTexts.length) {
            loadingText.value = loadingTexts[textIndex]
          }
          
          if (progressValue >= 100) {
            clearInterval(interval)
            setTimeout(() => {
              isLoading.value = false
            }, 1500)  // 增加最终等待时间
          }
        }, 250)  // 增加更新间隔
      }
      
      addCommand()
      updateProgress()
    }

    // 初始化随机内容
    const initializeRandomContent = () => {
      const titles = [
        "加密市场分析报告",
        "安全通信协议讨论",
        "匿名网络基础设施研究",
        "数据泄露警告",
        "TOR节点最新配置"
      ]
      
      const contents = [
        "经过深入研究，我们发现了系统中的一个严重漏洞，可能影响用户的匿名性。请尽快检查你的设置...",
        "近期观察到某种新型攻击模式，专门针对暗网市场，涉及加密货币交易的安全性。请注意保护你的资金...",
        "我们获取了一份关于某个政府机构监视活动的文件，该文件详细描述了他们的监控技术和方法论...",
        "有消息称某些供应商正在提供伪劣商品，建议用户在交易前进行更彻底的验证。",
        "本次泄露的数据包含大量敏感信息，我们已经采取了必要的保护措施。",
        "新的加密算法证明，现有的通信协议可能不再安全。我们正在开发替代方案..."
      ]
      
      formData.postTitle = titles[Math.floor(Math.random() * titles.length)]
      formData.postContent = contents[Math.floor(Math.random() * contents.length)]
      
      // 设置默认日期
      const now = new Date()
      const tomorrow = new Date()
      tomorrow.setDate(now.getDate() + 1)
      
      formData.postDate = now.toISOString().slice(0, 16)
      formData.returnDate = tomorrow.toISOString().slice(0, 16)
    }

    onMounted(() => {
      startLoadingAnimation()
      updateTime()
      startCountdown()
      initializeRandomContent()
      updateEncryptionStatus()
      updateNodeStatus()
      updateRouteStatus()
      loadSharedNotification()
      
      timeInterval = setInterval(updateTime, 1000)
      
      // 移动端优化
      const preventZoom = (e) => {
        if (e.touches.length > 1) {
          e.preventDefault()
        }
      }
      
      const preventDoubleClickZoom = (e) => {
        e.preventDefault()
        e.target.click()
      }
      
      // 防止双指缩放
      document.addEventListener('touchstart', preventZoom, { passive: false })
      document.addEventListener('touchmove', preventZoom, { passive: false })
      
      // 防止双击缩放（仅在移动设备上）
      if ('ontouchstart' in window) {
        document.addEventListener('dblclick', preventDoubleClickZoom)
      }
      
      // 防止iOS Safari的橡皮筋效果
      document.body.addEventListener('touchmove', (e) => {
        if (e.target === document.body) {
          e.preventDefault()
        }
      }, { passive: false })
    })

    onUnmounted(() => {
      if (timeInterval) clearInterval(timeInterval)
      if (countdownInterval) clearInterval(countdownInterval)
    })

    return {
      currentTime,
      countdown,
      isDestroyed,
      isLoading,
      loadingProgress,
      loadingText,
      terminalLines,
      formData,
      notificationData,
      recordId,
      generateNotificationId,
      getSeverityClass,
      getSeverityText,
      getReasonCodeDescription,
      generateNotification,
      submitAction,
      clearData,
      selfDestruct,
      createNewNotification,
      showLoadingAnimation,
      showRouteVisualization,
      showEncryptionStatus,
      showDarkMarket,
      encryptionStatusText,
      nodeStatusText,
      encryptionLayers,
      latency,
      packetSize,
      hopCount,
      anonymityLevel,
      circuitTime,
      fingerprintObfuscation,
      uploadedImageName,
      
      routeNodes,
      routeStatus,
      encryptionSteps,
      marketItems,
      showRouteDetails,
      closeRouteVisualization,
      showEncryptionDetails,
      closeEncryptionStatus,
      checkMarketAccess,
      closeDarkMarket,
      showPurchaseWarning,
      handleImageUpload,
      removeImage,
      shareNotification,
      saveNotificationAsImage
    }
  }
}
</script>