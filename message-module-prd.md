# 宠物社交App消息模块接入方案 PRD

## 1. 项目概述

### 1.1 项目名称
宠物社交App消息模块接入腾讯云IM

### 1.2 项目版本
v1.0

### 1.3 文档修订记录
| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | 2024-01-20 | - | 初始版本 |

## 2. 项目背景

### 2.1 业务背景
宠物社交App作为连接宠物主人的社交平台，需要建立完善的消息通信体系，包括：
- 平台到用户的活动消息推送
- 用户之间的即时通讯
- 未来扩展的社群功能

### 2.2 问题描述
当前存在的问题：
- 缺乏统一的消息推送管理平台
- 用户之间无法进行实时沟通
- 活动信息触达率低
- 无法追溯历史推送记录

### 2.3 机会分析
通过集成完善的消息系统：
- 提升用户活跃度和留存率
- 增强社交属性，建立用户粘性
- 提高运营活动的转化率
- 为商业化提供基础设施

## 3. 项目目标

### 3.1 业务目标
- 建立完整的消息推送体系，支持多渠道触达
- 实现用户间即时通讯功能，促进社交互动
- 提供消息数据分析能力，优化运营策略
- 为二期群聊功能预留扩展性

### 3.2 技术目标
- 接入腾讯云IM，实现稳定的即时通讯服务
- 集成极光推送，实现多渠道消息推送
- 建立消息中心，统一管理各类消息
- 实现消息记录的持久化和可追溯

### 3.3 成功指标
| 指标名称 | 目标值 | 衡量方式 |
|----------|--------|----------|
| 消息送达率 | >95% | 推送成功数/推送总数 |
| 消息延迟 | <2s | 端到端消息传输时间 |
| 系统可用性 | >99.9% | 正常运行时间/总时间 |
| 日活跃消息用户 | >30% | 发送消息用户数/DAU |

## 4. 需求范围

### 4.1 功能需求

#### 4.1.1 系统消息模块

##### 后台管理功能
- **功能描述**：
  - 创建和编辑活动消息
  - 多渠道选择（站内信、App Push、短信）
  - 定时发送设置
  - 目标用户筛选（全部用户、批量指定）
  - 推送任务管理和状态监控

- **用户场景**：
  运营人员在后台创建双十一活动推送，选择通过App Push和站内信渠道，设置在活动开始前1小时发送给所有活跃用户。

- **业务规则**：
  - 同一用户24小时内最多接收3条营销类推送
  - 短信推送需要用户授权同意
  - 定时任务最早可提前30天设置
  - 批量指定用户上限10万

##### 推送记录管理
- **功能描述**：
  - 推送历史记录查询
  - 推送效果统计（送达率、点击率）
  - 失败原因分析
  - 推送内容存档

- **业务规则**：
  - 推送记录保留180天
  - 敏感信息需脱敏展示
  - 支持按时间、渠道、状态筛选

#### 4.1.2 用户私信模块

##### 即时通讯基础功能
- **功能描述**：
  - 发送文本消息
  - 发送图片消息（支持jpg、png、gif，单张最大5MB）
  - 发送视频消息（支持mp4，单个最大50MB）
  - 消息已读/未读状态
  - 消息撤回（2分钟内）
  - 离线消息接收

- **用户场景**：
  用户A在宠物详情页点击"私信"按钮，向宠物主人B发送消息询问宠物情况。

- **业务规则**：
  - 未互相关注的用户，在对方回复前只能发送1条消息
  - 消息内容需通过敏感词过滤
  - 图片和视频需通过内容安全审核
  - 单个会话最多保存最近1000条消息

##### 会话管理
- **功能描述**：
  - 会话列表展示
  - 未读消息数显示
  - 会话置顶
  - 会话删除
  - 会话搜索

- **界面原型**：
  ```
  [会话列表页]
  ┌─────────────────────┐
  │  消息               │
  ├─────────────────────┤
  │ 🔍 搜索             │
  ├─────────────────────┤
  │ [头像] 用户昵称  2分钟前│
  │ 最新消息内容...   ②   │
  ├─────────────────────┤
  │ [头像] 用户昵称  昨天  │
  │ [图片]              │
  └─────────────────────┘
  ```

