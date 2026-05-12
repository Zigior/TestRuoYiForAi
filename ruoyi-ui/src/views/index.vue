<template>
  <div class="app-container home-page">
    <!-- 欢迎横幅 -->
    <div class="welcome-banner">
      <div class="banner-content">
        <div class="banner-text">
          <h1 class="greeting">{{ greeting }}, {{ nickName }}</h1>
          <p class="date-text">{{ currentDate }} {{ currentWeekday }}</p>
          <p class="subtitle">欢迎使用知识管理系统，您可以管理文档、分类，并与 AI 进行智能对话</p>
        </div>
        <div class="banner-illustration">
          <i class="el-icon-s-cooperation banner-icon"></i>
        </div>
      </div>
    </div>

    <!-- 统计卡片 -->
    <el-row :gutter="20" class="stat-row">
      <el-col :xs="12" :sm="12" :md="6" v-for="stat in stats" :key="stat.label">
        <div class="stat-card" :style="{ borderLeftColor: stat.color }">
          <div class="stat-info">
            <div class="stat-value">{{ stat.value }}</div>
            <div class="stat-label">{{ stat.label }}</div>
          </div>
          <div class="stat-icon" :style="{ color: stat.color }">
            <i :class="stat.icon"></i>
          </div>
        </div>
      </el-col>
    </el-row>

    <!-- 快捷入口 -->
    <el-row :gutter="20" class="quick-actions">
      <el-col :span="24">
        <h3 class="section-title">
          <i class="el-icon-s-management"></i> 快捷入口
        </h3>
      </el-col>
      <el-col :xs="24" :sm="8">
        <div class="action-card" @click="goTo('/km/category')">
          <div class="action-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
            <i class="el-icon-folder-opened"></i>
          </div>
          <div class="action-info">
            <h4>知识分类</h4>
            <p>管理文档分类结构</p>
          </div>
          <i class="el-icon-arrow-right action-arrow"></i>
        </div>
      </el-col>
      <el-col :xs="24" :sm="8">
        <div class="action-card" @click="goTo('/km/document')">
          <div class="action-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
            <i class="el-icon-document"></i>
          </div>
          <div class="action-info">
            <h4>知识文档</h4>
            <p>上传和管理知识文档</p>
          </div>
          <i class="el-icon-arrow-right action-arrow"></i>
        </div>
      </el-col>
      <el-col :xs="24" :sm="8">
        <div class="action-card" @click="goTo('/chat')">
          <div class="action-icon" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
            <i class="el-icon-chat-dot-round"></i>
          </div>
          <div class="action-info">
            <h4>AI 对话</h4>
            <p>基于知识库智能问答</p>
          </div>
          <i class="el-icon-arrow-right action-arrow"></i>
        </div>
      </el-col>
    </el-row>

    <!-- 底部信息 -->
    <div class="footer-info">
      <el-divider content-position="center">
        <i class="el-icon-info"></i> 提示
      </el-divider>
      <el-alert
        title="上传文档后即可在 AI 对话中使用知识库内容作为参考上下文"
        type="info"
        :closable="false"
        show-icon
      />
    </div>
  </div>
</template>

<script>
import { listDocument } from "@/api/system/km/document"
import { treeSelect } from "@/api/system/km/category"
import { listSessions } from "@/api/system/km/chat"
import { mapGetters } from "vuex"

