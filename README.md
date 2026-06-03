###  这是一个专为 `超星学习通` 阅读任务准备的自动阅读项目，

### 你可以直接复制下面的js至 `TemperMonkey(win)`/`UserScripts(mac)` 进行使用。

---

# UserScripts(mac)
```js  js adapt for Safari
// ==UserScript==
// @name         超星学习通自动阅读脚本 (Safari 兼容优化版)
// @namespace    https://mooc1.chaoxing.com/
// @version      1.5
// @description  兼容Safari跨域限制，自动滚动并在触底后精准匹配翻页按钮
// @author       Sudark
// @license      MIT
// @match        *://*.chaoxing.com/*
// @match        *://*.edu.cn/*
// @run-at       document-end
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 滚动速度（毫秒）
    const scrollSpeed = 12000;
    // 翻页检查间隔时间（秒） - 120秒
    const pageTime = 120;

    // 仅在实际包含内容的文档中运行，避免在无用的外层框架中白跑
    // 超星的内容通常在包含特定 class 或元素的页面中
    console.log("阅读脚本已注入层级:", window.location.href);

    // 1. 自动滚动逻辑 (直接在当前上下文中执行)
    const scrollTimer = setInterval(() => {
        try {
            // 每次滚动屏幕的 1/5
            window.scrollBy({
                top: window.innerHeight / 5,
                behavior: 'smooth' 
            });
        } catch (e) {
            console.error("滚动出错:", e);
        }
    }, scrollSpeed);

    // 2. 自动翻页逻辑
    const pageTimer = setInterval(() => {
        try {
            const doc = document;
            const win = window;

            const scrollHeight = Math.max(doc.documentElement.scrollHeight, doc.body.scrollHeight);
            const scrollTop = win.scrollY || doc.documentElement.scrollTop || doc.body.scrollTop;
            const clientHeight = win.innerHeight || doc.documentElement.clientHeight;

            // 预留 50px 误差
            const isAtBottom = Math.ceil(scrollTop + clientHeight) >= (scrollHeight - 50);

            if (!isAtBottom) {
                console.log("页面尚未到底部，等待下一次检查...");
                return; 
            }

            console.log("已到达页面底部，开始寻找下一页按钮...");

            const nextSelectors = [
                '.nodeItem.r',      // 根据截图精确定位
                '.nextBtn',         // 备用：老版本
                '.prev_next.next',  // 备用：其他阅读器
                '.jb_btn_next'      // 备用：新版界面
            ];

            let nextPageBtn = null;
            
            for (let selector of nextSelectors) {
                nextPageBtn = doc.querySelector(selector);
                if (nextPageBtn) {
                    // 确保按钮是可见的，而不是隐藏在后台的无效按钮
                    if (nextPageBtn.offsetParent !== null) {
                        break;
                    }
                }
            }

            if (nextPageBtn) {
                console.log("成功找到下一页按钮，执行点击...");
                nextPageBtn.click();
            } else {
                // 检查是否全部完成
                const allDone = doc.querySelector('.allDone');
                if (allDone && allDone.innerText.includes('全部完成')) {
                    console.log("🎉 检测到【全部完成】，已停止自动运行");
                    clearInterval(scrollTimer);
                    clearInterval(pageTimer);
                }
            }
        } catch (e) {
            console.error("执行检查时遇到错误：", e);
        }
    }, pageTime * 1000);

})();
```

# TemperMonkey(win)
```js
// ==UserScript==
// @name         超星学习通自动阅读脚本 (精准定位+到底翻页版)
// @namespace    https://mooc1.chaoxing.com/
// @version      1.4
// @description  自动滚动并在触底后精准匹配翻页按钮
// @author       Sudark
// @license      MIT
// @match        *://*.chaoxing.com/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 滚动速度（毫秒）
    const scrollSpeed = 12000;
    // 翻页检查间隔时间（秒） - 已修改为 120 秒（2分钟）
    const pageTime = 120;

    console.log("阅读脚本已启动，正在监控...");

    // 获取 iframe 内容，处理嵌套网页的问题
    function getIframeContext() {
        const iframe = document.querySelector('iframe');
        if (iframe && iframe.contentWindow && iframe.contentDocument) {
            return { win: iframe.contentWindow, doc: iframe.contentDocument };
        }
        return { win: window, doc: document };
    }

    // 1. 自动滚动逻辑
    const scrollTimer = setInterval(() => {
        try {
            const context = getIframeContext();
            context.win.scrollBy(0, context.win.innerHeight / 5);
        } catch (e) {}
    }, scrollSpeed);

    // 2. 自动翻页逻辑（每 2 分钟检查一次）
    const pageTimer = setInterval(() => {
        try {
            const context = getIframeContext();
            const win = context.win;
            const doc = context.doc;

            // 新增：判断是否到达页面底部 (预留 50px 误差以防页面缩放导致的像素不匹配)
            const scrollHeight = Math.max(doc.documentElement.scrollHeight, doc.body.scrollHeight);
            const scrollTop = win.scrollY || doc.documentElement.scrollTop || doc.body.scrollTop;
            const clientHeight = win.innerHeight || doc.documentElement.clientHeight;

            const isAtBottom = Math.ceil(scrollTop + clientHeight) >= (scrollHeight - 50);

            if (!isAtBottom) {
                console.log("页面尚未到底部，继续阅读等待下一次检查...");
                return; // 如果没到底部，直接退出本次检查，等待下一个 2 分钟
            }

            console.log("已到达页面底部，开始寻找下一页按钮...");

            // 🎯 核心修改：将你截图提取的 '.nodeItem.r' 放在最高优先级
            const nextSelectors = [
                '.nodeItem.r',      // 根据截图精确定位
                '.nextBtn',         // 备用：老版本
                '.prev_next.next',  // 备用：其他阅读器
                '.jb_btn_next'      // 备用：新版界面
            ];

            let nextPageBtn = null;
            // 遍历寻找按钮，同时在主页面和 iframe 内部寻找
            for (let selector of nextSelectors) {
                nextPageBtn = context.doc.querySelector(selector) || document.querySelector(selector);
                if (nextPageBtn) {
                    break; // 找到了就跳出循环
                }
            }

            // 执行点击或判断结束
            if (nextPageBtn) {
                console.log("成功找到下一页按钮，执行点击...");
                nextPageBtn.click();
            } else {
                const allDone = context.doc.querySelector('.allDone') || document.querySelector('.allDone');
                if (allDone && allDone.innerText.includes('全部完成')) {
                    console.log("🎉 检测到【全部完成】，已停止自动运行");
                    clearInterval(scrollTimer);
                    clearInterval(pageTimer);
                }
            }
        } catch (e) {
            console.error("执行时遇到错误：", e);
        }
    }, pageTime * 1000);

})();
```