### 4.2 非功能需求

#### 4.2.1 性能需求
- 消息发送延迟 < 2秒（同城网络环境）
- 支持10万用户同时在线
- 单用户QPS限制：10条/秒
- 消息推送并发能力：1000条/秒

#### 4.2.2 安全需求
- 所有消息传输使用TLS加密
- 敏感内容实时过滤
- 用户举报机制
- 消息内容加密存储
- 防止消息轰炸攻击

#### 4.2.3 兼容性需求
- iOS 12.0+
- Android 7.0+
- 主流浏览器最新版本
- 支持移动网络和WiFi环境

## 5. 技术方案

### 5.1 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                          前端应用层                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │   Vue3 C端   │    │  管理后台   │    │   B端应用   │    │
│  │  +Capacitor  │    │   (Vue3)    │    │   (Vue3)    │    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │
└─────────┼───────────────────┼───────────────────┼───────────┘
          │                   │                   │
          ├───────────────────┴───────────────────┤
          │                                       │
┌─────────▼───────────────────────────────────────▼───────────┐
│                         API网关层                            │
│                    (Spring Cloud Gateway)                    │
└─────────┬───────────────────────────────────────┬───────────┘
          │                                       │
┌─────────▼───────────────────────────────────────▼───────────┐
│                        业务服务层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ 消息中心服务  │  │   IM服务     │  │  推送服务    │     │
│  │ (Message     │  │ (IM Service) │  │ (Push       │     │
│  │  Center)     │  │              │  │  Service)   │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐     │
│  │ 用户服务     │  │ 内容审核服务  │  │  统计服务    │     │
│  │ (User       │  │ (Content     │  │ (Analytics  │     │
│  │  Service)   │  │  Moderation) │  │  Service)   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
          │                                       │
┌─────────▼───────────────────────────────────────▼───────────┐
│                        中间件层                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   RabbitMQ   │  │    Redis     │  │    MySQL     │     │
│  │  消息队列    │  │  缓存/会话   │  │   数据库     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
          │                                       │
┌─────────▼───────────────────────────────────────▼───────────┐
│                      第三方服务层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  腾讯云IM    │  │   极光推送   │  │  极光短信    │     │
│  │  (Tencent   │  │  (JPush)     │  │ (JSMS)      │     │
│  │   Cloud IM)  │  │              │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 技术选型

| 技术领域 | 选择方案 | 选择理由 |
|----------|----------|----------|
| 即时通讯 | 腾讯云IM | 稳定性高、功能完善、SDK成熟、支持亿级并发 |
| 推送服务 | 极光推送 | 多渠道支持、到达率高、统计功能完善 |
| 消息队列 | RabbitMQ | 已有技术栈、支持延迟队列、可靠性高 |
| 缓存方案 | Redis | 高性能、支持多种数据结构、可做分布式锁 |
| 内容审核 | 腾讯云内容安全 | 与IM配套、审核准确率高、支持多种内容类型 |
| 对象存储 | 腾讯云COS | 与IM集成方便、CDN加速、成本可控 |

### 5.3 数据库设计

#### 5.3.1 系统消息相关表

```sql
-- 推送任务表
CREATE TABLE `push_task` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `task_name` varchar(100) NOT NULL COMMENT '任务名称',
  `task_type` tinyint NOT NULL COMMENT '任务类型: 1-活动消息',
  `content` text NOT NULL COMMENT '消息内容JSON',
  `channels` varchar(50) NOT NULL COMMENT '推送渠道: PUSH,SMS,INBOX',
  `target_type` tinyint NOT NULL COMMENT '目标类型: 1-全部, 2-指定用户',
  `target_users` longtext COMMENT '目标用户ID列表JSON',
  `scheduled_time` datetime DEFAULT NULL COMMENT '定时发送时间',
  `status` tinyint NOT NULL DEFAULT '0' COMMENT '状态: 0-待发送,1-发送中,2-已完成,3-已取消',
  `creator_id` bigint NOT NULL COMMENT '创建人ID',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_scheduled_time` (`scheduled_time`),
  KEY `idx_status` (`status`)
) COMMENT='推送任务表';

