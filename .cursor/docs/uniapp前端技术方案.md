### **UniApp + Vue3 需求技术方案模板 **

#### **1. 需求背景与目标 (Why & What)**
*   **目的：** 明确需求来源和前端要做什么。
*   **描述：**
    *   **业务诉求：** (1句话) 简述这个需求要解决的用户问题或业务目标。(e.g., 用户希望在商品详情页快速查看附近的线下门店库存)
    *   **前端目标：** **具体**说明前端需要实现或修改的**功能点/交互/界面**。(e.g., 在商品详情页底部新增“附近门店”Tab页，展示地图和门店列表，支持查看实时库存)
    *   **关联改动：** 明确依赖的后端接口**变更/新增**。(e.g., 依赖新的 `GET /api/product/{id}/nearby-stores` 接口)
    *   **影响范围：** 涉及哪些**现有页面/组件/状态/路由**。(e.g., 修改 `pages/product/detail.vue`， 新增 `components/NearbyStores.vue`)

#### **2. 详细设计方案 (How - Vue3 & UniApp Focus)**
*   **目的：** 清晰描述前端如何实现，利用 Vue3 和 UniApp 特性。
*   **关键点：**
    *   **UI/UX 设计：**
        *   **设计稿链接/关键截图：** 附上 UI 设计稿地址或关键界面截图。
        *   **新增/修改的页面/组件结构：** 描述新增组件的层级关系或现有组件的修改点。(e.g., 在 `detail.vue` 的 `<template>` 中新增 `<uni-segmented-control>` 和 `<NearbyStores>` 组件)
        *   **交互逻辑：** 描述核心用户交互流程。(e.g., 用户点击“附近门店”Tab -> 组件加载 -> 获取定位权限 -> 调用接口获取数据 -> 渲染地图和列表)
    *   **组件设计 (Vue3 Composition API)：**
        *   **新增组件：** 组件名、主要职责、`props`、`emits`。
        *   **关键逻辑 (Composable/`setup()`)：** 使用 `ref`/`reactive` 管理哪些状态？使用哪些自定义 Composable (e.g., `useLocation`, `useApiFetch`)? 核心方法逻辑 (伪代码或清晰描述)。
        *   **示例代码片段 (关键部分)：**
            ```vue
            // components/NearbyStores.vue
            <script setup>
            import { ref, onMounted } from 'vue';
            import { useFetch } from '@/composables/useFetch'; // 假设的自定义hook
            import { getLocation } from '@/utils/location'; // 封装的定位方法
            
            const props = defineProps({
              productId: { type: String, required: true }
            });
            
            const stores = ref([]);
            const loading = ref(false);
            const error = ref(null);
            
            const { fetchData } = useFetch(); // 封装了uni.request的hook
            
            onMounted(async () => {
              loading.value = true;
              try {
                const location = await getLocation(); // 获取用户位置
                const params = { lat: location.latitude, lng: location.longitude };
                const data = await fetchData(`/api/product/${props.productId}/nearby-stores`, { params });
                stores.value = data.stores;
              } catch (err) {
                error.value = err.message || '获取门店信息失败';
                uni.showToast({ title: error.value, icon: 'none' });
              } finally {
                loading.value = false;
              }
            });
            </script>
            ```
    *   **状态管理 (Pinia - 如果需要)：**
        *   是否需要新增/修改 Pinia Store Module？ (e.g., 如果附近门店数据需要在多个页面共享，考虑新增 `useNearbyStore`)
        *   Store 的 `state`、`getters`、`actions` 定义简述。
    *   **路由/导航：** 是否需要新增路由？修改现有路由参数？导航跳转逻辑？(e.g., 点击列表项跳转到门店详情页 `uni.navigateTo({ url: '/pages/store/detail?id=' + store.id })`)
    *   **API 调用：**
        *   调用的接口 URL、Method、Request Params/Body 结构 (参考后端文档)。
        *   使用的请求库/封装 (e.g., 直接 `uni.request`, 封装的自定义 `useFetch`, 或 `uni.$u.http`)。
        *   **请求/响应拦截器处理：** 是否需要针对此接口做特殊处理 (e.g., 统一错误处理已在拦截器，无需额外写)。
        *   **Loading & Error 处理：** UI 如何展示加载中和错误状态？(e.g., 使用 `loading.value` 控制骨架屏/加载动画，`error.value` 显示错误提示)
    *   **数据映射/转换：** 如何将接口返回的数据结构转换为前端组件需要的格式？(e.g., 转换 `store.distance` 为 `X.Xkm`)
    *   **条件编译 (重要！)：** 功能在不同平台 (小程序/H5/App) 是否有差异？如何处理？
        ```javascript
        // 示例：只在App和H5使用高德/腾讯地图，小程序用原生Map组件
        const showMap = ref(false);
        onMounted(() => {
          // #ifdef APP-PLUS || H5
          initThirdPartyMap(); // 初始化高德/腾讯地图SDK
          showMap.value = true;
          // #endif
          // #ifdef MP-WEIXIN
          showMap.value = true; // 小程序直接使用 <map> 组件
          // #endif
        });
        ```
    *   **关键配置：** 新增或需要修改的 `config.js` 项或环境变量 (e.g., 地图 SDK Key)。

