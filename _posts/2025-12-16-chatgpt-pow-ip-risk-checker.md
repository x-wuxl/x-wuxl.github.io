---
layout: post
title: "ChatGPT IP 风险检测：使用油猴脚本检测 PoW 难度与 IP 质量"
date: 2025-12-16 14:10:00 +0800
categories: [AI, 实用工具]
tags: [ChatGPT, PoW, IP 检测, 油猴脚本, Tampermonkey, 降智检测]
description: "通过 Tampermonkey 油猴脚本检测 ChatGPT 的 PoW 难度值，辅助判断 IP 是否被判定为高风险或遭到服务降级（降智）。"
keywords: "ChatGPT, PoW difficulty, IP risk, Tampermonkey script, IP 降级, 降智检测"
---

本文介绍如何通过简单的油猴脚本检测你的 IP 是否被 ChatGPT 判定为高风险。在一定程度上，这可以帮助你判断 IP 是否遭到服务降级（降智），或者为何模型不进行思考直接回答。

## 原理说明

ChatGPT 会根据用户 IP 的风险等级下发不同难度的 PoW (Proof of Work) 计算任务。如果 PoW 难度值很低，通常代表你的 IP 被判定为高风险或受到限制。

## 使用教程

### 1. 安装 Tampermonkey

首先需要安装 Tampermonkey（油猴）插件：
*   [https://www.tampermonkey.net/](https://www.tampermonkey.net/)

### 2. 添加脚本

创建一个新脚本，将下方的完整代码复制进去并保存。

### 3. 查看结果

脚本安装完成后，打开 [ChatGPT](https://chatgpt.com/)。
你会看到屏幕右侧出现一个**绿色圆形图标**。将鼠标悬停在图标上，会显示详细信息：

*   **PoW 难度**：显示的数值代表十六进制长度，数值越小风险越高。
*   **IP 质量**：根据难度自动评级（高风险、中等、良好、优秀）。
*   **用户类型**：显示当前账号被识别的类型。

## 完整代码

```
// ==UserScript==
// @name         ChatGPT PoW检测
// @namespace    https://github.com/KoriIku/chatgpt-degrade-checker
// @homepage     https://github.com/KoriIku/chatgpt-degrade-checker
// @author       cangwei
// @version      2.1
// @description  检测 ChatGPT 数据库中的风险等级
// @match        *://chatgpt.com/*
// @grant        none
// @run-at       document-start
// @license      AGPLv3
// ==/UserScript==

(function () {
    'use strict';

    // 存储数据
    let powData = null;

    // 立即拦截 fetch
    const originalFetch = window.fetch;
    console.log('[ChatGPT降级检测] 脚本已加载,开始监听请求');

    window.fetch = async function (resource, options) {
        const url = typeof resource === 'string' ? resource : resource.url;
        // console.log('[ChatGPT降级检测] fetch请求:', url); // 减少日志输出

        const response = await originalFetch(resource, options);

        if (url.includes('/backend-api/sentinel/chat-requirements/prepare')) {
            console.log('[ChatGPT降级检测] 匹配到目标API:', url);
            const clonedResponse = response.clone();
            clonedResponse.json().then(data => {
                console.log('[ChatGPT降级检测] 响应数据:', data);
                const difficulty = data.proofofwork ? data.proofofwork.difficulty : 'N/A';
                const persona = data.persona || 'N/A';
                console.log('[ChatGPT降级检测] PoW难度:', difficulty);
                console.log('[ChatGPT降级检测] 用户类型:', persona);

                // 保存数据
                powData = { difficulty, persona };

                // 更新UI(如果已创建)
                updateUI(difficulty, persona);
            }).catch(e => console.error('[ChatGPT降级检测] 解析响应时出错:', e));
        }
        return response;
    };

    // 更新UI函数
    function updateUI(difficulty, persona) {
        const difficultyEl = document.getElementById('difficulty');
        if (!difficultyEl) return; // UI未创建

        // 计算并显示长度而不是十六进制值
        if (difficulty === 'N/A') {
            difficultyEl.innerText = 'N/A';
        } else {
            const cleanDifficulty = difficulty.replace('0x', '').replace(/^0+/, '');
            const hexLength = cleanDifficulty.length;
            difficultyEl.innerText = hexLength;
        }

        const personaContainer = document.getElementById('persona-container');
        // 免费用户也展示
        if (persona && persona !== 'N/A') {
            personaContainer.style.display = 'block';
            document.getElementById('persona').innerText = persona;
        } else {
            personaContainer.style.display = 'none';
        }

        updateDifficultyIndicator(difficulty);
    }

    // 等待DOM加载后创建UI
    function initUI() {
        // 创建显示框
        const displayBox = document.createElement('div');
        displayBox.style.position = 'fixed';
        displayBox.style.top = '20%';
        displayBox.style.right = '15px';
        displayBox.style.transform = 'translateY(-50%)';
        displayBox.style.width = '220px';
        displayBox.style.padding = '10px';
        displayBox.style.backgroundColor = 'rgba(0, 0, 0, 0.7)';
        displayBox.style.color = '#fff';
        displayBox.style.fontSize = '14px';
        displayBox.style.borderRadius = '8px';
        displayBox.style.boxShadow = '0 4px 8px rgba(0, 0, 0, 0.3)';
        displayBox.style.zIndex = '10000';
        displayBox.style.transition = 'all 0.3s ease';
        displayBox.style.display = 'none';

        displayBox.innerHTML = `
            <div style="margin-bottom: 10px;">
                <strong>PoW 信息</strong>
            </div>
            <div id="content">
                PoW难度: <span id="difficulty">N/A</span><span id="difficulty-level" style="margin-left: 3px"></span>
                <span id="difficulty-tooltip" style="
                    cursor: pointer;
                    color: #fff;
                    font-size: 12px;
                    display: inline-block;
                    width: 14px;
                    height: 14px;
                    line-height: 14px;
                    text-align: center;
                    border-radius: 50%;
                    border: 1px solid #fff;
                    margin-left: 3px;
                ">?</span><br>
                IP质量: <span id="ip-quality">N/A</span><br>
                <span id="persona-container" style="display: none">用户类型: <span id="persona">N/A</span></span>
            </div>
            <div style="
                margin-top: 12px;
                padding-top: 8px;
                border-top: 0.5px solid rgba(255, 255, 255, 0.15);
                font-size: 10px;
                color: rgba(255, 255, 255, 0.5);
                text-align: center;
                letter-spacing: 0.3px;
                cursor: pointer;
            " onclick="window.open('https://github.com/KoriIku/chatgpt-degrade-checker', '_blank')">
                ChatGPT Degrade Checker
            </div>`;
        document.body.appendChild(displayBox);

        // 创建收缩状态的指示器
        const collapsedIndicator = document.createElement('div');
        collapsedIndicator.style.position = 'fixed';
        collapsedIndicator.style.top = '8%';
        collapsedIndicator.style.right = '10px';
        collapsedIndicator.style.transform = 'translateY(-50%)';
        collapsedIndicator.style.width = '32px';
        collapsedIndicator.style.height = '32px';
        collapsedIndicator.style.backgroundColor = 'transparent';
        collapsedIndicator.style.borderRadius = '50%';
        collapsedIndicator.style.cursor = 'pointer';
        collapsedIndicator.style.zIndex = '10000';
        collapsedIndicator.style.padding = '4px';
        collapsedIndicator.style.display = 'flex';
        collapsedIndicator.style.alignItems = 'center';
        collapsedIndicator.style.justifyContent = 'center';
        collapsedIndicator.style.transition = 'all 0.3s ease';

        // 使用SVG作为指示器
        collapsedIndicator.innerHTML = `
            <svg id="status-icon" width="32" height="32" viewBox="0 0 64 64" style="transition: all 0.3s ease;">
                <defs>
                    <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#3498db;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#2ecc71;stop-opacity:1" />
                    </linearGradient>
                    <filter id="glow">
                        <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
                        <feMerge>
                            <feMergeNode in="coloredBlur"/>
                            <feMergeNode in="SourceGraphic"/>
                        </feMerge>
                    </filter>
                </defs>
                <g id="icon-group" filter="url(#glow)">
                    <circle cx="32" cy="32" r="28" fill="url(#gradient)" stroke="#fff" stroke-width="2"/>
                    <circle cx="32" cy="32" r="20" fill="none" stroke="#fff" stroke-width="2" stroke-dasharray="100">
                        <animateTransform
                            attributeName="transform"
                            attributeType="XML"
                            type="rotate"
                            from="0 32 32"
                            to="360 32 32"
                            dur="8s"
                            repeatCount="indefinite"/>
                    </circle>
                    <circle cx="32" cy="32" r="12" fill="none" stroke="#fff" stroke-width="2">
                        <animate
                            attributeName="r"
                            values="12;14;12"
                            dur="2s"
                            repeatCount="indefinite"/>
                    </circle>
                    <circle id="center-dot" cx="32" cy="32" r="4" fill="#fff">
                        <animate
                            attributeName="r"
                            values="4;6;4"
                            dur="2s"
                            repeatCount="indefinite"/>
                    </circle>
                </g>
            </svg>`;
        document.body.appendChild(collapsedIndicator);

        // 鼠标悬停事件
        collapsedIndicator.addEventListener('mouseenter', function () {
            displayBox.style.display = 'block';
            collapsedIndicator.style.opacity = '0';
        });

        displayBox.addEventListener('mouseleave', function () {
            displayBox.style.display = 'none';
            collapsedIndicator.style.opacity = '1';
        });

        // 创建提示框
        const tooltip = document.createElement('div');
        tooltip.id = 'tooltip';
        tooltip.innerText = '这个值越小，代表PoW难度越高，ChatGPT认为你的IP风险越高。';
        tooltip.style.position = 'fixed';
        tooltip.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
        tooltip.style.color = '#fff';
        tooltip.style.padding = '8px 12px';
        tooltip.style.borderRadius = '5px';
        tooltip.style.fontSize = '12px';
        tooltip.style.visibility = 'hidden';
        tooltip.style.zIndex = '10001';
        tooltip.style.width = '240px';
        tooltip.style.lineHeight = '1.4';
        tooltip.style.pointerEvents = 'none';
        document.body.appendChild(tooltip);

        // 显示提示
        document.getElementById('difficulty-tooltip').addEventListener('mouseenter', function (event) {
            tooltip.style.visibility = 'visible';

            const tooltipWidth = 240;
            const mouseX = event.clientX;
            const mouseY = event.clientY;

            let leftPosition = mouseX - tooltipWidth - 10;
            if (leftPosition < 10) {
                leftPosition = mouseX + 20;
            }

            let topPosition = mouseY - 40;

            tooltip.style.left = `${leftPosition}px`;
            tooltip.style.top = `${topPosition}px`;
        });

        // 隐藏提示
        document.getElementById('difficulty-tooltip').addEventListener('mouseleave', function () {
            tooltip.style.visibility = 'hidden';
        });

        // 如果已有数据,立即更新
        if (powData) {
            updateUI(powData.difficulty, powData.persona);
        }
    }

    // 更新difficulty指示器
    function updateDifficultyIndicator(difficulty) {
        const difficultyLevel = document.getElementById('difficulty-level');
        const ipQuality = document.getElementById('ip-quality');

        if (difficulty === 'N/A') {
            setIconColors('#888', '#666');
            difficultyLevel.innerText = '';
            ipQuality.innerHTML = 'N/A';
            return;
        }

        const cleanDifficulty = difficulty.replace('0x', '').replace(/^0+/, '');
        const hexLength = cleanDifficulty.length;

        let color, secondaryColor, textColor, level, qualityText;

        if (hexLength <= 2) {
            color = '#F44336';
            secondaryColor = '#d32f2f';
            textColor = '#ff6b6b';
            level = '(困难)';
            qualityText = '高风险';
        } else if (hexLength === 3) {
            color = '#FFC107';
            secondaryColor = '#ffa000';
            textColor = '#ffd700';
            level = '(中等)';
            qualityText = '中等';
        } else if (hexLength === 4) {
            color = '#8BC34A';
            secondaryColor = '#689f38';
            textColor = '#9acd32';
            level = '(简单)';
            qualityText = '良好';
        } else {
            color = '#4CAF50';
            secondaryColor = '#388e3c';
            textColor = '#98fb98';
            level = '(极易)';
            qualityText = '优秀';
        }

        setIconColors(color, secondaryColor);
        difficultyLevel.innerHTML = `<span style="color: ${textColor}">${level}</span>`;
        ipQuality.innerHTML = `<span style="color: ${textColor}">${qualityText}</span>`;
    }

    function setIconColors(primaryColor, secondaryColor) {
        const gradient = document.querySelector('#gradient');
        if(gradient) {
            gradient.innerHTML = `
                <stop offset="0%" style="stop-color:${primaryColor};stop-opacity:1" />
                <stop offset="100%" style="stop-color:${secondaryColor};stop-opacity:1" />
            `;
        }
    }

    // DOM加载完成后初始化UI
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', initUI);
    } else {
        initUI();
    }
})();
```