-- 推送记录表
CREATE TABLE `push_record` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `task_id` bigint NOT NULL COMMENT '任务ID',
  `user_id` bigint NOT NULL COMMENT '用户ID',
  `channel` varchar(20) NOT NULL COMMENT '推送渠道',
  `status` tinyint NOT NULL COMMENT '状态: 0-待发送,1-已发送,2-已送达,3-失败',
  `failure_reason` varchar(500) DEFAULT NULL COMMENT '失败原因',
  `send_time` datetime DEFAULT NULL COMMENT '发送时间',
  `reach_time` datetime DEFAULT NULL COMMENT '送达时间',
  `click_time` datetime DEFAULT NULL COMMENT '点击时间',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_task_id` (`task_id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_send_time` (`send_time`)
) COMMENT='推送记录表';

-- 站内信表
CREATE TABLE `inbox_message` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` bigint NOT NULL COMMENT '用户ID',
  `type` tinyint NOT NULL COMMENT '类型: 1-系统消息,2-活动消息',
  `title` varchar(200) NOT NULL COMMENT '标题',
  `content` text NOT NULL COMMENT '内容',
  `link_url` varchar(500) DEFAULT NULL COMMENT '跳转链接',
  `is_read` tinyint NOT NULL DEFAULT '0' COMMENT '是否已读',
  `read_time` datetime DEFAULT NULL COMMENT '阅读时间',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_user_id_read` (`user_id`,`is_read`),
  KEY `idx_create_time` (`create_time`)
) COMMENT='站内信表';
```

#### 5.3.2 IM相关表

```sql
-- IM用户映射表
CREATE TABLE `im_user_mapping` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` bigint NOT NULL COMMENT '系统用户ID',
  `im_user_id` varchar(50) NOT NULL COMMENT '腾讯云IM用户ID',
  `im_user_sig` text COMMENT '用户签名',
  `sig_expire_time` datetime DEFAULT NULL COMMENT '签名过期时间',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '状态: 1-正常,0-禁用',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_id` (`user_id`),
  UNIQUE KEY `uk_im_user_id` (`im_user_id`)
) COMMENT='IM用户映射表';

-- 私信限制记录表
CREATE TABLE `chat_restriction` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `sender_id` bigint NOT NULL COMMENT '发送者ID',
  `receiver_id` bigint NOT NULL COMMENT '接收者ID',
  `message_count` int NOT NULL DEFAULT '0' COMMENT '已发送消息数',
  `is_replied` tinyint NOT NULL DEFAULT '0' COMMENT '是否已回复',
  `first_message_time` datetime NOT NULL COMMENT '首次消息时间',
  `reply_time` datetime DEFAULT NULL COMMENT '回复时间',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_sender_receiver` (`sender_id`,`receiver_id`),
  KEY `idx_receiver_id` (`receiver_id`)
) COMMENT='私信限制记录表';

-- 消息审核记录表
CREATE TABLE `message_audit` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  `message_id` varchar(100) NOT NULL COMMENT '消息ID',
  `sender_id` bigint NOT NULL COMMENT '发送者ID',
  `receiver_id` bigint NOT NULL COMMENT '接收者ID',
  `content_type` tinyint NOT NULL COMMENT '内容类型: 1-文本,2-图片,3-视频',
  `content` text COMMENT '内容或URL',
  `audit_status` tinyint NOT NULL COMMENT '审核状态: 0-待审核,1-通过,2-违规',
  `audit_result` json DEFAULT NULL COMMENT '审核结果详情',
  `audit_time` datetime DEFAULT NULL COMMENT '审核时间',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_message_id` (`message_id`),
  KEY `idx_sender_id` (`sender_id`),
  KEY `idx_audit_status` (`audit_status`)
) COMMENT='消息审核记录表';
```

