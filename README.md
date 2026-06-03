##  这是一个专为 `超星学习通` 阅读任务准备的自动阅读项目，

你可以直接复制下面的js至 `TemperMonkey(win)`/`UserScripts(mac)` 进行使用。



```js  js adapt for Safari
// ==UserScript==
// @name         超星学习通自动阅读脚本 (Safari 兼容优化版)
// @namespace    https://mooc1.chaoxing.com/
// @version      1.5
// @description  兼容Safari跨域限制，自动滚动并在触底后精准匹配翻页按钮
// @author       AI Bot
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

