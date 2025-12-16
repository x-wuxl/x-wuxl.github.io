---
layout: post
title: "Grok/X.ai 自动化注册机：全自动账号注册与 Token 提取脚本"
date: 2025-12-16 20:10:00 +0800
categories: [AI, 自动化工具]
tags: [Grok, X.ai, 自动化注册, Tampermonkey, 油猴脚本, 账号批量注册]
description: "分享一个 Grok/X.ai 全自动注册机油猴脚本，支持自动生成随机信息、对接临时邮箱验证码、提取 Token 以及自动清理 Cookie 实现循环注册。"
keywords: "Grok 注册机, X.ai 自动化, Grok Token 提取, Tampermonkey 脚本, 批量注册 X.ai"
---

本文分享一个用于自动化注册 Grok (X.ai) 账号的 Tampermonkey 油猴脚本。该脚本集成了一系列自动化流程，包括随机生成用户信息、自动获取并将验证码填入、提取 API Token 以及自动清理 Cookie 以实现循环操作。

## 功能特性

*   **全自动流程**：从 Grok 首页跳转 -> 注册 -> 邮箱验证 -> 填写信息 -> 提交。
*   **随机数据生成**：内置随机姓名和强密码生成器。
*   **临时邮箱对接**：由脚本自动申请临时邮箱并轮询获取验证码。
*   **Token 提取**：注册成功后自动提取 Token 并上传（需配置服务器接口）。
*   **循环模式**：任务完成后自动清理 Cookie 并重启脚本，适合批量操作。

## 使用说明