### 5.4 接口设计

#### 5.4.1 系统消息接口

```yaml
# 创建推送任务
POST /api/v1/push/task
Request:
{
  "taskName": "双十一活动推送",
  "taskType": 1,
  "content": {
    "title": "双十一大促开始啦",
    "body": "全场宠物用品5折起",
    "linkUrl": "app://activity/double11"
  },
  "channels": ["PUSH", "INBOX"],
  "targetType": 1,
  "targetUsers": [],
  "scheduledTime": "2024-11-11 00:00:00"
}

Response:
{
  "code": 0,
  "message": "success",
  "data": {
    "taskId": 12345
  }
}

# 查询推送记录
GET /api/v1/push/records?taskId=12345&page=1&size=20
Response:
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100000,
    "records": [
      {
        "userId": 10001,
        "channel": "PUSH",
        "status": 2,
        "sendTime": "2024-11-11 00:00:01",
        "reachTime": "2024-11-11 00:00:02"
      }
    ]
  }
}
```

#### 5.4.2 IM相关接口

```yaml
# 获取IM登录凭证
GET /api/v1/im/credential
Response:
{
  "code": 0,
  "message": "success",
  "data": {
    "sdkAppId": 1400000000,
    "userId": "user_10001",
    "userSig": "eJyrVkosLVayUsrPSU1MTsksSwQz01Iy..."
  }
}

# 发送私信前检查限制
POST /api/v1/im/check-restriction
Request:
{
  "receiverId": 10002
}

Response:
{
  "code": 0,
  "message": "success",
  "data": {
    "canSend": true,
    "remainingCount": 1,
    "restrictionType": "NOT_FOLLOWING"
  }
}

# 上报消息已读
POST /api/v1/im/mark-read
Request:
{
  "conversationId": "C2C_user_10002",
  "lastMessageSeq": 12345
}

# 获取会话列表
GET /api/v1/im/conversations?page=1&size=20
Response:
{
  "code": 0,
  "message": "success", 
  "data": {
    "total": 10,
    "conversations": [
      {
        "conversationId": "C2C_user_10002",
        "userId": 10002,
        "nickname": "宠物爱好者",
        "avatar": "https://xxx.jpg",
        "lastMessage": {
          "content": "你家猫咪好可爱",
          "timestamp": 1705750000,
          "type": "text"
        },
        "unreadCount": 2
      }
    ]
  }
}
```

### 5.5 部署方案

#### 5.5.1 服务部署架构

```
生产环境部署架构：

┌─────────────────────────────────────────────────┐
│                   负载均衡                       │
│              (阿里云SLB/腾讯云CLB)              │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────┐
│                  API网关集群                     │
│         (2台, Spring Cloud Gateway)             │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────┐
│                微服务集群                        │
│  ┌──────────────┐  ┌──────────────┐           │
│  │消息中心服务   │  │   IM服务     │           │
│  │  (2实例)     │  │  (2实例)     │           │
│  └──────────────┘  └──────────────┘           │
│  ┌──────────────┐  ┌──────────────┐           │
│  │  推送服务     │  │  用户服务    │           │
│  │  (2实例)     │  │  (2实例)     │           │
│  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────┐
│                 数据层                          │
│  ┌──────────────┐  ┌──────────────┐           │
│  │ MySQL主从    │  │ Redis集群    │           │
│  │  (1主2从)    │  │ (3主3从)     │           │
│  └──────────────┘  └──────────────┘           │
│  ┌──────────────┐                              │
│  │ RabbitMQ集群 │                              │
│  │  (3节点)     │                              │
│  └──────────────┘                              │
└─────────────────────────────────────────────────┘
```

#### 5.5.2 部署流程

1. **基础设施准备**
   - 申请腾讯云IM应用
   - 配置极光推送应用
   - 准备服务器资源

2. **中间件部署**
   - 部署MySQL主从集群
   - 部署Redis集群
   - 部署RabbitMQ集群

