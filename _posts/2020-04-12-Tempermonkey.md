---
layout: post
title: Tempermonkey 工具脚本
categories: Tools Tempermonkey
description: 浏览器实用工具大全
keywords: Tempermonkey Tools
---

## 说明：所有工具都必须安装Tempermonkey插件 [下载地址](https://www.tampermonkey.net)

### 1. 网页视频倍速播放限制解除，跳过视频广告，vip视频免费看

选择工具栏上的 Tempermonkey 图标，点击添加新脚本；

复制以下代码到编辑框中，点击文件-保存

```javascript
// ==UserScript==
// @name        计时器掌控者|视频广告跳过|视频广告加速器
// @name:en      TimerHooker
// @name:zh-CN   计时器掌控者|视频广告跳过|视频广告加速器
// @namespace    https://gitee.com/HGJing/everthing-hook/
// @version      1.0.38
// @description       控制网页计时器速度|加速跳过页面计时广告|视频快进（慢放）|跳过广告|支持几乎所有网页.
// @description:en  it can hook the timer speed to change.
// @description:zh-CN  控制网页计时器速度|加速跳过页面计时广告|跳过广告|支持几乎所有网页.
// @include      *
// @require      https://greasyfork.org/scripts/372672-everything-hook/code/Everything-Hook.js?version=784972
// @author       Cangshi
// @match        http://*/*
// @run-at       document-start
// @grant        none
// ==/UserScript==
/**
 * ---------------------------
 * Time: 2017/11/20 19:28.
 * Author: Cangshi
 * View: http://palerock.cn
 * ---------------------------
 */
window.isDOMLoaded = false;
window.isDOMRendered = false;

document.addEventListener('readystatechange', function () {
    if (document.readyState === "interactive" || document.readyState === "complete") {
        window.isDOMLoaded = true;
    }
});

~function (global) {

    var workerURLs = [];
    var extraElements = [];

    var helper = function (eHookContext, timerContext, util) {
        return {
            applyUI: function () {
                var style = '._th-container ._th-item{margin-bottom:3px;position:relative;width:30px;height:30px;cursor:pointer;opacity:.3;background-color:aquamarine;border-radius:100%;text-align:center;line-height:30px;-webkit-transition:all .35s;-o-transition:all .35s;transition:all .35s;right:30px}._th-container ._th-item._item-x2{margin-left:18px;width:40px;height:40px;line-height:40px}._th-container ._th-item._item-x-2{margin-left:17px;width:38px;height:38px;line-height:38px}._th-container ._th-item._item-x4{width:36px;height:36px;margin-left:16px;line-height:36px}._th-container ._th-item._item-x-4{width:32px;height:32px;line-height:32px;margin-left:14px}._th-container ._th-item._item-reset{width:30px;line-height:30px;height:30px;margin-left:10px}._th-click-hover{position:relative;-webkit-transition:all .5s;-o-transition:all .5s;transition:all .5s;height:50px;width:50px;cursor:pointer;opacity:.3;border-radius:100%;background-color:aquamarine;text-align:center;line-height:50px;right:0}._th-container:hover{left:-10px}._th-container{font-size:12px;-webkit-transition:all .5s;-o-transition:all .5s;transition:all .5s;left:-40px;top:20%;position:fixed;-webkit-box-sizing:border-box;box-sizing:border-box;z-index:100000;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;user-select:none}._th-container ._th-item:hover{opacity:.8;background-color:#5fb492;color:aliceblue}._th-container ._th-item:active{opacity:.9;background-color:#316347;color:aliceblue}._th-container:hover ._th-click-hover{opacity:.8}._th-container:hover ._th-item{opacity:.6;right:0}._th-container ._th-click-hover:hover{opacity:.8;background-color:#5fb492;color:aliceblue}._th_cover-all-show-times{position:fixed;top:0;right:0;width:100%;height:100%;z-index:99999;opacity:1;font-weight:900;font-size:30px;color:#4f4f4f;background-color:rgba(0,0,0,0.1)}._th_cover-all-show-times._th_hidden{z-index:-99999;opacity:0;-webkit-transition:1s all;-o-transition:1s all;transition:1s all}._th_cover-all-show-times ._th_times{width:300px;height:300px;border-radius:50%;background-color:rgba(127,255,212,0.51);text-align:center;line-height:300px;position:absolute;top:50%;right:50%;margin-top:-150px;margin-right:-150px;}';

                // 在页面左边添加一个半圆便于修改
                var html = '<div class="_th-container">\n' +
                    '    <div class="_th-click-hover" onclick="changeTime()">\n' +
                    '        x' + 1 / timerContext._percentage + '\n' +
                    '    </div>\n' +
                    '    <div class="_th-item _item-x2" onclick="changeTime(2,0,true)">&gt;</div>\n' +
                    '    <div class="_th-item _item-x-2" onclick="changeTime(-2,0,true)">&lt;</div>\n' +
                    '    <div class="_th-item _item-x4" onclick="changeTime(0,4)">&gt;&gt;</div>\n' +
                    '    <div class="_th-item _item-x-4" onclick="changeTime(0,-4)">&lt;&lt;</div>\n' +
                    '    <div class="_th-item _item-reset" onclick="changeTime(0,0,false,true)">O</div>\n' +
                    '</div>\n' +
                    '<div class="_th_cover-all-show-times _th_hidden">\n' +
                    '    <div class="_th_times">x' + 1 / timerContext._percentage + '</div>\n' +
                    '</div>' +
                    '';
                var stylenode = document.createElement('style');
                stylenode.setAttribute("type", "text/css");
                if (stylenode.styleSheet) {// IE
                    stylenode.styleSheet.cssText = style;
                } else {// w3c
                    var cssText = document.createTextNode(style);
                    stylenode.appendChild(cssText);
                }
                var node = document.createElement('div');
                node.innerHTML = html;
                if (!global.isDOMLoaded) {
                    document.addEventListener('readystatechange', function () {
                        if ((document.readyState === "interactive" || document.readyState === "complete") && !global.isDOMRendered) {
                            document.head.appendChild(stylenode);
                            document.body.appendChild(node);
                            global.isDOMRendered = true;
                            console.log('Time Hooker Works!');
                        }
                    });
                } else {
                    document.head.appendChild(stylenode);
                    document.body.appendChild(node);
                    global.isDOMRendered = true;
                    console.log('Time Hooker Works!');
                }
            },
            applyGlobalAction: function (timer) {
                // 界面半圆按钮点击的方法
                timer.changeTime = function (anum, cnum, isa, isr) {
                    if (isr) {
                        global.timer.change(1);
                        return;
                    }
                    if (!global.timer) {
                        return;
                    }
                    var result;
                    if (!anum && !cnum) {
                        var t = prompt("输入欲改变计时器变化倍率（当前：" + 1 / timerContext._percentage + "）");
                        if (t == null) {
                            return;
                        }
                        if (isNaN(parseFloat(t))) {
                            alert("请输入正确的数字");
                            timer.changeTime();
                            return;
                        }
                        if (parseFloat(t) <= 0) {
                            alert("倍率不能小于等于0");
                            timer.changeTime();
                            return;
                        }
                        result = 1 / parseFloat(t);
                    } else {
                        if (isa && anum) {
                            if (1 / timerContext._percentage <= 1 && anum < 0) {
                                return;
                            }
                            result = 1 / (1 / timerContext._percentage + anum);
                        } else {
                            if (cnum < 0) {
                                cnum = 1 / -cnum
                            }
                            result = 1 / ((1 / timerContext._percentage) * cnum);
                        }
                    }
                    timer.change(result);
                };
                global.changeTime = timer.changeTime;
            },
            applyHooking: function () {
                // 劫持循环计时器
                eHookContext.hookReplace(window, 'setInterval', function (setInterval) {
                    return function () {
                        // 储存原始时间间隔
                        arguments[2] = arguments[1];
                        // 获取变速时间间隔
                        arguments[1] *= timerContext._percentage;
                        var resultId = setInterval.apply(window, arguments);
                        // 保存每次使用计时器得到的id以及参数等
                        timerContext._intervalIds[resultId] = {
                            args: arguments,
                            nowId: resultId
                        };
                        return resultId;
                    };
                });
                // 劫持循环计时器的清除方法
                eHookContext.hookBefore(window, 'clearInterval', function (method, args) {
                    var id = args[0];
                    if (timerContext._intervalIds[id]) {
                        args[0] = timerContext._intervalIds[id].nowId;
                    }
                    // 清除该记录id
                    delete timerContext._intervalIds[id];
                });
                // 劫持循环计时器的清除方法
                eHookContext.hookBefore(window, 'clearTimeout', function (method, args) {
                    var id = args[0];
                    if (timerContext._intervalIds[id]) {
                        args[0] = timerContext._intervalIds[id].nowId;
                    }
                    // 清除该记录id
                    delete timerContext._intervalIds[id];
                });
                // 劫持单次计时器setTimeout
                eHookContext.hookBefore(window, 'setTimeout', function (method, args) {
                    args[1] *= timerContext._percentage;
                });
                var newFunc = this.getHookedDateConstructor();
                eHookContext.hookClass(window, 'Date', newFunc, '_innerDate', ['now']);
                Date.now = function () {
                    return new Date().getTime();
                };
                eHookContext.hookedToString(timerContext._Date.now, Date.now);
                var objToString = Object.prototype.toString;

                eHookContext.hookAfter(Object.prototype, 'toString', function (m, args, result) {
                    if (this instanceof timerContext._mDate) {
                        return '[object Date]';
                    } else {
                        return result;
                    }
                }, false);

                eHookContext.hookedToString(objToString, Object.prototype.toString);
                eHookContext.hookedToString(timerContext._setInterval, setInterval);
                eHookContext.hookedToString(timerContext._setTimeout, setTimeout);
                eHookContext.hookedToString(timerContext._clearInterval, clearInterval);
                timerContext._mDate = window.Date;
                this.hookShadowRoot();
            },
            getHookedDateConstructor: function () {
                return function () {
                    if (arguments.length === 1) {
                        Object.defineProperty(this, '_innerDate', {
                            configurable: false,
                            enumerable: false,
                            value: new timerContext._Date(arguments[0]),
                            writable: false
                        });
                        return;
                    } else if (arguments.length > 1) {
                        var definedValue;
                        switch (arguments.length) {
                            case 2:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1]
                                );
                                break;
                            case 3:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1],
                                    arguments[2],
                                );
                                break;
                            case 4:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1],
                                    arguments[2],
                                    arguments[3],
                                );
                                break;
                            case 5:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1],
                                    arguments[2],
                                    arguments[3],
                                    arguments[4]
                                );
                                break;
                            case 6:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1],
                                    arguments[2],
                                    arguments[3],
                                    arguments[4],
                                    arguments[5]
                                );
                                break;
                            default:
                            case 7:
                                definedValue = new timerContext._Date(
                                    arguments[0],
                                    arguments[1],
                                    arguments[2],
                                    arguments[3],
                                    arguments[4],
                                    arguments[5],
                                    arguments[6]
                                );
                                break;
                        }

                        Object.defineProperty(this, '_innerDate', {
                            configurable: false,
                            enumerable: false,
                            value: definedValue,
                            writable: false
                        });
                        return;
                    }
                    var now = timerContext._Date.now();
                    var passTime = now - timerContext.__lastDatetime;
                    var hookPassTime = passTime * (1 / timerContext._percentage);
                    // console.log(__this.__lastDatetime + hookPassTime, now,__this.__lastDatetime + hookPassTime - now);
                    Object.defineProperty(this, '_innerDate', {
                        configurable: false,
                        enumerable: false,
                        value: new timerContext._Date(timerContext.__lastMDatetime + hookPassTime),
                        writable: false
                    });
                };
            },
            registerShortcutKeys: function (timer) {
                // 快捷键注册
                addEventListener('keydown', function (e) {
                    switch (e.keyCode) {
                        // [=]
                        case 190:
                        case 187: {
                            if (e.ctrlKey) {
                                // console.log('+2');
                                timer.changeTime(2, 0, true);
                            } else if (e.altKey) {
                                // console.log('x4');
                                timer.changeTime(0, 4);
                            }
                            break;
                        }
                        // [-]
                        case 188:
                        case 189: {
                            if (e.ctrlKey) {
                                // console.log('-2');
                                timer.changeTime(-2, 0, true);
                            } else if (e.altKey) {
                                // console.log('x-4');
                                timer.changeTime(0, -4);
                            }
                            break;
                        }
                        // [0]
                        case 48: {
                            if (e.ctrlKey || e.altKey) {
                                // console.log('reset');
                                timer.changeTime(0, 0, false, true);
                            }
                            break;
                        }
                        default:
                        // console.log(e);
                    }
                });
            },
            /**
             * 当计时器速率被改变时调用的回调方法
             * @param percentage
             * @private
             */
            percentageChangeHandler: function (percentage) {
                // 改变所有的循环计时
                util.ergodicObject(timerContext, timerContext._intervalIds, function (idObj, id) {
                    idObj.args[1] = Math.floor(idObj.args[2] * percentage);
                    // console.log(idObj.args[1]);
                    // 结束原来的计时器
                    this._clearInterval.call(window, idObj.nowId);
                    // 新开一个计时器
                    idObj.nowId = this._setInterval.apply(window, idObj.args);
                });
            },
            hookShadowRoot: function () {
                var origin = Element.prototype.attachShadow;
                eHookContext.hookAfter(Element.prototype, 'attachShadow',
                    function (m, args, result) {
                        extraElements.push(result);
                        return result;
                    }, false);
                eHookContext.hookedToString(origin, Element.prototype.attachShadow);
            }
        }
    };

    var normalUtil = {
        isInIframe: function () {
            return global.parent !== global && global.parent.document.body.tagName !== 'FRAMESET';
        },
        listenParentEvent: function (handler) {
            global.addEventListener('message', function (e) {
                var data = e.data;
                var type = data.type || '';
                if (type === 'changePercentage') {
                    handler(data.percentage || 0);
                }
            })
        },
        sentChangesToIframe: function (percentage) {
            var iframes = document.querySelectorAll('iframe') || [];
            var frames = document.querySelectorAll('frame');
            if (iframes.length) {
                for (var i = 0; i < iframes.length; i++) {
                    iframes[i].contentWindow.postMessage(
                        {type: 'changePercentage', percentage: percentage}, '*');
                }
            }
            if (frames.length) {
                for (var j = 0; j < frames.length; j++) {
                    frames[j].contentWindow.postMessage(
                        {type: 'changePercentage', percentage: percentage}, '*');
                }
            }
        }
    };

    var querySelectorAll = function (ele, selector, includeExtra) {
        var elements = ele.querySelectorAll(selector);
        elements = Array.prototype.slice.call(elements || []);
        if (includeExtra) {
            extraElements.forEach(function (element) {
                elements = elements.concat(querySelectorAll(element, selector, false));
            })
        }
        return elements;
    };

    var generate = function () {
        return function (util) {
            // disable worker
            workerURLs.forEach(function (url) {
                if (util.urlMatching(location.href, 'http.*://.*' + url + '.*')) {
                    window['Worker'] = undefined;
                    console.log('Worker disabled');
                }
            });
            var eHookContext = this;
            var timerHooker = {
                // 用于储存计时器的id和参数
                _intervalIds: {},
                // 计时器速率
                __percentage: 1.0,
                // 劫持前的原始的方法
                _setInterval: window['setInterval'],
                _clearInterval: window['clearInterval'],
                _clearTimeout: window['clearTimeout'],
                _setTimeout: window['setTimeout'],
                _Date: window['Date'],
                __lastDatetime: new Date().getTime(),
                __lastMDatetime: new Date().getTime(),
                videoSpeedInterval: 1000,
                /**
                 * 初始化方法
                 */
                init: function () {
                    var timerContext = this;
                    var h = helper(eHookContext, timerContext, util);

                    h.applyHooking();

                    // 设定百分比属性被修改的回调
                    Object.defineProperty(timerContext, '_percentage', {
                        get: function () {
                            return timerContext.__percentage;
                        },
                        set: function (percentage) {
                            if (percentage === timerContext.__percentage) {
                                return percentage;
                            }
                            h.percentageChangeHandler(percentage);
                            timerContext.__percentage = percentage;
                            return percentage;
                        }
                    });

                    if (!normalUtil.isInIframe()) {
                        console.log('[TimeHooker]', 'loading outer window...');
                        h.applyUI();
                        h.applyGlobalAction(timerContext);
                        h.registerShortcutKeys(timerContext);
                    } else {
                        console.log('[TimeHooker]', 'loading inner window...');
                        normalUtil.listenParentEvent((function (percentage) {
                            console.log('[TimeHooker]', 'Inner Changed', percentage)
                            this.change(percentage);
                        }).bind(this))
                    }
                },
                /**
                 * 调用该方法改变计时器速率
                 * @param percentage
                 */
                change: function (percentage) {
                    var _this = this;
                    this.__lastMDatetime = this._mDate.now();
                    // console.log(this._mDate.toString());
                    // console.log(new this._mDate());
                    this.__lastDatetime = this._Date.now();
                    // debugger;
                    //---------------------------------//
                    this._percentage = percentage;
                    var oldNode = document.getElementsByClassName('_th-click-hover');
                    var oldNode1 = document.getElementsByClassName('_th_times');
                    (oldNode[0] || {}).innerHTML = 'x' + 1 / this._percentage;
                    (oldNode1[0] || {}).innerHTML = 'x' + 1 / this._percentage;
                    var a = document.getElementsByClassName('_th_cover-all-show-times')[0] || {};
                    // console.log(a.className);
                    a.className = '_th_cover-all-show-times';
                    this._setTimeout.bind(window)(function () {
                        a.className = '_th_cover-all-show-times _th_hidden';
                    }, 100);
                    this.changeVideoSpeed();
                    this._clearInterval.bind(window)(this.videoSpeedIntervalId);
                    this.videoSpeedIntervalId = this._setInterval.bind(window)(function () {
                        _this.changeVideoSpeed();
                        var rate = 1 / _this._percentage;
                        if (rate === 1) {
                            _this._clearInterval.bind(window)(_this.videoSpeedIntervalId);
                        }
                    }, this.videoSpeedInterval);
                    normalUtil.sentChangesToIframe(percentage);
                },
                changeVideoSpeed: function () {
                    var rate = 1 / this._percentage;
                    rate > 16 && (rate = 16);
                    rate < 0.065 && (rate = 0.065);
                    // console.log(rate);
                    var videos = querySelectorAll(document, 'video', true) || [];
                    if (videos.length) {
                        for (var i = 0; i < videos.length; i++) {
                            videos[i].playbackRate = rate;
                        }
                    }
                }
            };
            // 默认初始化
            timerHooker.init();
            return timerHooker;
        }
    };

    if (global.eHook) {
        global.eHook.plugins({
            name: 'timer',
            /**
             * 插件装载
             * @param util
             */
            mount: generate()
        });
    }
}(window);
```