export default {
  name: "Index",
  data() {
    return {
      currentDate: "",
      currentWeekday: "",
      greeting: "",
      stats: [
        { label: "知识分类", value: 0, icon: "el-icon-folder-opened", color: "#667eea" },
        { label: "知识文档", value: 0, icon: "el-icon-document", color: "#f5576c" },
        { label: "对话会话", value: 0, icon: "el-icon-chat-dot-round", color: "#4facfe" },
        { label: "系统版本", value: "V3.9.2", icon: "el-icon-s-platform", color: "#43e97b" }
      ],
      timer: null
    }
  },
  computed: {
    ...mapGetters(["nickName"])
  },
  created() {
    this.updateTime()
    this.timer = setInterval(this.updateTime, 60000)
    this.loadStats()
  },
  beforeDestroy() {
    clearInterval(this.timer)
  },
  methods: {
    updateTime() {
      const now = new Date()
      const year = now.getFullYear()
      const month = String(now.getMonth() + 1).padStart(2, "0")
      const day = String(now.getDate()).padStart(2, "0")
      const weekdays = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"]
      this.currentDate = `${year}年${month}月${day}日`
      this.currentWeekday = weekdays[now.getDay()]
      const hour = now.getHours()
      if (hour < 6) this.greeting = "夜深了"
      else if (hour < 9) this.greeting = "早上好"
      else if (hour < 12) this.greeting = "上午好"
      else if (hour < 14) this.greeting = "中午好"
      else if (hour < 18) this.greeting = "下午好"
      else this.greeting = "晚上好"
    },
    loadStats() {
      treeSelect().then(res => {
        this.stats[0].value = res.data ? res.data.length : 0
      }).catch(() => {})
      listDocument({ pageSize: 1 }).then(res => {
        this.stats[1].value = res.total || 0
      }).catch(() => {})
      listSessions().then(res => {
        this.stats[2].value = res.rows ? res.rows.length : res.data ? res.data.length : 0
      }).catch(() => {})
    },
    goTo(path) {
      this.$router.push(path)
    }
  }
}
</script>

<style scoped lang="scss">
.home-page {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.welcome-banner {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 16px;
  padding: 36px 40px;
  margin-bottom: 24px;
  color: #fff;
  position: relative;
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(102, 126, 234, 0.3);

  &::before {
    content: "";
    position: absolute;
    top: -50%;
    right: -20%;
    width: 400px;
    height: 400px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.06);
  }

  &::after {
    content: "";
    position: absolute;
    bottom: -30%;
    right: 10%;
    width: 300px;
    height: 300px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.04);
  }

  .banner-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
    z-index: 1;
  }

  .greeting {
    font-size: 28px;
    font-weight: 600;
    margin: 0 0 8px 0;
    letter-spacing: 1px;
  }

  .date-text {
    font-size: 15px;
    opacity: 0.85;
    margin: 0 0 12px 0;
  }

  .subtitle {
    font-size: 14px;
    opacity: 0.7;
    margin: 0;
  }

  .banner-illustration {
    flex-shrink: 0;
    margin-left: 20px;
  }

  .banner-icon {
    font-size: 72px;
    opacity: 0.3;
  }
}

.stat-row {
  margin-bottom: 24px;
}

.stat-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-left: 4px solid;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  transition: transform 0.3s, box-shadow 0.3s;
  margin-bottom: 16px;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 6px 24px rgba(0, 0, 0, 0.1);
  }

  .stat-value {
    font-size: 28px;
    font-weight: 700;
    color: #333;
    line-height: 1.2;
  }

  .stat-label {
    font-size: 14px;
    color: #999;
    margin-top: 4px;
  }

  .stat-icon {
    font-size: 38px;
    opacity: 0.2;
  }
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin: 0 0 16px 0;
  display: flex;
  align-items: center;

  i {
    margin-right: 8px;
    font-size: 20px;
  }
}

.quick-actions {
  margin-bottom: 24px;
}

.action-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
  margin-bottom: 16px;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 6px 24px rgba(0, 0, 0, 0.1);

    .action-arrow {
      transform: translateX(4px);
      opacity: 1;
    }
  }

  .action-icon {
    width: 52px;
    height: 52px;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    margin-right: 16px;

    i {
      font-size: 26px;
      color: #fff;
    }
  }

  .action-info {
    flex: 1;
    min-width: 0;

    h4 {
      font-size: 16px;
      font-weight: 600;
      color: #333;
      margin: 0 0 4px 0;
    }

    p {
      font-size: 13px;
      color: #999;
      margin: 0;
    }
  }

  .action-arrow {
    font-size: 18px;
    color: #ccc;
    transition: all 0.3s;
    opacity: 0.6;
  }
}

.footer-info {
  .el-divider__text {
    padding: 0 12px;
    color: #999;
    font-size: 13px;
  }
}
</style>