3. **应用服务部署**
   - 构建Docker镜像
   - 使用K8s或Docker Compose编排
   - 配置服务发现和负载均衡

4. **监控告警配置**
   - 配置Prometheus + Grafana
   - 设置关键指标告警
   - 配置日志收集(ELK)

## 6. 项目计划

### 6.1 里程碑

| 阶段 | 开始时间 | 结束时间 | 交付物 |
|------|----------|----------|--------|
| 需求评审与技术方案设计 | 2024-01-22 | 2024-01-26 | 需求文档、技术方案 |
| 基础架构搭建 | 2024-01-29 | 2024-02-02 | 中间件部署、第三方服务接入 |
| 系统消息模块开发 | 2024-02-05 | 2024-02-16 | 推送管理后台、多渠道推送功能 |
| IM集成开发 | 2024-02-19 | 2024-03-01 | IM SDK集成、私信功能 |
| 联调测试 | 2024-03-04 | 2024-03-15 | 功能测试、性能测试 |
| 上线准备 | 2024-03-18 | 2024-03-22 | 部署文档、运维手册 |
| 灰度发布 | 2024-03-25 | 2024-03-29 | 5%用户灰度 |
| 全量发布 | 2024-04-01 | 2024-04-01 | 全量上线 |

### 6.2 资源需求

**人力资源**：
- 后端开发：2人
- 前端开发：2人（C端1人，管理后台1人）
- 测试：1人
- 产品经理：1人
- 项目经理：1人

**硬件资源**：
- 应用服务器：8核16G * 8台
- 数据库服务器：16核32G * 3台
- 缓存服务器：8核16G * 6台
- 消息队列服务器：8核16G * 3台

**第三方服务**：
- 腾讯云IM专业版
- 极光推送VIP套餐
- 腾讯云内容安全
- 腾讯云COS存储

## 7. 风险评估

### 7.1 技术风险

| 风险描述 | 影响程度 | 发生概率 | 应对措施 |
|----------|----------|----------|----------|
| 腾讯云IM服务不稳定 | 高 | 低 | 1. 实现消息本地缓存队列<br>2. 准备降级方案<br>3. 监控告警及时发现 |
| 消息审核误判率高 | 中 | 中 | 1. 建立白名单机制<br>2. 人工复审申诉通道<br>3. 持续优化审核规则 |
| 推送到达率不达标 | 中 | 中 | 1. 多渠道互补推送<br>2. 优化推送时机<br>3. 建立推送效果监控 |
| 高并发性能瓶颈 | 高 | 低 | 1. 做好压测和容量规划<br>2. 实现弹性伸缩<br>3. 优化数据库和缓存 |

### 7.2 业务风险

| 风险描述 | 影响程度 | 发生概率 | 应对措施 |
|----------|----------|----------|----------|
| 用户隐私泄露 | 高 | 低 | 1. 严格权限控制<br>2. 敏感信息加密<br>3. 安全审计日志 |
| 垃圾信息骚扰 | 中 | 高 | 1. 私信限制机制<br>2. 举报处理流程<br>3. 黑名单功能 |
| 运营成本超预算 | 中 | 中 | 1. 设置用量告警<br>2. 优化资源使用<br>3. 定期成本评估 |

## 8. 附录