### 2. 网盘自动填写提取码，直链下载

选择工具栏上的Tempermonkey图标，点击添加新脚本；

复制以下代码到编辑框中，点击文件-保存

```javascript
// ==UserScript==
// @name         网盘助手
// @namespace    http://pan.newday.me/
// @version      0.3.7
// @icon         http://pan.newday.me/pan/favicon.ico
// @author       哩呵
// @description  大概是最优雅好用的网盘助手了；插件主要功能有：[1]百度网盘、腾讯微云、蓝奏云万能钥匙 [2]百度网盘生成并展示下载链接 [3]百度网盘分享时自定义提取码
// @match        *://pan.baidu.com/*
// @match        *://yun.baidu.com/*
// @match        *://share.weiyun.com/*
// @match        *://*.lanzous.com/*
// @match        *://*.newday.me/*
// @connect      api.newday.me
// @connect      ypsuperkey.meek.com.cn
// @require      https://cdn.staticfile.org/jquery/1.12.4/jquery.min.js
// @require      https://cdn.staticfile.org/snap.svg/0.5.1/snap.svg-min.js
// @require      https://cdn.staticfile.org/vue/2.6.6/vue.min.js
// @run-at       document-start
// @grant        unsafeWindow
// @grant        GM_getValue
// @grant        GM_setValue
// @grant        GM_listValues
// @grant        GM_openInTab
// @grant        GM_xmlhttpRequest
// ==/UserScript==

(function () {
    'use strict';

    var injectConfig = {
        name: "wpzs",
        version: "0.3.7",
        addon: {
            options_page: "/page/wpzs/option.html"
        },
        script: {
            options_page: "http://pan.newday.me/script/option.html"
        }
    };

    var container = (function () {
        var obj = {
            module_defines: {},
            module_objects: {}
        };

        obj.define = function (name, requires, callback) {
            name = obj.processName(name);
            obj.module_defines[name] = {
                requires: requires,
                callback: callback
            };
        };

        obj.require = function (name, cache) {
            if (typeof cache == "undefined") {
                cache = true;
            }

            name = obj.processName(name);
            if (cache && obj.module_objects.hasOwnProperty(name)) {
                return obj.module_objects[name];
            }
            else if (obj.module_defines.hasOwnProperty(name)) {
                var requires = obj.module_defines[name].requires;
                var callback = obj.module_defines[name].callback;

                var module = obj.use(requires, callback);
                cache && obj.register(name, module);
                return module;
            }
        };

        obj.use = function (requires, callback) {
            var module = {
                exports: {}
            };
            var params = obj.buildParams(requires, module);
            var result = callback.apply(this, params);
            if (typeof result != "undefined") {
                return result;
            }
            else {
                return module.exports;
            }
        };

        obj.register = function (name, module) {
            name = obj.processName(name);
            obj.module_objects[name] = module;
        };

        obj.buildParams = function (requires, module) {
            var params = [];
            requires.forEach(function (name) {
                params.push(obj.require(name));
            });
            params.push(obj.require);
            params.push(module.exports);
            params.push(module);
            return params;
        };

        obj.processName = function (name) {
            return name.toLowerCase();
        };

        return {
            define: obj.define,
            use: obj.use,
            register: obj.register,
            modules: obj.module_objects
        };
    })();

    container.define("gm", [], function () {
        var obj = {};

        obj.ready = function (callback) {
            if (typeof GM_getValue != "undefined") {
                callback && callback();
            }
            else {
                setTimeout(function () {
                    obj.ready(callback);
                }, 100);
            }
        };

        return obj;
    });

    container.define("runtime", [], function () {
        var obj = {
            url: location.href,
            referer: document.referrer,
        };

        obj.getUrl = function () {
            return obj.url;
        };

        obj.setUrl = function (url) {
            obj.url = url;
        };

        obj.getReferer = function () {
            return obj.referer;
        };

        obj.setReferer = function (referer) {
            obj.referer = referer;
        };

        obj.getUrlParam = function (name) {
            var param = obj.parseUrlParam(obj.getUrl());
            if (name) {
                return param.hasOwnProperty(name) ? param[name] : null;
            }
            else {
                return param;
            }
        };

        obj.parseUrlParam = function (url) {
            if (url.indexOf("?")) {
                url = url.split("?")[1];
            }
            var reg = /([^=&\s]+)[=\s]*([^=&\s]*)/g;
            var obj = {};
            while (reg.exec(url)) {
                obj[RegExp.$1] = RegExp.$2;
            }
            return obj;
        };

        return obj;
    });

    container.define("object", [], function () {
        var obj = {};

        obj.keys = function (data) {
            var list = [];
            for (var key in data) {
                list.push(key);
            }
            return list;
        };

        obj.values = function (data) {
            var list = [];
            for (var key in data) {
                list.push(data[key]);
            }
            return list;
        };

        return obj;
    });

    container.define("storage", [], function () {
        var obj = {};

        obj.getValue = function (name, defaultValue) {
            return GM_getValue(name, defaultValue);
        };

        obj.setValue = function (name, value) {
            GM_setValue(name, value);
        };

        obj.getValueList = function () {
            var nameList = GM_listValues();
            var valueList = {};
            nameList.forEach(function (name) {
                valueList[name] = obj.getValue(name);
            });
            return valueList;
        };

        return obj;
    });

    container.define("addon", ["storage", "constant"], function (storage, constant) {
        var obj = {
            name: constant.name + "_status"
        };

        obj.isEnable = function () {
            if (storage.getValue(obj.name) == "off") {
                return false;
            }
            else {
                return true;
            }
        };

        return obj;
    });

    container.define("config", ["storage", "constant"], function (storage, constant) {
        var obj = {
            name: "config_json"
        };

        obj.getConfig = function (name) {
            var configJson = storage.getValue(obj.name);
            var configObject = obj.parseJson(configJson);
            if (name) {
                name = obj.processName(name);
                return configObject.hasOwnProperty(name) ? configObject[name] : null;
            }
            else {
                return configObject;
            }
        };

        obj.setConfig = function (name, value) {
            var configObject = obj.getConfig();
            configObject[obj.processName(name)] = value;
            storage.setValue(obj.name, JSON.stringify(configObject));
        };

        obj.parseJson = function (jsonStr) {
            var jsonObject = {};
            try {
                if (jsonStr) {
                    jsonObject = JSON.parse(jsonStr);
                }
            }
            catch (e) { }
            return jsonObject;
        };

        obj.processName = function (name) {
            return constant.name + "_" + name;
        };

        return obj;
    });

    container.define("option", ["config", "constant", "object"], function (config, constant, object) {
        var obj = {
            name: "option",
            constant: constant.option
        };

        obj.isOptionActive = function (item) {
            var name = item.name;
            var option = obj.getOption();
            return option.indexOf(name) >= 0 ? true : false;
        };

        obj.setOptionActive = function (item) {
            var name = item.name;
            var option = obj.getOption();
            if (option.indexOf(name) < 0) {
                option.push(name);
                obj.setOption(option);
            }
        };

        obj.setOptionUnActive = function (item) {
            var name = item.name;
            var option = obj.getOption();
            var index = option.indexOf(name);
            if (index >= 0) {
                delete option[index];
                obj.setOption(option);
            }
        };

        obj.getOption = function () {
            var option = [];
            var optionObject = obj.getOptionObject();
            object.values(obj.constant).forEach(function (item) {
                var name = item.name;
                if (optionObject.hasOwnProperty(name)) {
                    if (optionObject[name] != "no") {
                        option.push(name);
                    }
                }
                else if (item.value != "no") {
                    option.push(name);
                }
            });
            return option;
        };

        obj.setOption = function (option) {
            var optionObject = {};
            object.values(obj.constant).forEach(function (item) {
                var name = item.name;
                if (option.indexOf(name) >= 0) {
                    optionObject[name] = "yes";
                } else {
                    optionObject[name] = "no";
                }
            });
            obj.setOptionObject(optionObject);
        };

        obj.getOptionObject = function () {
            var optionObject = config.getConfig(obj.name);
            return optionObject ? optionObject : {};
        };

        obj.setOptionObject = function (optionObject) {
            config.setConfig(obj.name, optionObject);
        };

        return obj;
    });

    container.define("mode", [], function () {
        var obj = {
            constant: {
                addon: "addon",
                script: "script"
            }
        };

        obj.getMode = function () {
            if (GM_info.addon) {
                return obj.constant.addon;
            }
            else {
                return obj.constant.script;
            }
        };

        return obj;
    });

    container.define("user", ["storage"], function (storage) {
        var obj = {};

        obj.getUid = function () {
            var uid = storage.getValue("uid");
            if (!uid) {
                uid = obj.randString(32);
                storage.setValue("uid", uid);
            }
            return uid;
        };

        obj.randString = function (length) {
            var possible = "abcdefghijklmnopqrstuvwxyz0123456789";
            var text = "";
            for (var i = 0; i < length; i++) {
                text += possible.charAt(Math.floor(Math.random() * possible.length));
            }
            return text;
        };

        return obj;
    });

    container.define("browser", [], function () {
        var obj = {
            constant: {
                firefox: "firefox",
                edge: "edge",
                baidu: "baidu",
                liebao: "liebao",
                uc: "uc",
                qq: "qq",
                sogou: "sogou",
                opera: "opera",
                maxthon: "maxthon",
                ie2345: "2345",
                se360: "360",
                chrome: "chrome",
                safari: "safari",
                other: "other"
            }
        };

        obj.getBrowser = function () {
            return obj.matchBrowserType(navigator.userAgent);
        };

        obj.matchBrowserType = function (userAgent) {
            var browser = obj.constant.other;
            userAgent = userAgent.toLowerCase();
            if (userAgent.match(/firefox/) != null) {
                browser = obj.constant.firefox;
            } else if (userAgent.match(/edge/) != null) {
                browser = obj.constant.edge;
            } else if (userAgent.match(/bidubrowser/) != null) {
                browser = obj.constant.baidu;
            } else if (userAgent.match(/lbbrowser/) != null) {
                browser = obj.constant.liebao;
            } else if (userAgent.match(/ubrowser/) != null) {
                browser = obj.constant.uc;
            } else if (userAgent.match(/qqbrowse/) != null) {
                browser = obj.constant.qq;
            } else if (userAgent.match(/metasr/) != null) {
                browser = obj.constant.sogou;
            } else if (userAgent.match(/opr/) != null) {
                browser = obj.constant.opera;
            } else if (userAgent.match(/maxthon/) != null) {
                browser = obj.constant.maxthon;
            } else if (userAgent.match(/2345explorer/) != null) {
                browser = obj.constant.ie2345;
            } else if (userAgent.match(/chrome/) != null) {
                if (obj.existMime("type", "application/vnd.chromium.remoting-viewer")) {
                    browser = obj.constant.se360;
                } else {
                    browser = obj.constant.chrome;
                }
            } else if (userAgent.match(/safari/) != null) {
                browser = obj.constant.safari;
            }
            return browser;
        };

        obj.existMime = function (option, value) {
            if (typeof navigator != "undefined") {
                var mimeTypes = navigator.mimeTypes;
                for (var mt in mimeTypes) {
                    if (mimeTypes[mt][option] == value) {
                        return true;
                    }
                }
            }
            return false;
        };

        return obj;
    });

    container.define("env", ["mode", "user", "browser", "constant"], function (mode, user, browser, constant) {
        var obj = {};

        obj.isAddon = function () {
            if (mode.getMode() == mode.constant.addon) {
                return true;
            }
            else {
                return false;
            }
        };

        obj.isInject = function () {
            if (obj.isAddon()) {
                if (GM_info.addon.name != constant.name) {
                    return true;
                }
                else {
                    return false;
                }
            }
            else {
                if (GM_info.script.alias && GM_info.script.alias != constant.name) {
                    return true;
                }
                else {
                    return false;
                }
            }
        };

        obj.getMode = function () {
            return mode.getMode();
        };

        obj.getAid = function () {
            if (GM_info.addon && GM_info.addon.id) {
                return GM_info.addon.id;
            }
            else if (GM_info.scriptHandler) {
                return GM_info.scriptHandler.toLowerCase();
            }
            else {
                return "unknown";
            }
        };

        obj.getUid = function () {
            return user.getUid();
        };

        obj.getVersion = function () {
            if (obj.isInject()) {
                return injectConfig.version;
            }
            else {
                return GM_info.script.version;
            }
        };

        obj.getBrowser = function () {
            return browser.getBrowser();
        };

        obj.getInfo = function () {
            return {
                mode: obj.getMode(),
                aid: obj.getAid(),
                uid: obj.getUid(),
                version: obj.getVersion(),
                browser: obj.getBrowser()
            };
        };

        return obj;
    });

    container.define("http", [], function () {
        var obj = {};

        obj.ajax = function (option) {
            var details = {
                method: option.type,
                url: option.url,
                responseType: option.dataType,
                onload: function (result) {
                    option.success && option.success(result.response);
                },
                onerror: function (result) {
                    option.error && option.error(result.error);
                }
            };

            // 提交数据
            if (option.data) {
                if (option.data instanceof FormData) {
                    details.data = option.data;
                }
                else {
                    var formData = new FormData();
                    for (var i in option.data) {
                        formData.append(i, option.data[i]);
                    }
                    details.data = formData;
                }
            }

            // 自定义头
            if (option.headers) {
                details.headers = option.headers;
            }

            // 超时
            if (option.timeout) {
                details.timeout = option.timeout;
            }

            GM_xmlhttpRequest(details);
        };

        return obj;
    });

    container.define("router", [], function () {
        var obj = {};

        obj.goUrl = function (url) {
            obj.runCode('location.href = "' + url + '";');
        };

        obj.openUrl = function (url) {
            obj.runCode('window.open("' + url + '");');
        };

        obj.openTab = function (url, active) {
            GM_openInTab(url, !active);
        };

        obj.runCode = function (script) {
            var node = document.createElementNS(document.lookupNamespaceURI(null) || "http://www.w3.org/1999/xhtml", "script");
            node.textContent = script;
            (document.head || document.body || document.documentElement || document).appendChild(node);
            node.parentNode.removeChild(node)
        };

        return obj;
    });

    container.define("logger", ["env", "constant"], function (env, constant) {
        var obj = {
            level: 3,
            constant: {
                debug: 0,
                info: 1,
                warn: 2,
                error: 3
            }
        };

        obj.debug = function (message) {
            obj.log(message, obj.constant.debug);
        };

        obj.info = function (message) {
            obj.log(message, obj.constant.info);
        };

        obj.warn = function (message) {
            obj.log(message, obj.constant.warn);
        };

        obj.error = function (message) {
            obj.log(message, obj.constant.error);
        };

        obj.log = function (message, level) {
            if (level < obj.level) {
                return false;
            }

            console.group("[" + constant.name + "]" + env.getMode());
            console.log(message);
            console.groupEnd();
        };

        obj.setLevel = function (level) {
            obj.level = level;
        };

        return obj;
    });

    container.define("meta", ["constant", "$"], function (constant, $) {
        var obj = {};

        obj.existMeta = function (name) {
            name = obj.processName(name);
            if ($("meta[name='" + name + "']").length) {
                return true;
            }
            else {
                return false;
            }
        };

        obj.appendMeta = function (name, content) {
            name = obj.processName(name);
            content || (content = "on");
            $('<meta name="' + name + '" content="on">').appendTo($("head"));
        };

        obj.processName = function (name) {
            return constant.name + "::" + name;
        };

        return obj;
    });

    container.define("unsafe_window", [], function () {
        if (typeof unsafeWindow == "undefined") {
            return window;
        }
        else {
            return unsafeWindow;
        }
    });

    container.define("svg_crypt", ["snap"], function (snap) {
        var obj = {};

        obj.getReqData = function () {
            var reqTime = Math.round(new Date().getTime() / 1000);
            var reqPoint = obj.getStrPoint("timestamp:" + reqTime);
            return {
                req_time: reqTime,
                req_point: reqPoint
            };
        };

        obj.getStrPoint = function (str) {
            if (str.length < 2) {
                return "0:0";
            }

            var path = "";
            var current, last = str[0].charCodeAt();
            var sum = last;
            for (var i = 1; i < str.length; i++) {
                current = str[i].charCodeAt();
                if (i == 1) {
                    path = path + "M";
                } else {
                    path = path + " L";
                }
                path = path + current + " " + last;
                last = current;
                sum = sum + current;
            }
            path = path + " Z";
            var index = sum % str.length;
            var data = snap.path.getPointAtLength(path, str[index].charCodeAt());
            return data.m.x + ":" + data.n.y;
        };

        return obj;
    });

    container.define("calendar", ["object"], function (object) {
        var obj = {};

        obj.formatTime = function (timestamp, format) {
            timestamp || (timestamp = (new Date()).getTime());
            format || (format = "Y-m-d H:i:s");
            var date = new Date(timestamp);
            var year = 1900 + date.getYear();
            var month = "0" + (date.getMonth() + 1);
            var day = "0" + date.getDate();
            var hour = "0" + date.getHours();
            var minute = "0" + date.getMinutes();
            var second = "0" + date.getSeconds();
            var vars = {
                "Y": year,
                "m": month.substring(month.length - 2, month.length),
                "d": day.substring(day.length - 2, day.length),
                "H": hour.substring(hour.length - 2, hour.length),
                "i": minute.substring(minute.length - 2, minute.length),
                "s": second.substring(second.length - 2, second.length)
            };
            return obj.replaceVars(vars, format);
        };

        obj.replaceVars = function (vars, value) {
            object.keys(vars).forEach(function (key) {
                value = value.replace(key, vars[key]);
            });
            return value;
        };

        return obj;
    });

    /** custom start **/
    container.define("constant", ["mode", "browser"], function (mode, browser) {
        return {
            name: injectConfig.name,
            mode: mode.constant,
            browser: browser.constant,
            addon: injectConfig.addon,
            script: injectConfig.script,
            source: {
                baidu: "baidu",
                weiyun: "weiyun",
                lanzous: "lanzous"
            },
            option: {
                send_usage: {
                    name: "send_usage",
                    value: "yes"
                },
                baidu_page_home: {
                    name: "baidu_page_home",
                    value: "yes"
                },
                baidu_page_share: {
                    name: "baidu_page_share",
                    value: "yes"
                },
                baidu_page_verify: {
                    name: "baidu_page_verify",
                    value: "yes"
                },
                baidu_share_status: {
                    name: "baidu_share_status",
                    value: "yes"
                },
                baidu_custom_password: {
                    name: "baidu_custom_password",
                    value: "yes"
                },
                baidu_show_origin: {
                    name: "baidu_show_origin",
                    value: "yes"
                },
                baidu_multi_link: {
                    name: "baidu_multi_link",
                    value: "no"
                },
                baidu_auto_jump: {
                    name: "baidu_auto_jump",
                    value: "no"
                },
                weiyun_page_verify: {
                    name: "weiyun_page_verify",
                    value: "yes"
                },
                weiyun_share_status: {
                    name: "weiyun_share_status",
                    value: "yes"
                },
                weiyun_auto_jump: {
                    name: "weiyun_auto_jump",
                    value: "no"
                },
                lanzous_page_verify: {
                    name: "lanzous_page_verify",
                    value: "yes"
                },
                lanzous_share_status: {
                    name: "lanzous_share_status",
                    value: "yes"
                },
                lanzous_auto_jump: {
                    name: "lanzous_auto_jump",
                    value: "no"
                }
            }
        };
    });

    container.define("share_log", ["object", "config", "constant"], function (object, config, constant) {
        var obj = {
            name: constant.name + "_share_list"
        };

        obj.addShareLog = function (shareId, sharePwd, shareLink, shareSource) {
            var shareList = obj.getShareLogList();
            shareList[shareId] = {
                share_id: shareId,
                share_pwd: sharePwd,
                share_link: shareLink,
                share_source: shareSource,
                share_time: (new Date()).getTime()
            };
            config.setConfig(obj.name, shareList);
        };

        obj.getShareLogList = function () {
            var shareList = config.getConfig(obj.name);
            return shareList ? shareList : {};
        };

        obj.buildShareLink = function (shareId, shareSource, shareLink) {
            if (shareSource == constant.source.baidu) {
                shareLink = "https://pan.baidu.com/s/1" + shareId;
            }
            else if (shareSource == constant.source.baidu) {
                shareLink = "https://share.weiyun.com/" + shareId;
            } else if (shareSource == constant.source.baidu) {
                shareLink = "https://www.lanzous.com/" + shareId;
            }
            return shareLink;
        };

        obj.buildShareTime = function (shareTime) {
            var date = new Date(shareTime);
            var year = 1900 + date.getYear();
            var month = "0" + (date.getMonth() + 1);
            var day = "0" + date.getDate();
            var hour = "0" + date.getHours();
            var minute = "0" + date.getMinutes();
            var second = "0" + date.getSeconds();
            var vars = {
                "Y": year,
                "m": month.substring(month.length - 2, month.length),
                "d": day.substring(day.length - 2, day.length),
                "H": hour.substring(hour.length - 2, hour.length),
                "i": minute.substring(minute.length - 2, minute.length),
                "s": second.substring(second.length - 2, second.length)
            };
            return obj.replaceVars(vars, "Y-m-d H:i:s");
        };

        obj.replaceVars = function (vars, value) {
            object.keys(vars).forEach(function (key) {
                value = value.replace(key, vars[key]);
            });
            return value;
        };

        return obj;
    });

    container.define("api", ["object", "http", "env", "config", "option", "svg_crypt", "share_log", "constant"], function (object, http, env, config, option, svgCrypt, shareLog, constant) {
        var obj = {
            base: "https://api.newday.me"
        };

        obj.versionQuery = function (callback) {
            obj.requestApi("/share/disk/version", {}, callback);
        };

        obj.queryShareOrigin = function (shareSource, shareId, callback) {
            obj.queryShare(shareSource, shareId, function (response) {
                if (response && response.referrer) {
                    var result = {
                        code: 1,
                        data: {
                            list: object.values(response.referrer)
                        }
                    };
                    callback && callback(result);
                }
                else {
                    callback && callback("");
                }
            });
        };

        obj.querySharePwd = function (shareSource, shareId, shareLink, callback) {
            var data = {
                share_id: shareId,
                share_point: svgCrypt.getStrPoint(shareId),
                share_link: shareLink
            };
            obj.requestApi("/share/disk/query", data, function (response) {
                if (response && response.code == 1) {
                    callback && callback(response);
                }
                else {
                    obj.querySharePwdYp(shareSource, shareId, callback);
                }
            });
        };

        obj.querySharePwdYp = function (shareSource, shareId, callback) {
            obj.queryShare(shareSource, shareId, function (response) {
                if (response && response.access_code) {
                    var result = {
                        code: 1,
                        data: {
                            share_pwd: response.access_code
                        }
                    };
                    callback && callback(result);
                }
                else {
                    callback && callback("");
                }
            });
        };

        obj.storeSharePwd = function (shareId, sharePwd, shareLink, shareSource, callback) {
            // 记录日志
            shareLog.addShareLog(shareId, sharePwd, shareLink, shareSource);

            var data = {
                share_id: shareId,
                share_pwd: sharePwd,
                share_point: svgCrypt.getStrPoint(shareId),
                share_link: shareLink
            };
            obj.requestApi("/share/disk/store", data, callback);
        };

        obj.logOption = function (callback) {
            var data = {
                app_id: config.getConfig("app_id"),
                temp_path: config.getConfig("temp_path"),
                option_json: JSON.stringify(option.getOption())
            };
            obj.requestApi("/share/disk/option", data, callback);
        };

        obj.queryShare = function (shareSource, shareId, callback) {
            var prefix = "BDY";
            if (shareSource == constant.source.lanzous) {
                prefix = "LZY";
            }
            var url = "https://ypsuperkey.meek.com.cn/api/v1/items/" + prefix + "-" + shareId + "?client_version=2019.2";
            http.ajax({
                type: "post",
                url: url,
                dataType: "json",
                success: function (response) {
                    callback && callback(response);
                },
                error: function () {
                    callback && callback("");
                }
            });
        };

        obj.requestApi = function (path, data, callback) {
            data.mode = env.getMode();
            data.aid = env.getAid();
            data.uid = env.getUid();
            data.version = env.getVersion();
            data.browser = env.getBrowser();

            http.ajax({
                type: "post",
                url: obj.base + path,
                dataType: "json",
                data: data,
                success: function (response) {
                    callback && callback(response);
                },
                error: function (error) {
                    callback && callback("");
                }
            });
        };

        return obj;
    });

    container.define("updater", ["config", "calendar", "api"], function (config, calendar, api) {
        var obj = {};

        obj.init = function () {
            var versionDate = config.getConfig("version_date");
            var currentDate = calendar.formatTime(null, "Ymd");
            if (!versionDate || versionDate < currentDate) {
                api.versionQuery(function (response) {
                    config.setConfig("version_date", currentDate);
                    if (response && response.code == 1) {
                        config.setConfig("version_latest", response.data.version);
                    }
                });
            }
        };

        return obj;
    });

    container.define("core", ["router", "env", "constant", "updater"], function (router, env, constant, updater) {
        var obj = {};

        obj.openPage = function (url, mode) {
            switch (mode) {
                case 9:
                    // self
                    router.goUrl(url);
                    break;
                case 6:
                    // new
                    router.openUrl(url);
                    break;
                case 3:
                    // new & not active
                    router.openTab(url, false);
                    break;
                case 1:
                    // new & active
                    router.openTab(url, true);
                    break;
            }
        };

        obj.openOptionPage = function () {
            if (env.isAddon()) {
                if (env.isInject()) {
                    router.openTab(injectConfig.addon.options_page, true);
                }
                else {
                    router.openTab(GM_info.addon.options_page, true);
                }
            }
            else {
                router.openTab(constant.script.options_page, true);
            }
        };

        obj.initVersion = function () {
            updater.init();
        };

        obj.ready = function (callback) {
            obj.initVersion();
            callback && callback();
        };

        return obj;
    });

    /** app start **/
    container.define("app_baidu", ["runtime", "config", "option", "router", "logger", "unsafe_window", "constant", "core", "api", "$"], function (runtime, config, option, router, logger, unsafeWindow, constant, core, api, $) {
        var obj = {
            app_id: 250528,
            temp_path: "/apps/temp",
            yun_data: null,
            verify_page: {
                share_pwd: null,
                setPwd: null,
                backupPwd: null,
                restorePwd: null,
                submit_pwd: null
            }
        };

        obj.run = function () {
            var url = runtime.getUrl();
            if (url.indexOf(".baidu.com/s/") > 0) {
                option.isOptionActive(option.constant.baidu_page_share) && obj.initSharePage();
                return true;
            }
            else if (url.indexOf(".baidu.com/disk/home") > 0) {
                option.isOptionActive(option.constant.baidu_page_home) && obj.initHomePage();
                return true;
            } else if (url.indexOf(".baidu.com/disk/timeline") > 0) {
                option.isOptionActive(option.constant.baidu_page_home) && obj.initTimeLinePage();
                return true;
            } else if (url.indexOf(".baidu.com/share/init") > 0) {
                option.isOptionActive(option.constant.baidu_page_verify) && obj.initVerifyPage();
                return true;
            }
            else {
                return false;
            }
        };

        obj.initSharePage = function () {
            obj.registerCustomAppId();

            obj.removeVideoLimit();

            obj.prettySingleSharePage();

            obj.initButtonShare();

            obj.initButtonEvent();

            if (option.isOptionActive(option.constant.baidu_show_origin)) {
                obj.showShareOrigin();
            }
        };

        obj.initHomePage = function () {
            obj.registerCustomAppId();

            obj.registerCustomSharePwd();

            obj.initButtonHome();

            obj.initButtonEvent();
        };

        obj.initTimeLinePage = function () {
            obj.registerCustomAppId();

            obj.registerCustomSharePwd();

            obj.initButtonTimeLine();

            obj.initButtonEvent();
        };

        obj.initVerifyPage = function () {
            obj.registerStoreSharePwd();

            if (obj.initVerifyPageElement()) {
                obj.autoPaddingSharePwd();

                obj.registerPwdShareSwitch();
            }
        };

        obj.initVerifyPageElement = function () {
            var shareId = obj.getShareId();
            var $pwd = $(".input-area input");
            if (shareId && $pwd.length) {
                // 设置提取码
                obj.verify_page.setPwd = function (pwd) {
                    $pwd.val(pwd);
                };

                // 备份提取码
                obj.verify_page.backupPwd = function (pwd) {
                    $pwd.attr("data-pwd", pwd);
                };

                // 还原提取码
                obj.verify_page.restorePwd = function () {
                    $pwd.val($pwd.attr("data-pwd"));
                };

                // 提交提取码
                var $button = $(".input-area .g-button");
                if ($button.length) {
                    obj.verify_page.submit_pwd = function () {
                        $button.click();
                    };
                }

                return true;
            }
            else {
                return false;
            }
        };

        obj.autoPaddingSharePwd = function () {
            var shareId = obj.getShareId();
            var shareLink = runtime.getUrl();
            api.querySharePwd(constant.source.baidu, shareId, shareLink, function (response) {
                if (response && response.code == 1) {
                    var sharePwd = response.data.share_pwd;
                    obj.verify_page.share_pwd = sharePwd;
                    obj.verify_page.setPwd(sharePwd);
                    obj.showTipSuccess("填充提取码成功");

                    if (option.isOptionActive(option.constant.baidu_auto_jump)) {
                        obj.verify_page.submit_pwd && obj.verify_page.submit_pwd();
                    }
                }
                else {
                    obj.showTipError("暂无人分享提取码");
                }
            });
        };

        obj.registerPwdShareSwitch = function () {
            // 添加开关
            $(".pickpw").after('<dl class="clearfix"><dt>提取码分享设置<span style="float:right"><input type="checkbox" checked id="nd-share-check" style="vertical-align: middle;"> <a class="nd-open-page-option" href="javascript:;" title="点击查看更多脚本配置">共享提取码</a></span></dt></dl>');
            obj.isPwdShareOpen() || $("#nd-share-check").removeAttr("checked");

            // 开关-事件
            $("#nd-share-check").on("change", function () {
                if ($(this).is(':checked')) {
                    option.setOptionActive(option.constant.baidu_share_status);
                }
                else {
                    option.setOptionUnActive(option.constant.baidu_share_status);
                }
            });

            // 打开配置页
            $(".nd-open-page-option").click(function () {
                core.openOptionPage();
            });
        };

        obj.registerStoreSharePwd = function () {
            obj.getJquery()(document).ajaxComplete(function (event, xhr, options) {
                var requestUrl = options.url;
                if (requestUrl.indexOf("/share/verify") >= 0) {
                    var match = options.data.match(/pwd=([a-z0-9]+)/i);
                    if (!match) {
                        return logger.warn("pwd share not match");
                    }

                    // 拒绝*号
                    if (obj.verify_page.backupPwd) {
                        obj.verify_page.backupPwd(match[1]);
                        setTimeout(obj.verify_page.restorePwd, 500);
                    }

                    var response = xhr.responseJSON;
                    if (!(response && response.errno == 0)) {
                        return logger.warn("pwd share error");
                    }

                    var sharePwd = match[1];
                    if (sharePwd == obj.verify_page.share_pwd) {
                        return logger.warn("pwd share not change");
                    }

                    if (!obj.isPwdShareOpen()) {
                        return logger.warn("pwd share closed");
                    }

                    var shareId = obj.getShareId();
                    var shareLink = runtime.getUrl();
                    api.storeSharePwd(shareId, sharePwd, shareLink, constant.source.baidu);
                }
            });
        };

        obj.registerCustomAppId = function () {
            obj.getJquery()(document).ajaxSend(function (event, xhr, options) {
                var requestUrl = options.url;
                if (requestUrl.indexOf("/api/download") >= 0 || requestUrl.indexOf("/api/sharedownload") >= 0) {
                    var match = requestUrl.match(/app_id=(\d+)/);
                    if (match) {
                        options.url = requestUrl.replace(match[0], "app_id=" + obj.getAppId());
                    }
                }
            });
        };

        obj.registerCustomSharePwd = function () {
            // 功能开关
            if (!option.isOptionActive(option.constant.baidu_custom_password)) {
                return;
            }

            // 生成提取码
            obj.async("function-widget-1:share/util/shareFriend/createLinkShare.js", function (shareLink) {
                shareLink.prototype.makePrivatePasswordOrigin = shareLink.prototype.makePrivatePassword;
                shareLink.prototype.makePrivatePassword = function () {
                    var sharePwd = config.getConfig("share_pwd");
                    return sharePwd ? sharePwd : this.makePrivatePasswordOrigin();
                };
            });

            // 分享事件
            obj.async("function-widget-1:share/util/shareDialog.js", function (shareDialog) {
                shareDialog.prototype.onVisibilityChangeOrigin = shareDialog.prototype.onVisibilityChange;
                shareDialog.prototype.onVisibilityChange = function (status) {
                    if ($(".nd-input-share-pwd").length == 0) {
                        var sharePwd = config.getConfig("share_pwd");
                        var html = '<tr><td class="first-child"><label>提取码</label></td><td><input type="text" class="nd-input-share-pwd" value="' + (sharePwd ? sharePwd : "") + '" placeholder="为空则随机四位" style="padding: 6px; width: 100px;border: 1px solid #e9e9e9;"></td></tr>';
                        $("#share .dialog-body table").append(html);
                    }
                    this.onVisibilityChangeOrigin(status);
                };
            });

            // 提取码更改事件
            $(document).on("change", ".nd-input-share-pwd", function () {
                var value = this.value;
                if (value && !value.match(/^[0-9a-z]{4}$/i)) {
                    obj.showTipError("提取码只能是四位数字或字母");
                }
                config.setConfig("share_pwd", value);
            });
        };

        obj.removeVideoLimit = function () {
            var message = obj.getSystemContext().message;
            if (message) {
                message.trigger("share-video-after-transfer");
            }
            else {
                logger.warn("wait removeVideoLimit...");
                obj.setTimeout(obj.removeVideoLimit, 500);
            }
        };

        obj.prettySingleSharePage = function () {
            if (!obj.isSharePageMulti()) {
                $("#layoutMain").css({
                    "width": "auto",
                    "min-width": "1180px",
                    "margin": "88px 30px"
                });
            }
        };

        obj.showShareOrigin = function () {
            var shareId = obj.getShareId();
            api.queryShareOrigin(constant.source.baidu, shareId, function (response) {
                if (response && response.code == 1) {
                    var data = response.data;
                    if (data.list && data.list.length) {
                        var html = '<div style="padding: 10px 5px; border-bottom: 1px solid #f6f6f6; line-height: 30px;">';
                        var item = data.list[0];
                        if (data.list.length > 1) {
                            html += '<p>分享来源：<a target="_blank" href="' + item.url + '">' + item.title + '</a> [<a class="show-origin-dialog" href="javascript:;" style="color:#ff0000;"> 查看更多 </a>]</p>';
                        }
                        else {
                            html += '<p>分享来源：<a target="_blank" href="' + item.url + '">' + item.title + '</a></p>';
                        }
                        html += '</div>';
                        $(".module-share-header").after(html);

                        $(document).on("click", ".show-origin-dialog", function () {
                            var title = "分享来源";
                            var body = '<div style="padding: 20px 20px;min-height: 120px; max-height: 300px; overflow-y: auto;">';

                            data.list.forEach(function (item, index) {
                                body += '<p>' + (++index) + '：<a target="_blank" href="' + item.url + '">' + item.title + '</a></p>';
                            });

                            body += '</div>';
                            var footer = obj.renderFooterAppId();
                            obj.showDialog(title, body, footer);
                        });
                    }
                    else {
                        obj.showTipError("暂未查询到分享的来源");
                    }
                }
            });
        };

        obj.initButtonShare = function () {
            if ($(".x-button-box").length) {
                var html = '<a class="g-button nd-button-build"><span class="g-button-right"><em class="icon icon-disk" title="下载"></em><span class="text">生成链接</span></span></a>';
                $(".x-button-box").append(html);
            }
            else {
                logger.warn("wait initButtonShare...");
                setTimeout(obj.initButtonShare, 500);
            }
        };

        obj.initButtonHome = function () {
            var listTools = obj.getSystemContext().Broker.getButtonBroker("listTools");
            if (listTools && listTools.$box) {
                var html = '<a class="g-button nd-button-build"><span class="g-button-right"><em class="icon icon-disk" title="下载"></em><span class="text">生成链接</span></span></a>';
                $(listTools.$box).prepend(html);
            }
            else {
                logger.warn("wait initButtonHome...");
                setTimeout(obj.initButtonHome, 500);
            }
        };

        obj.initButtonTimeLine = function () {
            if ($(".module-operateBtn .group-button").length) {
                var html = '<span class="button"><a class="g-v-button g-v-button-middle nd-button-build"><span class="g-v-button-right"><em class="icon icon-disk"></em><span class="text">生成链接</span></span></a></span>';
                $(".module-operateBtn .group-button").prepend(html);
            }
            else {
                logger.warn("wait initButtonTimeLine...");
                setTimeout(obj.initButtonTimeLine, 500);
            }
        };

        obj.initButtonEvent = function () {
            // 生成链接
            $(document).on("click", ".nd-button-build", function () {
                var yunData = obj.getYunData();
                if (yunData.MYUK) {
                    var fileList = obj.getSelectedFileList();
                    var fileStat = obj.getFileListStat(fileList);
                    if (fileList.length) {
                        if (fileList.length > 1 && fileStat.file_num) {
                            obj.showDownloadSelect(fileList, fileStat);
                        }
                        else if (fileStat.file_num == 1 && !obj.isHomePage()) {
                            obj.showDownloadSingle(fileList, fileStat);
                        }
                        else {
                            var pack = fileStat.file_num ? false : true;
                            if (obj.isHomePage()) {
                                obj.showDownloadInfoHome(fileList, pack);
                            }
                            else {
                                obj.showDownloadInfoShare(fileList, pack);
                            }
                        }
                    }
                    else {
                        obj.showTipError("请至少选择一个文件或文件夹");
                    }
                }
                else {
                    obj.showLogin();
                }
            });

            // 压缩包
            $(document).on("click", ".nd-button-pack", function () {
                var fileList = obj.getSelectedFileList();
                if (obj.isHomePage()) {
                    obj.showDownloadInfoHome(fileList, true);
                }
                else {
                    obj.showDownloadInfoShare(fileList, true);
                }
            });

            // 多文件
            $(document).on("click", ".nd-button-multi", function () {
                var fileList = obj.getSelectedFileList();

                // 过滤文件夹
                fileList = obj.filterFileListDir(fileList);

                if (obj.isHomePage()) {
                    obj.showDownloadInfoHomeOffical(fileList, false);
                }
                else {
                    obj.showDownloadInfoShareOffical(fileList, false);
                }
            });

            // 多文件
            $(document).on("click", ".nd-button-api", function () {
                var fileList = obj.getSelectedFileList();

                // 过滤文件夹
                fileList = obj.filterFileListDir(fileList);

                if (obj.isHomePage()) {
                    obj.showDownloadInfoHome(fileList, false);
                }
                else {
                    obj.showDownloadInfoShare(fileList, false);
                }
            });

            // 应用ID
            $(document).on("click", ".nd-change-app-id", function () {
                obj.showAppIdChange();
            });
            $(document).on("change", ".nd-input-app-id", function () {
                obj.setAppId(this.value);
            });

            // 打开配置页
            $(document).on("click", ".nd-open-page-option", function () {
                core.openOptionPage();
            });

            // 打开临时页面
            $(document).on("click", ".nd-open-page-temp", function () {
                router.openTab("https://pan.baidu.com/disk/home#/all?vmode=list&path=" + encodeURIComponent(obj.getTempPath()), true);
            });
        };

        obj.showLogin = function () {
            obj.getJquery()("[node-type='header-login-btn']").click();
        };

        obj.showDownloadInfoShare = function (fileList, pack) {
            logger.info(fileList);
            if (pack) {
                obj.showDownloadInfoShareOffical(fileList, pack);
            }
            else {
                obj.applyTransferFile(fileList, obj.getTempPath(), function (response) {
                    if (response && response.extra && response.extra.list) {
                        var listMap = {};
                        response.extra.list.forEach(function (item) {
                            listMap[item.from_fs_id] = item;
                        });

                        var downList = [];
                        fileList.forEach(function (item) {
                            if (listMap.hasOwnProperty(item.fs_id)) {
                                var fileName = listMap[item.fs_id].from.split("/").pop();
                                if (item.server_filename.indexOf(".") == 0) {
                                    item.server_filename = fileName;
                                }
                                item.dlink = obj.buildDownloadUrl(listMap[item.fs_id].to, fileName);
                                downList.push(item);
                            }
                        });
                        obj.showDownloadLinkFile(downList);
                    }
                });
            }
        };

        obj.showDownloadInfoShareOffical = function (fileList, pack) {
            obj.getDownloadShare(fileList, pack, function (response) {
                obj.hideTip();
                logger.info(response);

                if (response.list && response.list.length) {
                    // 文件
                    obj.showDownloadLinkFile(response.list);
                }
                else if (response.dlink) {
                    // 压缩包
                    obj.showDownloadLinkPack(fileList, {
                        dlink: response.dlink
                    });
                }
                else {
                    // 其他
                    obj.showDialogUnKnownResponse(response);
                }
            });
        };

        obj.showDownloadInfoHome = function (fileList, pack) {
            logger.info(fileList);
            if (pack) {
                obj.getDownloadHome(fileList, pack, function (response) {
                    obj.hideTip();
                    logger.info(response);

                    if (response.dlink && typeof response.dlink == "string") {
                        // 压缩包
                        obj.showDownloadLinkPack(fileList, {
                            dlink: response.dlink
                        });
                    }
                    else {
                        // 其他
                        obj.showDialogUnKnownResponse(response);
                    }
                });
            }
            else {
                fileList.forEach(function (item) {
                    item.dlink = obj.buildDownloadUrl(item.path, item.server_filename);
                });
                obj.showDownloadLinkFile(fileList);
            }
        };

        obj.showDownloadInfoHomeOffical = function (fileList, pack) {
            logger.info(fileList);
            obj.getDownloadHome(fileList, pack, function (response) {
                obj.hideTip();
                logger.info(response);

                if (response.dlink && typeof response.dlink == "object" && response.dlink.length) {
                    // 文件
                    var dlinkMapping = {};
                    response.dlink.forEach(function (item) {
                        dlinkMapping[item.fs_id] = item.dlink;
                    });

                    fileList.forEach(function (item) {
                        item.dlink = dlinkMapping[item.fs_id];
                    });
                    obj.showDownloadLinkFile(fileList);
                }
                else if (response.dlink && typeof response.dlink == "string") {
                    // 压缩包
                    obj.showDownloadLinkPack(fileList, {
                        dlink: response.dlink
                    });
                }
                else {
                    // 其他
                    obj.showDialogUnKnownResponse(response);
                }
            });
        };

        obj.showDownloadLinkFile = function (fileList) {
            var title = "文件下载";
            var body = '<div style="padding: 20px 20px;min-height: 120px; max-height: 300px; overflow-y: auto; ">';

            if (fileList.length > 1 && option.isOptionActive(option.constant.baidu_multi_link)) {
                var dlinkList = [];
                var dlinkApiList = [];
                fileList.forEach(function (item) {
                    dlinkList.push(item.dlink + "&filename=" + encodeURIComponent(item.server_filename));
                    item.dlink_api && dlinkApiList.push(item.dlink_api);
                });
                body += '<div style="margin-bottom: 10px;">';
                body += '<div>批量链接</div><div style="height: 80px; overflow: auto;"><textarea name="dlink_offical" style="width:2500px; border: 1px solid #f2f2f2;" rows="' + dlinkList.length + '">' + dlinkList.join("\n") + '</textarea></div>';
                body += '</div>';
            }

            fileList.forEach(function (item, index) {
                body += '<div style="margin-bottom: 10px;">';

                body += '<div>' + (index + 1) + '：' + item.server_filename + '</div>';

                body += '<div><a href="' + item.dlink + '&filename=' + encodeURIComponent(item.server_filename) + '" title="' + item.dlink + '" style="display:block; overflow:hidden; white-space:nowrap; text-overflow:ellipsis;">' + item.dlink + '</a></div>';

                body += '</div>';
            });
            body += '</div>';
            var footer = obj.renderFooterAppId();
            obj.showDialog(title, body, footer);
        };

        obj.showDownloadLinkPack = function (fileList, data) {
            var title = "文件下载";
            var body = '<div style="padding: 20px 20px;min-height: 120px; max-height: 300px; overflow-y: auto; ">';

            var packName = obj.getDownloadPackName(fileList);
            body += '<div>' + packName + '</div><div><a href="' + data.dlink + '&zipname=' + encodeURIComponent(packName) + '" title="' + data.dlink + '" style="display:block; overflow:hidden; white-space:nowrap; text-overflow:ellipsis;">' + data.dlink + '</a></div>';

            body += '<div style="margin-top: 15px;">打包的文件/文件夹列表</div>';
            fileList.forEach(function (item, index) {
                body += '<div title="' + item.path + '" style="color: ' + (item.isdir ? "blue" : "inherit") + ';">[' + (index + 1) + '] ' + item.server_filename + '</div>';
            });

            body += '</div>';
            var footer = obj.renderFooterAppId();
            obj.showDialog(title, body, footer);
        };

        obj.getDownloadPackName = function (fileList) {
            return fileList[0].server_filename + " 等" + fileList.length + "个文件.zip";
        };

        obj.buildDownloadUrl = function (path, name) {
            return "https://pcs.baidu.com/rest/2.0/pcs/file?method=download&app_id=" + obj.getAppId() + "&filename=" + encodeURIComponent(name) + "&path=" + encodeURIComponent(path);
        };

        obj.showDownloadSingle = function (fileList, fileStat) {
            var title = "链接类型";
            var body = '<div style="padding: 40px 20px; max-height: 300px; overflow-y: auto;">';

            body += '<div class="normalBtnBox g-center">';
            body += '<a class="g-button g-button-large g-button-gray-large nd-button-multi" title="调用官方接口生成链接"><span class="g-button-right"><em class="icon icon-download"></em> 官方链接</span></a>';
            body += '<a class="g-button g-button-large g-button-gray-large nd-button-api" style="margin-left:50px;" title="以转存文件的方式间接生成文件链接"><span class="g-button-right"><em class="icon icon-save-disk"></em> 转存链接</span></a>';
            body += '</div>';

            if (fileStat.dir_num) {
                body += '<div style="margin-top: 40px; padding-top: 10px; margin-bottom: -20px; border-top: 1px solid #D0DFE7;"><p class="g-center">选择 [多文件] 会过滤当前选中的 <span style="color: red">' + fileStat.dir_num + '</span> 个文件夹</p>';

                var index = 1;
                fileList.forEach(function (item) {
                    if (item.isdir) {
                        body += '<p title="' + item.path + '" style="color: blue;">[' + index + '] ' + item.server_filename + '</p>';
                        index++;
                    }
                });
                body += '</div>';
            }

            body += '</div>';
            var footer = obj.renderFooterAppId();
            obj.showDialog(title, body, footer);
        };

        obj.showDownloadSelect = function (fileList, fileStat) {
            var title = "链接类型";
            var body = '<div style="padding: 40px 20px; max-height: 300px; overflow-y: auto;">';

            body += '<div class="normalBtnBox g-center">';
            body += '<a class="g-button g-button-large g-button-gray-large nd-button-multi"><span class="g-button-right" title="调用官方接口生成文件链接"><em class="icon icon-download"></em> 官方多文件</span></a>';
            if (obj.isHomePage()) {
                body += '<a class="g-button g-button-large g-button-gray-large nd-button-api" style="margin-left:50px;" title="以构造链接的形式生成文件链接"><span class="g-button-right"><em class="icon icon-save-disk"></em> 接口多文件</span></a>';
            }
            else {
                body += '<a class="g-button g-button-large g-button-gray-large nd-button-api" style="margin-left:50px;" title="以转存文件的方式间接生成文件链接"><span class="g-button-right"><em class="icon icon-save-disk"></em> 转存多文件</span></a>';
            }
            body += '<a class="g-button g-button-large g-button-gray-large nd-button-pack" style="margin-left:50px;" title="调用官方接口生成压缩包链接"><span class="g-button-right"><em class="icon icon-poly"></em> 压缩包</span></a>';
            body += '</div>';

            if (fileStat.dir_num) {
                body += '<div style="margin-top: 40px; padding-top: 10px; margin-bottom: -20px; border-top: 1px solid #D0DFE7;"><p class="g-center">选择 [多文件] 会过滤当前选中的 <span style="color: red">' + fileStat.dir_num + '</span> 个文件夹</p>';
                var index = 1;
                fileList.forEach(function (item) {
                    if (item.isdir) {
                        body += '<p title="' + item.path + '" style="color: blue;">[' + index + '] ' + item.server_filename + '</p>';
                        index++;
                    }
                });
                body += '</div>';
            }

            body += '</div>';
            var footer = obj.renderFooterAppId();
            obj.showDialog(title, body, footer);
        };

        obj.showAppIdChange = function () {
            var title = "应用ID";
            var body = '<div style="padding: 60px 20px; max-height: 300px; overflow-y: auto;"><div class="g-center" style="margin-bottom: 10px;">当前应用ID：<input type="text" class="nd-input-app-id" style="border: 1px solid #f2f2f2; padding: 4px 5px;" value="' + obj.getAppId() + '"></div><div class="g-center"><p>如生成链接或者下载文件异常，请尝试修改为官方应用ID【' + obj.app_id + '】</p><p>修改应用ID可能存在未知的风险，请慎重使用，更多应用ID请查看<a target="_blank" href="https://greasyfork.org/zh-CN/scripts/378301"> 脚本主页 </a></p></div></div>';
            var footer = '';
            obj.showDialog(title, body, footer);
        };

        obj.showDialogUnKnownResponse = function (response) {
            var title = "未知结果";
            var body = '<div style="padding: 20px 20px; max-height: 300px; overflow-y: auto;"><pre style="white-space: pre-wrap; word-wrap: break-word; word-break: break-all;">' + JSON.stringify(response, null, 4) + '</pre></div>';
            var footer = obj.renderFooterAppId();
            obj.showDialog(title, body, footer);
        };

        obj.renderFooterAppId = function () {
            return '<p style="padding-top: 10px; border-top: 1px solid #D0DFE7;">应用ID：' + obj.getAppId() + ' <a href="javascript:;" class="nd-change-app-id">修改</a>，其他页面： <a class="nd-open-page-option" href="javascript:;">配置页面</a> 、<a class="nd-open-page-temp" href="javascript:;">临时文件</a></p>';
        };

        obj.showDialog = function (title, body, footer) {
            var dialog = obj.require("system-core:system/uiService/dialog/dialog.js").verify({
                title: title,
                img: "img",
                vcode: "vcode"
            });

            // 内容
            $(dialog.$dialog).find(".dialog-body").html(body);

            // 底部
            $(dialog.$dialog).find(".dialog-footer").html(footer);

            dialog.show();
        };

        obj.showTipSuccess = function (msg, hasClose, autoClose) {
            obj.showTip("success", msg, hasClose, autoClose);
        };

        obj.showTipError = function (msg, hasClose, autoClose) {
            obj.showTip("failure", msg, hasClose, autoClose);
        };

        obj.showTipLoading = function (msg, hasClose, autoClose) {
            obj.showTip("loading", msg, hasClose, autoClose);
        };

        obj.showTip = function (mode, msg, hasClose, autoClose) {
            var option = {
                mode: mode,
                msg: msg
            };

            // 关闭按钮
            if (typeof hasClose != "undefined") {
                option.hasClose = hasClose;
            }

            // 自动关闭
            if (typeof autoClose != "undefined") {
                option.autoClose = autoClose;
            }

            obj.require("system-core:system/uiService/tip/tip.js").show(option);
        };

        obj.hideTip = function () {
            obj.require("system-core:system/uiService/tip/tip.js").hide({
                hideTipsAnimationFlag: 1
            });
        };

        obj.isHomePage = function () {
            var url = runtime.getUrl();
            if (url.indexOf(".baidu.com/disk") > 0) {
                return true;
            }
            else {
                return false;
            }
        };

        obj.isTimelinePage = function () {
            var url = runtime.getUrl();
            if (url.indexOf(".baidu.com/disk/timeline") > 0) {
                return true;
            }
            else {
                return false;
            }
        };

        obj.isSharePageMulti = function () {
            var yunData = obj.getYunData();
            if (yunData.SHAREPAGETYPE == "single_file_page") {
                return false;
            }
            else {
                return true;
            }
        };

        obj.getSelectedFileList = function () {
            if (obj.isHomePage()) {
                return obj.getSelectedFileListHome();
            }
            else {
                return obj.getSelectedFileListShare();
            }
        };

        obj.getSelectedFileListHome = function () {
            if (obj.isTimelinePage()) {
                return obj.require("pan-timeline:widget/store/index.js").getters.getChoosedItemArr;
            }
            else {
                return obj.require('system-core:context/context.js').instanceForSystem.list.getSelected();
            }
        };

        obj.getSelectedFileListShare = function () {
            return obj.require('system-core:context/context.js').instanceForSystem.list.getSelected();
        };

        obj.getFileListStat = function (fileList) {
            var fileStat = {
                file_num: 0,
                dir_num: 0
            };
            fileList.forEach(function (item) {
                if (item.isdir == 0) {
                    fileStat.file_num++;
                }
                else {
                    fileStat.dir_num++;
                }
            });
            return fileStat;
        };

        obj.filterFileListDir = function (fileList) {
            var fileListFilter = [];
            fileList.forEach(function (item) {
                if (item.isdir == 0) {
                    fileListFilter.push(item);
                }
            });
            return fileListFilter;
        };

        obj.parseFidList = function (fileList) {
            var fidList = [];
            fileList.forEach(function (item) {
                fidList.push(item.fs_id);
            });
            return fidList;
        };

        obj.getDownloadShare = function (fileList, pack, callback) {
            obj.showTipLoading("生成链接中，请稍等...");
            obj.initWidgetContext("function-widget-1:download/util/context.js");
            obj.async("function-widget-1:download/service/dlinkService.js", function (dl) {
                var yunData = obj.getYunData();
                var data = {
                    list: fileList,
                    share_uk: yunData.SHARE_UK,
                    share_id: yunData.SHARE_ID,
                    sign: yunData.SIGN,
                    timestamp: yunData.TIMESTAMP,
                    type: pack ? "batch" : "nolimit"
                };
                dl.getDlinkShare(data, callback);
            });
        };

        obj.getDownloadHome = function (fileList, pack, callback) {
            obj.showTipLoading("生成链接中，请稍等...");
            obj.initWidgetContext("function-widget-1:download/util/context.js");
            obj.async("function-widget-1:download/service/dlinkService.js", function (dl) {
                var fidList = obj.parseFidList(fileList);
                var type = pack ? "batch" : "nolimit";
                dl.getDlinkPan(JSON.stringify(fidList), type, callback);
            });
        };

        obj.applyTransferFile = function (fileList, path, callback) {
            obj.listDir(path, function (response) {
                if (response && response.errno == 0) {
                    obj.transferFile(fileList, path, callback);
                }
                else if (response) {
                    obj.createDir(path, function (response) {
                        if (response && response.errno == 0) {
                            obj.transferFile(fileList, response.path, callback);
                        }
                        else {
                            callback && callback("");
                        }
                    });
                }
                else {
                    callback && callback("");
                }
            });
        };

        obj.transferFile = function (fileList, path, callback) {
            var yunData = obj.getYunData();
            var fidList = obj.parseFidList(fileList);
            var url = "/share/transfer?ondup=newcopy&async=1&shareid=" + yunData.SHARE_ID + "&from=" + yunData.SHARE_UK
            var data = {
                fsidlist: "[" + fidList.join(",") + "]",
                path: path
            };
            obj.ajax({
                type: "post",
                url: url,
                data: data,
                dataType: "json",
                timeout: 1e5,
                error: function () {
                    callback && callback("");
                },
                success: function (response) {
                    callback && callback(response);
                }
            });
        };

        obj.listDir = function (path, callback) {
            var url = "/api/list";
            obj.ajax({
                type: "get",
                url: url,
                data: {
                    order: "name",
                    desc: 0,
                    showempty: 0,
                    web: 1,
                    page: 1,
                    num: 10,
                    dir: path
                },
                dataType: "json",
                timeout: 1e5,
                error: function () {
                    callback && callback("");
                },
                success: function (response) {
                    callback && callback(response);
                }
            });
        };

        obj.createDir = function (path, callback) {
            var url = "/api/create?a=commit";
            obj.ajax({
                type: "post",
                url: url,
                data: {
                    path: path,
                    isdir: 1,
                    block_list: "[]"
                },
                dataType: "json",
                timeout: 1e5,
                error: function () {
                    callback && callback("");
                },
                success: function (response) {
                    callback && callback(response);
                }
            });
        };

        obj.getShareId = function () {
            var shareId = runtime.getUrlParam("surl");
            if (shareId) {
                return shareId;
            }
            else {
                var match = location.pathname.match(/\/s\/1(\S+)/);
                return match ? match[1] : null;
            }
        };

        obj.isPwdShareOpen = function () {
            return option.isOptionActive(option.constant.baidu_share_status);
        };

        obj.getYunData = function () {
            if (!obj.yun_data) {
                obj.yun_data = unsafeWindow.yunData;
            }
            return obj.yun_data;
        };

        obj.getTempPath = function () {
            var tempPath = config.getConfig("temp_path");
            if (tempPath) {
                return tempPath;
            }
            else {
                return obj.temp_path;
            }
        };

        obj.setTempPath = function (tempPath) {
            config.setConfig("temp_path", tempPath);
            api.logOption();
        };

        obj.getAppId = function () {
            var appId = config.getConfig("app_id");
            if (appId) {
                return appId;
            }
            else {
                return obj.app_id;
            }
        };

        obj.setAppId = function (appId) {
            config.setConfig("app_id", appId);
            api.logOption();
        };

        obj.initWidgetContext = function (name, callback) {
            var initFunc = function (widget) {
                if (!widget.getContext()) {
                    widget.setContext(obj.getSystemContext());
                }
                callback && callback();
            };
            if (callback) {
                obj.async(name, initFunc);
            }
            else {
                initFunc(obj.require(name));
            }
        };

        obj.ajax = function (option) {
            obj.getJquery().ajax(option);
        };

        obj.getSystemContext = function () {
            return obj.require("system-core:context/context.js").instanceForSystem;
        };

        obj.getJquery = function () {
            return obj.require("base:widget/libs/jquerypacket.js");
        };

        obj.require = function (name) {
            return unsafeWindow.require(name);
        };

        obj.async = function (name, callback) {
            unsafeWindow.require.async(name, callback);
        };

        return obj;
    });

    container.define("app_weiyun", ["runtime", "object", "option", "logger", "unsafe_window", "constant", "core", "api", "$"], function (runtime, object, option, logger, unsafeWindow, constant, core, api, $) {
        var obj = {
            axios: null,
            modal: null,
            store: null,
            inject_name: "_nd_inject_",
            webpack_require: null,
            verify_page: {
                setPwd: null,
                share_pwd: null,
                submit_pwd: null
            }
        };

        obj.run = function () {
            var url = runtime.getUrl();
            if (url.indexOf("share.weiyun.com") > 0) {
                option.isOptionActive(option.constant.weiyun_page_verify) && obj.initVerifyPage();
                return true;
            }
            else {
                return false;
            }
        };

        obj.initVerifyPage = function () {
            obj.initWebpackRequire(function () {
                obj.registerStoreSharePwd();
            });


            obj.initVerifyPageElement(function () {
                obj.autoPaddingSharePwd();
                obj.registerPwdShareSwitch();
            });
        };

        obj.initVerifyPageElement = function (callback) {
            var shareId = obj.getShareId();
            var $pwd = $(".card-inner .input-txt[type='password']");
            var $button = $(".card-inner .btn-main");
            if (shareId && $pwd.length && $button.length) {

                // 显示分享密码
                $pwd.attr("type", "text");

                // 设置分享密码
                obj.verify_page.setPwd = function (pwd) {
                    $pwd.val(pwd);
                };

                // 重造按钮
                var $itemButton = $button.parent();
                $itemButton.html($button.prop("outerHTML"));
                $button = $itemButton.find(".btn-main");

                // 按钮事件
                $button.on("click", function () {
                    obj.getStore() && obj.getStore().default.dispatch("shareInfo/loadShareInfoWithoutLogin", $pwd.val());
                });

                // 提交密码
                obj.verify_page.submit_pwd = function () {
                    $button.click();
                };

                callback && callback();
            }
            else {
                setTimeout(function () {
                    obj.initVerifyPageElement(callback)
                }, 500);
            }
        };

        obj.initWebpackRequire = function (callback) {
            var moreModules = {};
            moreModules[obj.inject_name] = function (module, exports, __webpack_require__) {
                obj.webpack_require = __webpack_require__;
                callback && callback();
            };
            unsafeWindow.webpackJsonp([obj.inject_name], moreModules, [obj.inject_name]);
        };

        obj.autoPaddingSharePwd = function () {
            var shareId = obj.getShareId();
            var shareLink = obj.getShareLink();
            api.querySharePwd(constant.source.weiyun, shareId, shareLink, function (response) {
                if (response && response.code == 1) {
                    var sharePwd = response.data.share_pwd;
                    obj.verify_page.share_pwd = sharePwd;
                    obj.verify_page.setPwd(sharePwd);
                    obj.showTipSuccess("填充密码成功");

                    if (option.isOptionActive(option.constant.weiyun_auto_jump)) {
                        obj.verify_page.submit_pwd && obj.verify_page.submit_pwd();
                    }
                }
                else {
                    obj.showTipError("暂无人分享密码");
                }
            });
        };

        obj.registerPwdShareSwitch = function () {
            // 添加开关
            $(".card-inner .form-item-label .form-item-tit").html('<span class="form-item-tit">请输入分享密码<span style="margin-left: 45px;"><input type="checkbox" checked id="nd-share-check" style="vertical-align: middle;"> <a class="nd-open-page-option" href="javascript:;" title="点击查看更多脚本配置">共享密码</a></span></span>');
            obj.isPwdShareOpen() || $("#nd-share-check").removeAttr("checked");

            // 开关-事件
            $("#nd-share-check").on("change", function () {
                if ($(this).is(':checked')) {
                    option.setOptionActive(option.constant.weiyun_share_status);
                }
                else {
                    option.setOptionUnActive(option.constant.weiyun_share_status);
                }
            });

            // 打开配置页
            $(".nd-open-page-option").click(function () {
                core.openOptionPage();
            });
        };

        obj.registerStoreSharePwd = function () {
            obj.addResponseInterceptor(function (request, response) {
                var requestUrl = request.responseURL;
                if (requestUrl.indexOf("weiyunShareNoLogin/WeiyunShareView") > 0) {
                    if (response.data.data.rsp_header.retcode == 0) {
                        var match = response.config.data.match(/\\"share_pwd\\":\\"([\w]+)\\"/);
                        if (!match) {
                            return logger.warn("pwd share not match");
                        }

                        var sharePwd = match[1];
                        if (sharePwd == obj.verify_page.share_pwd) {
                            return logger.warn("pwd share not change");
                        }

                        if (!obj.isPwdShareOpen()) {
                            return logger.warn("pwd share closed");
                        }

                        var shareId = obj.getShareId();
                        var shareLink = obj.getShareLink();
                        api.storeSharePwd(shareId, sharePwd, shareLink, constant.source.weiyun);
                    }
                    else {
                        return logger.warn("pwd share error");
                    }
                }
            });
        };

        obj.addResponseInterceptor = function (callback) {
            var success = function (response) {
                try {
                    callback && callback(response.request, response);
                }
                catch (e) {
                    logger.warn(e);
                }
                return response;
            };
            var error = function () {
                return Promise.reject(error);
            };
            obj.getAxios() && obj.getAxios().interceptors.response.use(success, error);
        };

        obj.showTipSuccess = function (msg) {
            obj.getModal() && obj.getModal().success(msg);
        };

        obj.showTipError = function (msg) {
            obj.getModal() && obj.getModal().error(msg);
        };

        obj.getShareId = function () {
            var url = runtime.getUrl();
            var match = url.match(/share.weiyun.com\/([0-9a-z]+)/i);
            return match ? match[1] : null;
        };

        obj.getShareLink = function () {
            return runtime.getUrl();
        };

        obj.isPwdShareOpen = function () {
            return option.isOptionActive(option.constant.weiyun_share_status);
        };

        obj.getAxios = function () {
            if (!obj.axios) {
                obj.axios = obj.matchWebpackModule(function (module, name) {
                    if (module && module.Axios) {
                        return module;
                    }
                });
            }
            return obj.axios;
        };

        obj.getModal = function () {
            if (!obj.modal) {
                obj.modal = obj.matchWebpackModule(function (module, name) {
                    if (module && module.confirm && module.success) {
                        return module;
                    }
                });
            }
            return obj.modal;
        };

        obj.getStore = function () {
            if (!obj.store) {
                obj.store = obj.matchWebpackModule(function (module, name) {
                    if (module && module.default && module.default._modulesNamespaceMap) {
                        return module;
                    }
                });
            }
            return obj.store;
        };

        obj.matchWebpackModule = function (matchFunc) {
            var names = object.keys(obj.webpack_require.c);
            for (var i in names) {
                var name = names[i];
                var match = matchFunc(obj.webpack_require(name), name);
                if (match) {
                    return match;
                }
            }
        };

        return obj;
    });

    container.define("app_lanzous", ["runtime", "option", "logger", "unsafe_window", "constant", "core", "api", "$"], function (runtime, option, logger, unsafeWindow, constant, core, api, $) {
        var obj = {
            verify_page: {
                setPwd: null,
                share_pwd: null,
                submit_pwd: null
            }
        };

        obj.run = function () {
            var url = runtime.getUrl();
            if (url.indexOf("lanzous.com") > 0) {
                option.isOptionActive(option.constant.lanzous_page_verify) && obj.initVerifyPage();
                return true;
            }
            else {
                return false;
            }
        };

        obj.initVerifyPage = function () {
            obj.registerStoreSharePwd();

            obj.initVerifyPageElement(function () {
                obj.autoPaddingSharePwd();

                obj.registerPwdShareSwitch();
            });
        };

        obj.initVerifyPageElement = function (callback) {
            var shareId = obj.getShareId();
            var $pwd = $("#pwd");
            if (shareId && $pwd.length) {

                // 设置分享密码
                obj.verify_page.setPwd = function (pwd) {
                    $pwd.val(pwd);
                };

                // 提交密码
                obj.verify_page.submit_pwd = function () {
                    $("#sub").click();
                };

                callback && callback();
            }
            else {
                setTimeout(function () {
                    obj.initVerifyPageElement(callback)
                }, 500);
            }
        };

        obj.autoPaddingSharePwd = function () {
            var shareId = obj.getShareId();
            var shareLink = obj.getShareLink();
            api.querySharePwd(constant.source.lanzous, shareId, shareLink, function (response) {
                if (response && response.code == 1) {
                    var sharePwd = response.data.share_pwd;
                    obj.verify_page.share_pwd = sharePwd;
                    obj.verify_page.setPwd(sharePwd);
                    obj.showTip(1, "填充密码成功", 2000);

                    if (option.isOptionActive(option.constant.lanzous_auto_jump)) {
                        obj.verify_page.submit_pwd && obj.verify_page.submit_pwd();
                    }
                }
                else {
                    obj.showTip(0, "暂无人分享密码", 2000);
                }
            });
        };

        obj.registerPwdShareSwitch = function () {
            var html = '<div style="text-align: center; margin-top: 10px;">分享设置 <input type="checkbox" checked id="nd-share-check" style="vertical-align: middle;" > <a style="cursor: pointer;" class="nd-open-page-option" href="javascript:;" title="点击查看更多脚本配置">共享密码</a></div>';
            if ($(".off").length) {
                $(".off").after(html);
            }
            else {
                $(".passwddiv-user").after(html);
            }
            obj.isPwdShareOpen() || $("#nd-share-check").removeAttr("checked");

            // 开关-事件
            $("#nd-share-check").on("change", function () {
                if ($(this).is(':checked')) {
                    option.setOptionActive(option.constant.lanzous_share_status);
                }
                else {
                    option.setOptionUnActive(option.constant.lanzous_share_status);
                }
            });

            // 打开配置页
            $(".nd-open-page-option").click(function () {
                core.openOptionPage();
            });
        };

        obj.registerStoreSharePwd = function () {
            unsafeWindow.$(document).ajaxComplete(function (event, xhr, options) {
                var match = options.data.match(/pwd=(\w+)/);
                if (!match) {
                    var match = options.data.match(/p=(\w+)/);
                    if (!match) {
                        return logger.warn("pwd share not match");
                    }
                }

                var sharePwd = match[1];

                if (sharePwd == obj.verify_page.share_pwd) {
                    return logger.warn("pwd share not change");
                }

                if (!obj.isPwdShareOpen()) {
                    return logger.warn("pwd share closed");
                }

                var shareId = obj.getShareId();
                var shareLink = obj.getShareLink();
                var response = obj.parseJson(xhr.response);
                if (response && response.zt == 1 && sharePwd) {
                    api.storeSharePwd(shareId, sharePwd, shareLink, constant.source.lanzous);
                }
                else {
                    logger.warn("pwd share error");
                }
            });
        };

        obj.showTip = function (code, msg, timeout) {
            if (unsafeWindow.sms) {
                unsafeWindow.sms(msg);
            }
            else {
                var selector;
                if ($(".off").length) {
                    selector = "#pwderr";
                }
                else {
                    selector = "#info";
                }
                if (code) {
                    $(selector).html('<span style="color: green;">' + msg + '</span>');
                }
                else {
                    $(selector).html('<span style="color: red;">' + msg + '</span>');
                }
                setTimeout(function () {
                    $(selector).html("");
                }, timeout);
            }
        };

        obj.getShareId = function () {
            return location.pathname.split("/")[1];
        };

        obj.getShareLink = function () {
            return top.location.href;
        };

        obj.isPwdShareOpen = function () {
            return option.isOptionActive(option.constant.lanzous_share_status);
        };

        obj.parseJson = function (jsonStr) {
            var jsonObject = {};
            try {
                if (jsonStr) {
                    jsonObject = JSON.parse(jsonStr);
                }
            }
            catch (e) { }
            return jsonObject;
        };

        return obj;
    });

    container.define("app_newday", ["object", "meta", "config", "option", "router", "env", "constant", "api", "share_log", "core", "$", "vue"], function (object, meta, config, option, router, env, constant, api, shareLog, core, $, vue) {
        var obj = {};

        obj.run = function () {
            if (meta.existMeta("info")) {
                obj.initInfoPage();
                return true;
            }
            else if (meta.existMeta("option")) {
                obj.initOptionPage();
                return true;
            }
            else if (meta.existMeta("share")) {
                obj.initSharePage();
                return true;
            }
            else if (meta.existMeta("dev")) {
                obj.initDevPage();
                return true;
            }
            else {
                return false;
            }
        };

        obj.initInfoPage = function () {
            new vue({
                el: "#container",
                data: {
                    info: env.getInfo()
                },
                created: function () {
                    obj.initAddonReady();
                }
            });
        };

        obj.initOptionPage = function () {
            new vue({
                el: "#container",
                data: {
                    app_id: config.getConfig("app_id"),
                    temp_path: config.getConfig("temp_path"),
                    info: env.getInfo(),
                    option: option.getOption()
                },
                created: function () {
                    obj.initAddonReady();
                },
                watch: {
                    option: function (value) {
                        option.setOption(value);
                        api.logOption();
                    },
                    app_id: function (value) {
                        config.setConfig("app_id", value);
                        value && api.logOption();
                    },
                    temp_path: function (value) {
                        config.setConfig("temp_path", value);
                        value && api.logOption();
                    }
                },
                methods: {
                    openTempPage: function () {
                        router.openTab("https://pan.baidu.com/disk/home#/all?vmode=list&path=" + encodeURIComponent(this.temp_path ? this.temp_path : "/apps/temp"), true);
                    }
                }
            });
        };

        obj.initSharePage = function () {
            var shareLogList = shareLog.getShareLogList();
            new vue({
                el: "#container",
                data: {
                    info: env.getInfo(),
                    list: []
                },
                created: function () {
                    obj.initAddonReady();

                    this.loadShareLogList("all");
                },
                methods: {
                    showShareLogList: function (source) {
                        $(".am-nav-tabs .am-active").removeClass("am-active");
                        $(".source-" + source).addClass("am-active");
                        this.loadShareLogList(source);
                    },
                    loadShareLogList: function (source) {
                        this.list = this.processShareLogList(source);
                    },
                    processShareLogList: function (source) {
                        var filterShareLogList = [];
                        object.keys(shareLogList).forEach(function (shareId) {
                            var item = shareLogList[shareId];
                            if (source == "all" || source == item.share_source) {
                                filterShareLogList.push({
                                    share_id: shareId,
                                    share_pwd: item.share_pwd,
                                    share_link: shareLog.buildShareLink(shareId, item.share_source, item.share_link),
                                    share_time: shareLog.buildShareTime(item.share_time),
                                    create_time: item.share_time
                                });
                            }
                        });
                        return filterShareLogList.sort(function (a, b) {
                            return b.create_time - a.create_time;
                        });
                    }
                }
            });
        };

        obj.initDevPage = function () {
            $("#dev-addon-info").val(JSON.stringify(env.getInfo()));

            $(".dev-open-page-option").addClass("nd-open-page-option").removeClass("dev-open-page-option");
            $(document).on("click", ".nd-open-page-option", function () {
                core.openOptionPage();
            });
        };

        obj.initAddonReady = function () {
            $("body").addClass("nd-addon-ready");
        };

        return obj;
    });

    container.define("app", ["runtime", "addon", "logger", "meta", "$"], function (runtime, addon, logger, meta, $, require) {
        var obj = {};

        obj.run = function () {
            var metaName = "status";
            if (meta.existMeta(metaName)) {
                logger.warn("setup already");
            }
            else if (addon.isEnable()) {
                logger.info("setup success");

                // 添加meta
                meta.appendMeta(metaName);

                // 运行应用
                $(obj.runApp);
            }
            else {
                logger.warn("addon disabled");
            }
        };

        obj.getAppList = function () {
            return [
                {
                    name: "app_baidu",
                    matchs: [
                        "baidu.com"
                    ]
                },
                {
                    name: "app_weiyun",
                    matchs: [
                        "weiyun.com"
                    ]
                },
                {
                    name: "app_lanzous",
                    matchs: [
                        "lanzous.com"
                    ]
                },
                {
                    name: "app_newday",
                    matchs: [
                        "*"
                    ]
                }
            ];
        };

        obj.runApp = function () {
            var url = runtime.getUrl();
            logger.info(url);

            var appList = obj.getAppList();
            for (var i in appList) {
                var app = appList[i];
                logger.debug(app);

                var match = obj.matchApp(url, app);
                logger.debug("match " + (match ? "yes" : "no"));

                if (match == false) {
                    continue;
                }

                logger.info("run " + app.name);
                if (require(app.name).run() == true) {
                    break;
                }
            }
        };

        obj.matchApp = function (url, app) {
            var match = false;
            app.matchs.forEach(function (item) {
                if (url.indexOf(item) > 0 || item == "*") {
                    match = true;
                }
            });
            return match;
        };

        return obj;
    });

    // lib
    container.define("$", [], function () {
        return window.$;
    });
    container.define("snap", [], function () {
        if (typeof Snap != "undefined") {
            return Snap;
        }
        else {
            return window.Snap;
        }
    });
    container.define("vue", [], function () {
        return window.Vue;
    });

    container.use(["gm", "core", "app", "logger", "share_log"], function (gm, core, app, logger, shareLog) {
        gm.ready(function () {
            // 日志级别
            logger.setLevel(logger.constant.info);

            core.ready(app.run);
        });
    });
})();
```

