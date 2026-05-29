---
layout: post
title: GoPay 循环换绑开通 ChatGPT Plus：一次接码，无限免费订阅实操指南
date: 2026-05-29
last_modified_at: 2026-05-29
categories: [AI工具, 实用教程]
tags: [ChatGPT, ChatGPT Plus, GoPay, 虚拟支付, 换绑教程, 薅羊毛, 免费试用]
keywords: 免费开通 ChatGPT Plus, ChatGPT Plus 订阅, ChatGPT Plus 试用, GoPay 换绑, GoPay 支付 ChatGPT, 印度尼西亚 GoPay, ChatGPT 便宜订阅
description: 本文详细介绍如何通过 GoPay 钱包的换绑机制，仅需一次 WhatsApp 接码，即可实现低成本、循环开通 ChatGPT Plus 订阅的实操方法。
author: wuxl
excerpt: 本文分享了利用印尼 GoPay 钱包的换绑特性，释放主 WhatsApp 账号并实现单号循环注册、低成本订阅 ChatGPT Plus 的具体步骤，内附付款长链提取脚本。
toc: true
toc_sticky: true
---

利用印度尼西亚电子钱包 GoPay 订阅 ChatGPT Plus 是一种常见的低成本支付方式。然而，频繁购买新的接码号注册 GoPay 会增加操作成本。

本文将分享一种**“换绑释放”**的技巧：通过将当前的 GoPay 账号绑定到临时的接码号上，从而释放出您最初的印尼 WhatsApp 账号；随后，您可以使用该 WhatsApp 账号重新注册全新的 GoPay 钱包并再次享受新户订阅优惠。

在开始操作前，请确保准备好以下工具和环境：

1. **印度尼西亚 WhatsApp 账号**：必须使用印尼手机号注册，后续将重复使用它来登录和注册 GoPay。
2. **GoPay 应用程序**：手机端安装好最新版 GoPay App。
3. **Stripe 付款长链提取脚本**：用于在电脑端浏览器中生成 ChatGPT Plus 的 GoPay 专属支付链接。

### 长链提取脚本
在浏览器控制台（F12 -> Console）中运行以下 JavaScript 代码

```javascript
(async () => {
  "use strict";

  const PATH = "/checkout/openai_llc/";

  const PAYLOAD = {
    plan_name: "chatgptplusplan",
    billing_details: {
      country: "ID",
      currency: "IDR",
    },
    cancel_url: "https://chatgpt.com/#pricing",
    promo_campaign: {
      promo_campaign_id: "plus-1-month-free",
      is_coupon_from_query_param: false,
    },
    checkout_ui_mode: "hosted",
  };

  function log(message, data) {
    console.log(`[GoPay Checkout] ${message}`, data || "");
  }

  async function fetchJson(url, options = {}) {
    const response = await fetch(url, options);
    const data = await response.json().catch(() => null);

    if (!response.ok) {
      console.error("[GoPay Checkout] 请求失败：", data);
      throw new Error(`HTTP ${response.status}`);
    }

    return data;
  }

  try {
    if (!location.pathname.startsWith(PATH)) {
      alert(`当前不是 checkout 页面。\n\n请先进入 ${PATH} 开头的页面再执行。`);
      console.warn("[GoPay Checkout] 当前路径：", location.pathname);
      return;
    }

    log("正在获取登录 Token...");

    const session = await fetchJson("/api/auth/session", {
      credentials: "include",
    });

    const token = session?.accessToken;

    if (!token) {
      alert("获取登录 Token 失败，请确认你已经登录 ChatGPT。");
      throw new Error("没有获取到 accessToken");
    }

    log("Token 获取成功，正在生成 Stripe 付款链接...");

    const data = await fetchJson("https://chatgpt.com/backend-api/payments/checkout", {
      method: "POST",
      credentials: "include",
      headers: {
        Authorization: `Bearer ${token}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify(PAYLOAD),
    });

    const checkoutUrl = data?.url || data?.stripe_hosted_url || data?.checkout_url;

    if (!checkoutUrl) {
      console.error("[GoPay Checkout] 原始响应：", data);
      alert("没有在响应里找到付款链接，请看控制台原始响应。");
      return;
    }

    console.log("[GoPay Checkout] Stripe 付款链接：", checkoutUrl);
    console.log("[GoPay Checkout] 原始响应：", data);

    location.href = checkoutUrl;
  } catch (error) {
    console.error("[GoPay Checkout] 执行失败：", error);
    alert(`执行失败：${error.message || error}`);
  }
})();
```

### 第一步：换绑释放当前的 WhatsApp 号码

1. 打开手机上的 GoPay 客户端，进入个人资料页面（**Profile**）。
2. 使用接码平台（例如 HeroSMS），购买一个**印度尼西亚 Gojek 虚拟号码**（仅用于接收一次换绑验证码）。
3. 在 GoPay 的个人设置中选择修改手机号，将当前账号的绑定手机号修改为刚刚购买的虚拟号码。
4. 电子邮箱（Email）可以随意填写一个格式正确的邮箱。
5. 选择**发送验证码至 SMS** 输入验证码完成换绑。
6. 成功解绑后，原来的 WhatsApp 号码即被释放。此时，在 App 中退出（Log out）当前的 GoPay 账号。

### 第二步：使用原 WhatsApp 注册新 GoPay 账号

1. 重新打开 GoPay 注册页面，输入您刚刚释放的**主 WhatsApp 号码**进行注册。
2. 验证码会直接发送到您的主 WhatsApp 上，完成登录。
3. 进入新账号后，在设置（**Settings**）中设置好支付密码（**PIN**）。
4. 等待系统赠送的 `1 Rp`（Rupiah）体验金到账。如果资金未实时到账，可以在 App 内随意点击浏览或体验内置的小游戏以激活账户状态。

### 第三步：提取链接并完成 ChatGPT Plus 订阅

1. 登录您需要开通 ChatGPT Plus 的 OpenAI 账号（需要有 Free Offer 的账号）。
2. 在电脑浏览器中进入 ChatGPT Plus 订阅付款初始化页面（即 URL 以 `/checkout/openai_llc/` 开头的页面）。
3. 按下键盘上的 `F12` 键打开开发者工具，切换到 `Console`（控制台）面板。
4. 复制并粘贴上文提供的**长链提取脚本**，按下回车运行。
5. 脚本运行成功后，页面会自动跳转至 Stripe 收银台。在支付方式中，账单国家选择**印度尼西亚（ID）**，付款方式选择 **GoPay**。
6. 按照页面提示，使用刚刚新注册的手机 GoPay 扫码或跳转完成付款。

### ⚠️ 关键注意事项：
* **防风控间隔时间**：在完成一次 Plus 订阅后，请**不要立即**进行下一次换绑。建议在操作完一个 Plus 订阅后，保持账号静置，**等待 1 小时以上**再进行下一次换绑和注册操作。频繁操作极易触发 GoPay 或 Stripe 的安全风控。