### 8.1 参考文档
- [腾讯云即时通信 IM 产品文档](https://cloud.tencent.com/document/product/269)
- [腾讯云 IM Web SDK 开发指南](https://cloud.tencent.com/document/product/269/64506)
- [极光推送开发文档](https://docs.jiguang.cn/jpush/guideline/intro)
- [Spring Cloud Gateway 官方文档](https://spring.io/projects/spring-cloud-gateway)

### 8.2 术语表

| 术语 | 说明 |
|------|------|
| IM | Instant Messaging，即时通讯 |
| SDK | Software Development Kit，软件开发工具包 |
| UserSig | 腾讯云IM的用户登录凭证 |
| Push | 应用推送通知 |
| SLB | Server Load Balancer，服务器负载均衡 |
| QPS | Queries Per Second，每秒查询率 |
| DAU | Daily Active Users，日活跃用户数 |
| C2C | Client to Client，用户到用户的通信 |
| COS | Cloud Object Storage，云对象存储 |

### 8.3 接入流程详解

#### 8.3.1 腾讯云IM接入步骤

1. **创建应用**
   - 登录腾讯云控制台
   - 创建IM应用，获取SDKAppID
   - 配置回调地址

2. **服务端集成**
   ```java
   // 1. 添加依赖
   <dependency>
       <groupId>com.tencentcloudapi</groupId>
       <artifactId>tencentcloud-sdk-java</artifactId>
       <version>3.1.270</version>
   </dependency>

   // 2. 生成UserSig
   public class IMService {
       @Value("${tencent.im.sdkAppId}")
       private long sdkAppId;
       
       @Value("${tencent.im.secretKey}")
       private String secretKey;
       
       public String generateUserSig(String userId) {
           return TLSSigAPIv2.genSig(sdkAppId, secretKey, userId, 86400);
       }
   }
   ```

3. **前端集成**
   ```javascript
   // 1. 安装SDK
   npm install tim-js-sdk --save

   // 2. 初始化
   import TIM from 'tim-js-sdk';
   
   const tim = TIM.create({
     SDKAppID: 1400000000
   });
   
   // 3. 登录
   tim.login({
     userID: 'user_10001',
     userSig: 'xxx'
   });
   ```

4. **消息监听与发送**
   ```javascript
   // 监听消息
   tim.on(TIM.EVENT.MESSAGE_RECEIVED, (event) => {
     const messageList = event.data;
     messageList.forEach((message) => {
       console.log('收到消息:', message);
     });
   });

   // 发送消息
   const message = tim.createTextMessage({
     to: 'user_10002',
     conversationType: TIM.TYPES.CONV_C2C,
     payload: {
       text: '你好'
     }
   });
   
   tim.sendMessage(message);
   ```

#### 8.3.2 极光推送接入步骤

1. **后端集成**
   ```java
   // 1. 添加依赖
   <dependency>
       <groupId>cn.jpush.api</groupId>
       <artifactId>jpush-client</artifactId>
       <version>3.6.6</version>
   </dependency>

   // 2. 推送实现
   @Service
   public class JPushService {
       @Value("${jpush.appKey}")
       private String appKey;
       
       @Value("${jpush.masterSecret}")
       private String masterSecret;
       
       private JPushClient jPushClient;
       
       @PostConstruct
       public void init() {
           jPushClient = new JPushClient(masterSecret, appKey);
       }
       
       public void pushToUsers(List<String> userIds, String title, String content) {
           PushPayload payload = PushPayload.newBuilder()
               .setPlatform(Platform.all())
               .setAudience(Audience.alias(userIds))
               .setNotification(Notification.newBuilder()
                   .setAlert(content)
                   .addPlatformNotification(AndroidNotification.newBuilder()
                       .setTitle(title)
                       .build())
                   .addPlatformNotification(IosNotification.newBuilder()
                       .incrBadge(1)
                       .build())
                   .build())
               .build();
               
           jPushClient.sendPush(payload);
       }
   }
   ```

2. **前端集成（Capacitor）**
   ```javascript
   // 1. 安装插件
   npm install jpush-phonegap-plugin --save
   npm install @jiguang-ionic/jpush --save

   // 2. 初始化
   import { JPush } from '@jiguang-ionic/jpush';
   
   JPush.init();
   JPush.setAlias({ 
     sequence: 1, 
     alias: 'user_10001' 
   });
   
   // 3. 监听推送
   document.addEventListener('jpush.receiveNotification', (event) => {
     console.log('收到推送:', event);
   });
   ```

这份PRD文档详细描述了宠物社交App消息模块的需求和技术实现方案，包括系统架构、数据库设计、接口设计等关键内容。可以根据实际开发过程中的反馈进行调整和优化。