#### **3. 关联影响与风险评估 (Impact & Risk - Mobile Focus)**
*   **目的：** 评估改动对现有功能、性能、多端兼容性的影响。
*   **关键点：**
    *   **兼容性：**
        *   **API/组件兼容性：** 修改的组件 `props`/`emits` 是否破坏父组件？修改的 Pinia Store 接口是否影响其他使用者？
        *   **多端兼容性：** **重点！** 新增功能在所有目标平台 (iOS, Android, 微信小程序, H5) 是否表现一致？条件编译是否覆盖所有场景？是否有平台 API 差异 (e.g., 定位、地图)？如何处理？
    *   **性能影响：**
        *   **页面加载性能：** 新增组件/地图 SDK 是否显著增加页面加载时间？(e.g., 地图 SDK 较大，考虑懒加载或按需引入)
        *   **运行时性能：** 列表渲染大量门店项是否卡顿？(e.g., 使用 `uni.$u.throttle` 优化滚动事件，或考虑虚拟列表 `uni-list` 组件)
        *   **内存占用：** 地图组件、大量数据是否会增加内存占用？
    *   **依赖方影响：**
        *   **后端依赖：** 新接口是否 Ready？Mock 数据是否准备好？
        *   **其他前端模块：** 是否影响其他页面或组件？需要谁配合？
    *   **权限与安全：**
        *   **用户权限：** 是否需要新权限 (e.g., `scope.userLocation`)？如何引导用户授权？拒绝授权如何处理？
        *   **数据安全：** 敏感信息 (位置、门店地址) 在前端展示是否有风险？
    *   **回滚方案：** 如果上线失败，如何回滚？(e.g., 通过发布平台回退到旧版本前端包，或通过开关/配置隐藏新功能)
    *   **关键风险 & 应对：**
        *   **定位失败/不准：** 降级方案？(e.g., 展示默认城市的门店，提示用户手动选择位置)
        *   **地图 SDK 加载失败/兼容问题：** 降级为纯列表展示？开关控制？
        *   **接口响应慢：** 超时设置？优雅降级？(e.g., 展示骨架屏后超时则提示)
        *   **目标平台审核风险：** (e.g., 小程序新权限需更新隐私协议)

#### **4. 测试要点 (Testing Focus - UniApp Multi-Platform)**
*   **目的：** 明确测试范围和重点，尤其是多端。
*   **关键点：**
    *   **功能测试：**
        *   核心流程走通 (e.g., 进入详情页 -> 切换Tab -> 成功加载并展示附近门店)。
        *   边界/异常：定位权限拒绝、定位失败、接口返回空列表、接口返回错误、网络异常、快速切换Tab。
        *   **回归测试：** 确保商品详情页**原有功能** (加入购物车、立即购买等) 正常。
    *   **多端兼容性测试：** **核心！** 在**所有目标平台** (iOS App, Android App, 微信小程序, H5) 验证：
        *   功能是否可用且符合预期。
        *   UI 布局和样式是否适配。
        *   平台特有行为是否正常 (e.g., 小程序右上角胶囊按钮不影响布局，App 的导航栏)。
    *   **性能测试：**
        *   页面加载时间 (首屏，尤其是包含地图时)。
        *   切换 Tab 或滚动列表的流畅度 (FPS)。
        *   内存占用变化 (DevTools)。
    *   **权限测试：**
        *   首次请求定位权限的弹窗提示。
        *   用户允许/拒绝/关闭弹窗后的逻辑处理。
        *   用户手动去设置页开启/关闭权限后的状态同步。
    *   **弱网/离线测试：** 弱网下加载状态、超时提示、离线时友好提示。