### 3. CSDN工具大全

选择工具栏上的Tempermonkey图标，点击添加新脚本；

复制以下代码到编辑框中，点击文件-保存

```javascript
// ==UserScript==
// @name         (持续更新)CSDN页面浮窗广告完全过滤净化(净化复制内容|自动展开|让你专注于文章|不影响功能使用)
// @namespace    https://github.com/AdlerED
// @version      2.1.9
// @description  ⚡️拥有数项独家功能的最强脚本，不服比一比⚡️|✔️CSDN体验秒杀AdBlock|✔️分辨率自适配，分屏不用滚动|✔️超级预优化|✔️独家超级免会员|✔️独家原创文章免登录展开|✔️独家推荐内容自由开关|✔️独家免登录复制|✔️独家防外链重定向|✔️独家论坛未登录自动展开文章、评论|✔️全面净化|✔️沉浸阅读|✔️净化剪贴板|✔️作者信息文章顶部展示
// @author       Adler
// @connect      www.csdn.net
// @include      *://*.csdn.net/*
// @require      https://cdnjs.cloudflare.com/ajax/libs/jquery-cookie/1.4.1/jquery.cookie.js
// @note         20-03-30 2.1.9 干掉“记录你的创作历程”，干掉未登录情况下的角标提醒
// @note         20-03-13 2.1.8 窄屏适配加强
// @note         20-03-13 2.1.7 更新简介
// @note         20-03-12 2.1.6 宽度自适应（感谢来自GitHub的朋友LeonG7的建议）！修复剪贴板净化无效的问题。
// @note         20-03-04 2.1.5 适配AdGuard
// @note         20-02-27 2.1.4 优化免登录复制
// @note         20-02-25 2.1.3 免登录复制更新，现已可用
// @note         20-02-24 2.1.2 By Github@JalinWang 更改去除剪贴板劫持的方式，使得原文格式在复制时能够保留
// @note         20-02-22 2.1.1 紧急修复由于 CSDN 修改前端结构导致的文章错位
// @note         20-02-11 2.1.0 若干动画优化，视觉体验更流畅
// @note         20-02-06 2.0.9 武汉加油！修改推荐内容切换开关位置，减少违和感
// @note         20-01-17 2.0.8 去除右侧广告
// @note         20-01-17 2.0.7 感谢来自GitHub的朋友“gleans”的建议，去掉页头广告
// @note         19-12-12 2.0.6 感谢来自GitHub的朋友“yexuesong”的建议，将作者信息在文章顶部展示
// @note         19-10-30 2.0.5 美化隐藏按钮，增加点击动画
// @note         19-10-30 2.0.4 删除CSDN官方在主页推送的文章（大多是广告）
// @note         19-10-30 2.0.3 添加更多屏蔽脚本
// @note         19-10-30 2.0.0 祝自己生日快乐~完全重写CSDNGreener，统一使用JQuery，效率更高
// @note         19-10-27 1.5.2 删除分享海报提示&&感谢GitHub的朋友“CHN-STUDENT”的反馈，去除底部课程推荐
// @note         19-10-27 1.5.1 感谢来自GitHub的朋友“CHN-STUDENT”的细致复现反馈，去除了底部的课程推荐广告
// @note         19-10-04 1.5.0 移除了底部主题信息和打赏
// @note         19-09-10 1.4.9 感谢来自GitHub的朋友“programmerZe”的细致复现反馈，修复了评论区点击查看回复后，已经展开的评论会收起的问题
// @note         19-09-04 1.4.8 感谢来自GitHub的朋友“dwdcth”的细致复现反馈，现在查看原创文章不会无限弹登录窗口了，且加强了自动展开功能
// @note         19-08-20 1.4.7 感谢油叉用户“SupremeSir”的反馈，修复了右侧悬浮栏遮挡文章的问题
// @note         19-08-14 1.4.6 无语。刚更新的免登录复制，又改了。修复！
// @note         19-08-13 1.4.5 更新了独家功能：免登录复制
// @note         19-08-13 1.4.4 感谢来自GitHub的朋友“iamsunxing”的反馈，修复了顶部不贴边的问题
// @note         19-08-01 1.4.3 感谢油叉用户“ddddy”的反馈，去除了更多推广广告
// @note         19-07-30 1.4.2 感谢油叉用户“周义杰”的反馈，增加了防CSDN外链重定向的功能（CSDN臭流氓）
// @note         19-07-20 1.4.1 修复了推荐内容开关跨文章无效问题（忘了配置Cookie作用域）
// @note         19-07-19 1.4.0 1. 构架大更新 2. 感谢来自GitHub的朋友"lukemin"的反馈，加入了下方推荐内容是否隐藏开关（实用）
// @note         19-07-13 1.3.0 感谢来自GitHub的朋友“Holaplace”的反馈，修复了文章无法自动展开的问题（CSDN总改这个，令人头疼）
// @note         19-06-08 1.2.6 感谢油叉用户“DeskyAki”的反馈，修复了文章无法自动展开的问题
// @note         19-06-07 1.2.4 修复了登录后评论无法正常打开的问题
// @note         19-06-07 1.2.3 感谢油叉用户"永远的殿下"的反馈，在一上午的努力攻克下，终于实现了未登录展开评论的语句
// @note         19-06-05 1.2.0 修复了评论无法自动展开的BUG
// @note         19-06-04 1.1.9 修复了无法自动展开的BUG（自闭了）
// @note         19-06-04 1.1.6 CSDN太坏了，把“消息”按钮的Class设置成了“GitChat”，所以修复了“消息”按钮消失的问题
// @note         19-06-04 1.1.5 1. 优化了论坛体验 2.美化、优化代码结构
// @note         19-06-04 1.1.4 感谢来自GitHub的朋友“iamsunxing”的反馈，增加了论坛广告匹配规则
// @note         19-06-03 1.1.3 感谢来自GitHub的朋友“wangwei135”的反馈，去除了评论区上方的广告
// @note         19-05-27 1.1.2 感谢油叉用户“夏伟杰”的反馈，修复了富文本编辑器无法使用的问题
// @note         19-05-25 1.1.0 1. 修复了主页广告的问题 2. 论坛自动展开 3. 论坛广告消除
// @note         19-05-25 1.0.9 感谢油叉用户“渣渣不准说话”的反馈，修复了收藏按钮消失的问题
// @note         19-03-01 1.0.3 添加页面选择性过滤规则
// @note         19-03-01 1.0.2 增加了净化剪贴板功能
// @note         19-03-01 1.0.1 修复了排版问题, 优化了代码结构
// @note         19-02-26 1.0.0 初版发布
// ==/UserScript==
var version = "2.1.9";
var currentURL = window.location.href;
var list;

(function () {
    'use strict';

    l("CSDNGreener V" + version);
    var blockURL = currentURL.split("/").length;
    var main = /(www\.csdn\.net\/)$/;
    var article = /article/;
    var bbs = /bbs\.csdn\.net/;
    var blog = /blog\.csdn\.net/;
    var blog2 = /\/article\/list\//;
    var download = /download\.csdn\.net/;
    var login = /passport\.csdn\.net/;
    var zone = /me\.csdn\.net/;

    // 数组初始化
    list = [];
    // 头部分
    // APP
    put(".app-app");
    // VIP
    put(".vip-caise");
    // 记录你的成长历程（记个毛）
    put("#writeGuide");
    // 新通知小圆点（未登录才消掉）
    if ($(".userinfo a").text() === '登录/注册') {
        put("#msg-circle");
    }
    // Cookie
    common(6, 1);

    if (main.test(currentURL)) {
        l("正在优化主页体验...");
        // 常规
        // 头部广告
        put(".banner-ad-box");
        // 右侧广告
        put(".slide-outer");
        // 右侧浮窗
        put(".csdn-side-toolbar");
        // 右侧详情
        put(".persion_article");
        clean(10);
        common(5, 10);
        loop(1);
    } else if ((blog.test(currentURL) && blockURL === 4) || blog2.test(currentURL)) {
        l("正在优化个人博客主页体验...");
        // 常规
        // 左侧广告
        put(".mb8");
        put("#kp_box_503");
        clean(10);
        common(5, 10);
    } else if (article.test(currentURL)) {
        l("正在优化阅读体验...");
        // 常规
        // 右侧广告，放到第一个清除
        put(".recommend-right");
        // 快来写博客吧
        put(".blog_tip_box");
        // 推荐关注用户
        put(".blog-expert-recommend-box");
        // 右下角VIP
        put(".meau-gotop-box");
        // 广告
        put(".mediav_ad");
        put(".pulllog-box");
        put(".recommend-ad-box");
        put(".box-shadow");
        put(".type_hot_word");
        put(".fourth_column");
        // 高分辨率时右侧文章推荐
        put(".right-item");
        // 广告
        put("#asideFooter");
        put("#ad-div");
        put("#479");
        put("#480");
        // 打赏
        put(".postTime");
        // 课程推荐
        put(".t0");
        // 分享海报
        put(".shareSuggest");
        // 底部主题
        put(".template-box");
        // 评论区广告
        put("div#dmp_ad_58");
        // 打赏
        put(".reward-user-box");
        // 右侧打赏按钮
        put(".to-reward");
        // 推荐内容广告
        put(".recommend-recommend-box");
        // 右侧广告
        put(".indexSuperise");
        clean(10);
        // 填充
        common(4, 5);
        // 评论
        common(1, 30);
        // 其它
        common(2, 20);
        // 顶部显示作者信息
        common(8, 1);
        // 循环线程开始
        loop(2);
    } else if (bbs.test(currentURL)) {
        l("正在优化论坛体验...");
        // 常规
        // 评论嵌入小广告
        put(".post_recommend");
        // 底部推荐
        put("#post_feed_wrap");
        // 底部相关文章里面的广告
        put(".bbs_feed_ad_box");
        put(".recommend-ad-box");
        // 底部相关文字里面的热词提示
        put(".type_hot_word");
        // 底部蓝色flex属性的广告栏+登录注册框
        put(".pulllog-box");
        // 猜你喜欢
        put(".personalized-recommend-box");
        // 发帖减半提示
        put(".totast-box");
        // 顶部广告
        put(".recommend-right");
        // 顶部广告
        put(".ad_top");
        clean(10);
        // 展开
        common(3, 50);
        common(5, 10);
    } else if (download.test(currentURL)) {
        l("正在优化下载页体验...");
        // 常规
        put(".fixed_dl");
        put("indexSuperise");
        clean(10);
        common(5, 10);
    } else if (login.test(currentURL)) {
        l("正在优化登录页体验...");
        // 常规
        // 登录界面大图广告
        put(".main-tu");
        clean(10);
        common(5, 10);
    } else if (zone.test(currentURL)) {
        l("正在优化个人空间体验...");
        // 常规
        clean(10);
        common(7, 10);
        common(5, 10);
    } else {
        e("不受支持的页面!");
    }
    l("超级优化完毕。");
    l("如果觉得好用，来 https://greasyfork.org/zh-CN/scripts/378351 收藏脚本来支持我吧！");
    l("开源&&提建议：https://github.com/AdlerED/CSDNGreener");
    l("我的博客：https://www.stackoverflow.wiki/");
    l("我的微信：1101635162");
})();

function l(log) {
    console.log("[CSDNGreener] " + log);
}

function e(error) {
    console.error("[CSDNGreener] " + error);
}

function clear() {
    list = [];
}

function put(tag) {
    list.push(tag);
}

function clean(times) {
    var loop = setInterval(function () {
        --times;
        if (times <= 0) {
            clearInterval(loop);
        }
        /* for (var j = 0; j < list.length; j++) {
            $(list[j]).hide(1000);
        }
        setInterval(function () {
        for (var k = 0; k < list.length; k++) {
            $(list[k]).remove();
        }
        }, 1000); */
        for (var k = 0; k < list.length; k++) {
            $(list[k]).remove();
        }
    }, 100);
}

function loop(num) {
    var did = false;
    setInterval(function () {
        if (num === 1) {
            // 主页中间的广告
            $(".J_adv").remove();
            // 主页有新的内容横条
            $(".feed-fix-box").remove();
            // 主页广告 iframe
            if (currentURL == "https://www.csdn.net/") {
                $("iframe").remove();
            }
            // 删除 CSDN 官方在主页的文章（大多是广告）
            $("li.clearfix").each(function(index, ele) {
                var banned = /csdn<\/a>/;
                var aMark = $(ele).find(".name").html();
                if (banned.test(aMark)) {
                    $(ele).remove();
                }
            });
        } else if (num === 2) {
            // 评论查看更多展开监听
            $("div.comment-list-box").css("max-height", "none");
            // 文章宽度自适应
            if (window.innerWidth < 1100) {
                $("article").width(window.innerWidth - 150);
                did = true;
            } else {
                if (did === true) {
                    $("article").removeAttr("style");
                    did = false;
                }
            }
        }
    }, 500);
}

function common(num, times) {
    var loop = setInterval(function () {
        --times;
        if (times <= 0) {
            clearInterval(loop);
        }
        if (num === 1) {
            // 查看更多
            $(".btn-readmore").removeClass("no-login");
            $(".btn-readmore").addClass("fans-read-more");
            $(".btn-readmore").removeAttr("href");
            $(".btn-readmore").removeAttr("target");
            $(".btn-readmore").removeAttr("rel");
            $(".btn-readmore").click();
            // 已登录用户展开评论
            document.getElementById("btnMoreComment").click();
            // 删除查看更多按钮
            $("#btnMoreComment").parent("div.opt-box").remove();
            // 展开内容
            $("div.comment-list-box").css("max-height", "none");
            // 改回背景颜色
            $(".login-mark").remove();
            // 删除登录框
            $(".login-box").remove();
        } else if (num === 2) {
            // 挡住评论的“出头推荐”
            if ($(".recommend-box").length > 1) {
                $(".recommend-box")[0].remove();
            }
            // 去外链
            $("#content_views").off();
            // 去除推广广告
            $("li[data-type='ad']").remove();
            // 免登录复制
            $(".hljs-button").removeClass("signin");
            $(".hljs-button").addClass("{2}");
            $(".hljs-button").attr("data-title", "免登录复制");
            // 去除剪贴板劫持
            $("code").attr("onclick", "mdcp.copyCode(event)");
            try {
                // 复制时保留原文格式，参考 https://greasyfork.org/en/scripts/390502-csdnremovecopyright/code
                Object.defineProperty(window, "articleType", {
                    value: 0,
                    writable: false,
                    configurable: false
                });
            } catch (err) {
            }
            csdn.copyright.init("", "", "");
            // 页头广告
            document.getElementsByClassName("column-advert-box")[0].style.display="none";
        } else if (num == 3) {
            //论坛自动展开
            $(".js_show_topic").click();
        } else if (num == 4) {
            // 左侧栏填充屏幕
            $(".blog_container_aside").hide();
            var screenWidth = document.body.clientWidth;
            if (screenWidth <= 1500) {
                $("main").css("cssText", "width:96% !important;");
            } else {
                $("main").css("cssText", "width:100% !important;");
            }
            // 右侧栏靠右
            $(".tool-box").css("right", "0px");
            $(".csdn-side-toolbar").css("right", "0px");
        } else if (num == 5) {
            // 改回背景颜色
            $(".login-mark").remove();
            // 删除登录框
            $(".login-box").remove();
        } else if (num == 6) {
            // 推荐内容开关 cookie
            var removeCookie = $.cookie("remove");
            var remove;
            if (removeCookie == undefined) {
                $.cookie('remove', true, {
                    path: '/'
                });
                remove = true;
            }
            if (removeCookie == "true") {
                remove = true;
            } else {
                remove = false;
            }
            // 删除推荐内容（自定义）
            if (remove) {
                $(".recommend-box").hide();
            }
            // 推荐内容开关
            $(".blog-content-box").append("<br><div class='blog-content-box' id='switch'></div>");
            //$(".comment-edit-box").after("<center><font size='1px'><a href='https://greasyfork.org/zh-CN/scripts/378351'>CSDN Greener V" + version + "</a><br><a href='https://github.com/AdlerED'>By GitHub :: AdlerED</a></font></center>");
            if (remove) {
                $("#switch").append("<button class='hide-recommend-button'>显示推荐内容</button>");
            } else {
                $("#switch").append("<button class='hide-recommend-button'>隐藏推荐内容</button>");
            }
            renderHideButton();
            // 开关监听
            $(".hide-recommend-button").click(function () {
                if ($.cookie('remove') == "true") {
                    $.cookie('remove', false, {
                        path: '/'
                    });
                    $(".recommend-box").slideDown(2000);
                    $(".hide-recommend-button").html("隐藏推荐内容");
                } else {
                    $.cookie('remove', true, {
                        path: '/'
                    });
                    $(".recommend-box").slideUp(1000);
                    $(".hide-recommend-button").html("显示推荐内容");
                }
                renderHideButton();
            });
        } else if (num === 7) {
            $(".me_r")[1].remove();
        } else if (num === 8) {
            $(".article-bar-top").append("<br>");
            $(".article-bar-top").append($(".aside-box-footerClassify").children("dd").html());
            $("dl").each(function (index, element) {
            var key = $(this).children("dt");
            var value = $(this).children("dd").children("span");
            if (key.html().indexOf("原创") != -1) {
            	key = $(this).children("dt").children("a")
            	value = $(this).children("dd").children("a").children("span");
            	addInfo(key, value);
            } else
            if (value.html() != undefined) {
            	addInfo(key, value);
            }
            } );
            function addInfo(key, value) {
            	var bind = key.html() + "&nbsp;" + value.html() + "&nbsp;&nbsp;";
            	$(".article-bar-top").append(bind + " ");
            }
            $(".blog_container_aside").remove();
        }
    }, 100);
}

function renderHideButton() {
    $(".hide-recommend-button").css({
        "width": "270px",
        "height": "40px",
        "border-width": "0px",
        "border-radius": "3px",
        "background": "#F7F7F7",
        "cursor": "pointer",
        "outline": "none",
        "font-family": "Microsoft YaHei",
        "color": "rgba(15,15,15,0.6)",
        "font-size": "17px",
        "display": "block",
        "margin": "0 auto"
    });
    $(".hide-recommend-button").hover(
        function () {
            $(".hide-recommend-button").css("background", "#EEEEEE");
        }, function () {
            $(".hide-recommend-button").css("background", "#F7F7F7");
        }
    );
}

```