1.  **安装油猴插件**：确保浏览器已安装 [Tampermonkey](https://www.tampermonkey.net/)。
2.  **配置脚本**：
    *   将下方代码复制到新脚本中。
    *   **重要**：找到代码中的 `extractAndUploadToken` 函数，将 `http://xxx/api/tokens/add` 修改为你自己的 Token 接收接口地址，并填写正确的 `Bearer Token` 凭证。
3.  **运行**：
    *   打开无痕窗口（推荐，以免影响主账号）。
    *   访问 [https://grok.com/](https://grok.com/)。
    *   脚本通常会自动启动；如未启动，可通过油猴菜单点击 "▶️ 启动/继续"。
4.  **注意事项**：
    *   如果遇到 Cloudflare (CF) 验证，脚本可能无法自动通过，需手动点击。
    *   IP 质量对注册成功率有很大影响。

> **⚠️ 免责声明**：本脚本仅供技术研究与学习交流使用。请勿用于非法用途或违反目标平台的服务条款（ToS）。由此产生的任何账号封禁或法律风险由使用者自行承担。

## 完整脚本代码

```javascript
// ==UserScript==
// @name         Grok/X.ai 自动化注册机 (集成自动清理与循环版)
// @namespace    http://tampermonkey.net/
// @version      5.0
// @description  全自动流程：注册 -> 提取Token -> 清理Cookie -> 循环重启
// @author       Bytebender
// @match        *://*/*
// @match        https://x.ai/*
// @match        https://www.x.ai/*
// @match        https://accounts.x.ai/*
// @match        https://grok.com/*
// @match        https://www.grok.com/*
// @grant        GM_setValue
// @grant        GM_getValue
// @grant        GM_registerMenuCommand
// @grant        GM_notification
// @grant        GM_xmlhttpRequest
// @grant        GM_setClipboard
// @grant        GM_cookie
// @connect      mail.chatgpt.org.uk
// @connect      100.64.0.101
// @connect      api.x.ai
// @connect      x.ai
// @connect      grok.com
// @connect      www.grok.com
// @connect      accounts.x.ai
// ==/UserScript==

(function() {
    'use strict';

    // ========================================================
    // 1. 随机数据生成工具
    // ========================================================

    // 生成随机姓名 (首字母大写)
    function getRandomName() {
        const chars = 'abcdefghijklmnopqrstuvwxyz';
        const len = Math.floor(Math.random() * 5) + 4; // 长度 4-8
        let result = '';
        for (let i = 0; i < len; i++) {
            result += chars.charAt(Math.floor(Math.random() * chars.length));
        }
        return result.charAt(0).toUpperCase() + result.slice(1);
    }

    // 生成强密码 (12位，包含大小写+数字+特殊符号)
    function getRandomPassword() {
        const lower = "abcdefghijklmnopqrstuvwxyz";
        const upper = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        const nums = "0123456789";
        const symbols = "!@#$%^&*";

        // 1. 确保每种字符至少有一个
        let pass = "";
        pass += lower[Math.floor(Math.random() * lower.length)];
        pass += upper[Math.floor(Math.random() * upper.length)];
        pass += nums[Math.floor(Math.random() * nums.length)];
        pass += symbols[Math.floor(Math.random() * symbols.length)];

        // 2. 补足剩余长度
        const allChars = lower + upper + nums + symbols;
        for (let i = 0; i < 8; i++) {
            pass += allChars[Math.floor(Math.random() * allChars.length)];
        }

        // 3. 打乱顺序 (洗牌)
        return pass.split('').sort(() => 0.5 - Math.random()).join('');
    }

    // ========================================================
    // 2. 任务编排配置
    // ========================================================
    const actions = [
        // --- 阶段一：Grok 首页跳转 ---
        {
            "step_name": "1. 点击 Grok 首页入口",
            "type": "click",
            "selector": "html > body > div:nth-of-type(2) > div > div > div > main > div > div > div:nth-of-type(3) > div:nth-of-type(2) > a:nth-of-type(2)",
            "url_keyword": "grok.com"
        },
        // --- 阶段二：进入注册页 (X.ai) ---
        {
            "step_name": "2. 点击 X.ai 注册/登录按钮",
            "type": "click",
            "selector": "html > body > div:nth-of-type(2) > div > div > div:nth-of-type(2) > div > div:nth-of-type(2) > button",
            "url_keyword": "accounts.x.ai"
        },
        // --- 阶段三：自动化邮箱 ---
        {
            "step_name": "3. 自动申请并填写临时邮箱",
            "type": "get_email",
            "selector": "html > body > div:nth-of-type(2) > div > div > div:nth-of-type(2) > div > form > div > div > input",
        },
        {
            "step_name": "4. 点击下一步 (提交邮箱)",
            "type": "click",
            "selector": "html > body > div:nth-of-type(2) > div > div > div:nth-of-type(2) > div > form > div:nth-of-type(2) > button"
        },
        // --- 阶段四：验证码 ---
        {
            "step_name": "5. 等待邮件验证码并自动填写",
            "type": "fill_code",
            "selector": "input[name='code']"
        },
        // --- 阶段五：填写个人信息 (全随机) ---
        {
            "step_name": "6. 填写名 (First Name)",
            "type": "input",
            "selector": "input[name='givenName']",
            "value": "__RANDOM__"
        },
        {
            "step_name": "7. 填写姓 (Last Name)",
            "type": "input",
            "selector": "input[name='familyName']",
            "value": "__RANDOM__"
        },
        {
            "step_name": "8. 填写密码 (强密码)",
            "type": "input",
            "selector": "input[name='password']",
            "value": "__RANDOM_PASS__"
        },
        // --- 阶段六：提交 ---
        {
            "step_name": "9. 点击最终提交",
            "type": "click",
            "selector": "button[type='submit']"
        },
        // --- 阶段七：提取 Token ---
        {
            "step_name": "10. 检查跳转并上传 Token",
            "type": "wait_url_and_upload",
            "target_url": "grok.com"
        },
        // --- 阶段八：清理环境并循环 ---
        {
            "step_name": "11. 清理 Cookie 并重启循环",
            "type": "clean_and_restart",
            "clean_targets": [
                "https://x.ai/",
                "https://www.x.ai/",
                "https://accounts.x.ai/",
                "https://grok.com/",
                "https://www.grok.com/"
            ]
        }
    ];

    // ========================================================
    // 3. 核心功能类 (邮箱/网络/Cookie)
    // ========================================================

    // 3.1 网络请求封装
    function gmFetch(url, options) {
        return new Promise((resolve, reject) => {
            GM_xmlhttpRequest({
                url: url,
                method: options.method || 'GET',
                headers: options.headers || {},
                data: options.body || null,
                onload: (response) => {
                    if (response.status >= 200 && response.status < 300) {
                        try { resolve(JSON.parse(response.responseText)); }
                        catch (e) { resolve(response.responseText); }
                    } else { reject(new Error(`HTTP Error: ${response.status}`)); }
                },
                onerror: () => reject(new Error('Network Error')),
                ontimeout: () => reject(new Error('Timeout'))
            });
        });
    }

    // 3.2 临时邮箱客户端
    class TempMailClient {
        constructor() {
            this.baseUrl = "https://mail.chatgpt.org.uk/api";
            this.headers = {
                "User-Agent": "Mozilla/5.0",
                "Origin": "https://mail.chatgpt.org.uk",
                "Referer": "https://mail.chatgpt.org.uk/"
            };
        }
        async getEmail() {
            const result = await gmFetch(`${this.baseUrl}/generate-email`, {
                method: "GET",
                headers: { ...this.headers, "content-type": "application/json" }
            });
            if (result && result.success && result.data?.email) return result.data.email;
            throw new Error("邮箱API返回异常");
        }
        async fetchMessages(email) {
            const url = `${this.baseUrl}/emails?email=${encodeURIComponent(email)}`;
            const result = await gmFetch(url, {
                method: "GET",
                headers: { ...this.headers, "cache-control": "no-cache" }
            });
            return (result.success && result.data?.emails) ? result.data.emails : [];
        }
        async waitForCode(email, timeoutSec = 120) {
            console.log(`[Mail] 开始监听 ${email} ...`);
            const startTime = Date.now();
            const codeRegex = /\b[A-Z0-9]{3}-[A-Z0-9]{3}\b|\b\d{6}\b/;
            return new Promise((resolve, reject) => {
                const timer = setInterval(async () => {
                    if (Date.now() - startTime > timeoutSec * 1000) {
                        clearInterval(timer);
                        reject(new Error("等待验证码超时"));
                    }
                    try {
                        const msgs = await this.fetchMessages(email);
                        if (msgs.length > 0) {
                            for (const msg of msgs) {
                                const content = (msg.subject || "") + " " + (msg.html_content || "");
                                const match = content.match(codeRegex);
                                if (match) {
                                    clearInterval(timer);
                                    resolve(match[0]);
                                    return;
                                }
                            }
                        }
                    } catch(e) { console.warn("Polling error:", e); }
                }, 3000);
            });
        }
    }

    // 3.3 Token 上传逻辑
    async function extractAndUploadToken() {
        return new Promise((resolve, reject) => {
            GM_cookie.list({ name: "sso" }, (cookies, error) => {
                if (error || !cookies || cookies.length === 0) {
                    return reject(new Error("SSO Cookie missing"));
                }
                const ssoToken = cookies[0].value;
                console.log("获取到 Token:", ssoToken.substring(0, 10) + "...");

                GM_xmlhttpRequest({
                    url: "http://xxx/api/tokens/add", // 请修改此处
                    method: "POST",
                    headers: {
                        "content-type": "application/json",
                        "authorization": "Bearer xxxxx" // 请修改此处
                    },
                    data: JSON.stringify({ tokens: [ssoToken], token_type: "sso" }),
                    onload: (response) => {
                        if (response.status >= 200 && response.status < 300) {
                            console.log("Token 上传成功!");
                            resolve();
                        } else {
                            reject(new Error("Upload failed: " + response.responseText));
                        }
                    },
                    onerror: (err) => reject(err)
                });
            });
        });
    }

    // 3.4 Cookie 清理逻辑
    function executeCleanCookies(targetUrls) {
        return new Promise((resolve) => {
            if (!targetUrls || targetUrls.length === 0) return resolve();
            let completed = 0;
            targetUrls.forEach(url => {
                GM_cookie.list({ url: url }, function(cookies, error) {
                    if (cookies && cookies.length > 0) {
                        cookies.forEach(c => {
                            GM_cookie.delete({ name: c.name, url: url }, () => {});
                        });
                    }
                    completed++;
                    if (completed === targetUrls.length) {
                        setTimeout(resolve, 800); // 缓冲
                    }
                });
            });
        });
    }

    // 3.5 Native 输入模拟 (绕过 React/Vue 绑定)
    function setNativeValue(element, value) {
        const valueSetter = Object.getOwnPropertyDescriptor(element, 'value').set;
        const prototype = Object.getPrototypeOf(element);
        const prototypeValueSetter = Object.getOwnPropertyDescriptor(prototype, 'value').set;
        if (valueSetter && valueSetter !== prototypeValueSetter) {
            prototypeValueSetter.call(element, value);
        } else {
            valueSetter.call(element, value);
        }
        element.dispatchEvent(new Event('input', { bubbles: true }));
    }

    // 3.6 元素等待
    const waitForElement = (selector, timeout = 10000) => {
        return new Promise((resolve, reject) => {
            const el = document.querySelector(selector);
            if (el) return resolve(el);
            const observer = new MutationObserver(() => {
                const el = document.querySelector(selector);
                if (el) { observer.disconnect(); resolve(el); }
            });
            observer.observe(document.body, { childList: true, subtree: true });
            setTimeout(() => { observer.disconnect(); reject(new Error('元素超时: ' + selector)); }, timeout);
        });
    };

    // ========================================================
    // 4. 自动化执行引擎
    // ========================================================
    const mailClient = new TempMailClient();
    let isRunning = GM_getValue('script_is_running', false);
    let currentIndex = GM_getValue('script_step_index', 0);

    console.log(`🚀 [注册机状态] Running: ${isRunning} | Step: ${currentIndex}`);

    GM_registerMenuCommand(`▶️ 启动/继续`, () => {
        GM_setValue('script_is_running', true);
        isRunning = true;
        runCurrentStep();
    });

    GM_registerMenuCommand("🔄 强制重置", () => {
        GM_setValue('script_step_index', 0);
        GM_setValue('script_is_running', false);
        GM_setValue('current_temp_email', '');
        location.reload();
    });

    async function runCurrentStep() {
        if (!GM_getValue('script_is_running', false)) return;

        // 异常保护：索引越界重置
        if (currentIndex >= actions.length) {
            GM_setValue('script_step_index', 0);
            return location.reload();
        }

        const action = actions[currentIndex];
        console.log(`[Step ${currentIndex + 1}] ${action.step_name} (${action.type})`);

        // URL 检查 (如果不在目标域名，等待跳转)
        if (action.url_keyword && !location.href.includes(action.url_keyword)) {
            console.log(`等待跳转到 ${action.url_keyword}...`);
            return setTimeout(runCurrentStep, 2000);
        }

        try {
            await new Promise(r => setTimeout(r, 3000)); // 基础缓冲
            let el = null;
            if (action.selector) el = await waitForElement(action.selector);

            // --- 动作分发 ---
            if (action.type === 'get_email') {
                const email = await mailClient.getEmail();
                console.log("获取邮箱:", email);
                GM_setValue('current_temp_email', email);
                GM_setClipboard(email);

                el.click(); el.focus();
                setNativeValue(el, email);
                el.dispatchEvent(new Event('change', { bubbles: true }));
                el.blur();
            }
            else if (action.type === 'fill_code') {
                const email = GM_getValue('current_temp_email');
                const rawCode = await mailClient.waitForCode(email);
                const code = rawCode.replace(/-/g, ''); // 清洗连字符
                console.log('填入验证码:', code);

                el.scrollIntoView({block: "center"});
                el.click(); el.focus();
                const nativeSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;
                nativeSetter.call(el, code);
                el.dispatchEvent(new Event('input', { bubbles: true }));
                el.dispatchEvent(new Event('change', { bubbles: true }));
                await new Promise(r => setTimeout(r, 500));
                el.blur();
            }
            else if (action.type === 'input') {
                el.focus();
                let val = action.value;

                // 处理随机变量
                if (val === '__RANDOM__') val = getRandomName();
                if (val === '__RANDOM_PASS__') val = getRandomPassword();

                console.log(`[Input] 填写值: ${val}`);
                setNativeValue(el, val);
                el.blur();
            }
            else if (action.type === 'wait_url_and_upload') {
                if (!location.href.includes(action.target_url)) {
                    return setTimeout(runCurrentStep, 1500); // URL不对，继续等待
                }

                // 尝试多次上传，防止 Cookie 未即时写入
                let retry = 0;
                while (retry < 5) {
                    try {
                        await extractAndUploadToken();
                        GM_notification({ text: 'Token 上传成功！准备清理...', title: '成功' });
                        break;
                    } catch (e) {
                        console.warn("Token提取失败，重试中...", e);
                        await new Promise(r => setTimeout(r, 2000));
                        retry++;
                    }
                }
                // 继续下一步
            }
            else if (action.type === 'clean_and_restart') {
                GM_notification({ text: '清理 Cookie 并重启循环...', title: '系统维护' });
                await executeCleanCookies(action.clean_targets);

                GM_setValue('script_step_index', 0);
                GM_setValue('current_temp_email', '');

                console.log(">>> 循环重置完成，3秒后刷新");
                setTimeout(() => {
                    window.location.href = "https://grok.com/";
                }, 3000);
                return; // 结束本次执行栈
            }
            else if (action.type === 'click') {
                el.click();
            }

            // --- 步进逻辑 ---
            currentIndex++;
            GM_setValue('script_step_index', currentIndex);
            setTimeout(runCurrentStep, 1500);

        } catch (e) {
            console.error("执行出错:", e);
            // 遇到严重错误可以考虑刷新页面重试
            // setTimeout(() => location.reload(), 5000);
        }
    }

    // 启动检测
    function tryStart() {
        if (isRunning) {
            setTimeout(runCurrentStep, 1500);
        }
    }

    if (document.readyState === 'complete' || document.readyState === 'interactive') {
        tryStart();
    } else {
        window.addEventListener('load', tryStart);
    }

})();
```