#### **5. 上线计划 (Rollout Plan - App & H5 & Mini Program)**
*   **目的：** 安全可控地发布，考虑多端发布特性。
*   **关键点：**
    *   **依赖发布顺序：**
        *   后端接口先上线 (或提供稳定的 Mock)。
        *   前端发布。
    *   **发布策略 (考虑多端)：**
        *   **H5：** 通常直接部署，通过 Nginx 或 CDN 控制灰度/全量。**监控：** 前端错误监控 (Sentry/Bugsnag)、性能监控、接口错误率。
        *   **小程序：**
            *   提交体验版供测试。
            *   **正式发布：** 提交审核 -> 审核通过 -> 选择“直接发布”或“分阶段发布”(灰度)。**监控：** 小程序后台错误日志、性能分析、自定义分析。
        *   **App：**
            *   **热更新 (wgt)：** 如果符合条件，优先走 wgt 热更新。测试充分后推全量。
            *   **整包更新：** 如果需要 native 模块 (如新地图 SDK)，需发整包。走应用商店审核流程 (iOS App Store, 国内安卓商店)。考虑 **强制更新/可选更新** 策略。
            *   **监控：** UniApp 自带的统计、第三方 APM 工具、应用商店崩溃报告。
    *   **开关/配置：** 是否设计**功能开关**？(e.g., 在配置中心控制是否显示“附近门店”Tab，方便紧急下线)
    *   **回滚方案：**
        *   **H5：** 快速回滚到旧版本静态资源。
        *   **小程序：** 紧急回滚通常需重新提交审核旧版本代码包（不推荐，强调测试和灰度的重要性）。**热修复** (如果支持且问题匹配)。
        *   **App (wgt)：** 发布一个回滚的 wgt 包。**App (整包)：** 回滚困难，需发新包上商店审核。**强调测试和灰度！**
    *   **监控告警：** 上线后密切监控：错误日志、页面加载性能、关键接口成功率、特定平台崩溃率。设置告警阈值。

**资深前端/UniApp 开发者建议（每个技术方案都要考虑到）：**

*   **组件化与复用：** 优先考虑将新功能拆分为可复用组件，并使用 Composition API 抽离可复用的逻辑到 Composables。
*   **性能敏感：** 时刻考虑移动端性能限制。善用懒加载、虚拟列表、节流防抖、避免不必要的响应式开销。
*   **拥抱条件编译：** 这是 UniApp 的核心能力。清晰规划不同平台的代码路径，并在设计和测试阶段就充分考虑。
*   **权限体验：** 设计流畅的权限引导、申请、拒绝处理流程，这是移动端良好体验的基础。
*   **错误边界与降级：** 对网络请求、异步操作、第三方库 (如地图 SDK) 做好错误捕获和优雅降级，避免页面白屏或卡死。
*   **监控与可观测性：** 集成前端监控，及时捕获线上问题和性能瓶颈。多端都要覆盖。
*   **与后端紧密协作：** 明确接口契约，积极参与接口设计评审，使用 Swagger 或 Mock 数据提前开发。
*   **利用 UniApp 插件市场：** 评估是否有成熟插件 (如封装好的地图组件、UI 库) 可加速开发，但需评估插件质量、兼容性和维护性。
*   **文档随手记：** 在组件/Composable 头部写好注释，说明用途、Props、Emits、注意事项。新增的复杂逻辑或平台差异处理也建议写注释。
