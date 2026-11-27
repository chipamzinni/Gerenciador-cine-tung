<!DOCTYPE html>
<html lang="pt-BR" style="--bg: #000000;"><head><script>window.__CINE_TUNG_FROM_REMOTE__=1;window.__CINE_TUNG_APP__=1;</script>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<script>
(function() {
    if (window.__htmlToApkCleartextHooked) return;
    window.__htmlToApkCleartextHooked = true;
    var PROXY_PREFIX = 'https://appassets.androidplatform.net/__cleartext_proxy__/?url=';
    function toProxyUrl(url) {
        if (!url) return url;
        if (typeof url === 'string') {
            if (url.startsWith('http://')) {
                return PROXY_PREFIX + encodeURIComponent(url);
            }
            return url;
        }
        try {
            var str = url.toString();
            if (str.startsWith('http://')) {
                return PROXY_PREFIX + encodeURIComponent(str);
            }
        } catch (_) {}
        return url;
    }
    function fromProxyUrl(url) {
        if (!url || typeof url !== 'string') return url;
        if (url.startsWith(PROXY_PREFIX)) {
            try {
                return decodeURIComponent(url.substring(PROXY_PREFIX.length));
            } catch (_) {}
        }
        return url;
    }
    if (typeof window.fetch === 'function') {
        var origFetch = window.fetch;
        window.fetch = function(input, init) {
            try {
                if (typeof input === 'string') {
                    input = toProxyUrl(input);
                } else if (typeof URL !== 'undefined' && input instanceof URL) {
                    input = toProxyUrl(input.toString());
                } else if (input && typeof input.url === 'string' && input.url.startsWith('http://')) {
                    var newUrl = toProxyUrl(input.url);
                    input = new Request(newUrl, input);
                }
            } catch (_) {}
            return origFetch.call(this, input, init);
        };
    }
    if (typeof window.XMLHttpRequest === 'function') {
        var origOpen = XMLHttpRequest.prototype.open;
        XMLHttpRequest.prototype.open = function(method, url, async, user, password) {
            this.__originalUrl = url;
            var proxied = toProxyUrl(typeof url === 'string' ? url : (url ? url.toString() : url));
            return origOpen.call(this, method, proxied, async !== false, user, password);
        };
    }
    try {
        var mediaProto = window.HTMLMediaElement && window.HTMLMediaElement.prototype;
        if (mediaProto) {
            var mediaSrcDesc = Object.getOwnPropertyDescriptor(mediaProto, 'src');
            if (mediaSrcDesc && mediaSrcDesc.set) {
                var origMediaSet = mediaSrcDesc.set;
                var origMediaGet = mediaSrcDesc.get;
                Object.defineProperty(mediaProto, 'src', {
                    get: function() {
                        return fromProxyUrl(origMediaGet.call(this));
                    },
                    set: function(val) {
                        return origMediaSet.call(this, toProxyUrl(val));
                    },
                    configurable: true,
                    enumerable: true
                });
            }
            var currentSrcDesc = Object.getOwnPropertyDescriptor(mediaProto, 'currentSrc');
            if (currentSrcDesc && currentSrcDesc.get) {
                var origCurrentSrcGet = currentSrcDesc.get;
                Object.defineProperty(mediaProto, 'currentSrc', {
                    get: function() {
                        return fromProxyUrl(origCurrentSrcGet.call(this));
                    },
                    configurable: true,
                    enumerable: true
                });
            }
        }
    } catch (_) {}
    try {
        var sourceProto = window.HTMLSourceElement && window.HTMLSourceElement.prototype;
        if (sourceProto) {
            var sourceSrcDesc = Object.getOwnPropertyDescriptor(sourceProto, 'src');
            if (sourceSrcDesc && sourceSrcDesc.set) {
                var origSourceSet = sourceSrcDesc.set;
                var origSourceGet = sourceSrcDesc.get;
                Object.defineProperty(sourceProto, 'src', {
                    get: function() {
                        return fromProxyUrl(origSourceGet.call(this));
                    },
                    set: function(val) {
                        return origSourceSet.call(this, toProxyUrl(val));
                    },
                    configurable: true,
                    enumerable: true
                });
            }
        }
    } catch (_) {}
    try {
        var origSetAttr = Element.prototype.setAttribute;
        var origGetAttr = Element.prototype.getAttribute;
        Element.prototype.setAttribute = function(name, val) {
            if (typeof name === 'string' && name.toLowerCase() === 'src') {
                var tag = this.tagName ? this.tagName.toLowerCase() : '';
                if (tag === 'video' || tag === 'audio' || tag === 'source' || tag === 'img' || tag === 'iframe') {
                    val = toProxyUrl(val);
                }
            }
            return origSetAttr.call(this, name, val);
        };
        Element.prototype.getAttribute = function(name) {
            var val = origGetAttr.call(this, name);
            if (typeof name === 'string' && name.toLowerCase() === 'src' && val) {
                return fromProxyUrl(val);
            }
            return val;
        };
    } catch (_) {}
    function rewriteMediaElement(el) {
        try {
            if (!el || !el.getAttribute) return;
            var rawSrc = el.getAttribute('src');
            if (rawSrc && rawSrc.startsWith('http://')) {
                el.src = toProxyUrl(rawSrc);
            }
        } catch (_) {}
    }
    function rewriteExistingTags() {
        try {
            var mediaEls = document.querySelectorAll('video, audio, source, img, iframe');
            for (var i = 0; i < mediaEls.length; i++) {
                rewriteMediaElement(mediaEls[i]);
            }
        } catch (_) {}
    }
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', rewriteExistingTags);
    } else {
        rewriteExistingTags();
    }
    try {
        if (typeof MutationObserver !== 'undefined') {
            var observer = new MutationObserver(function(mutations) {
                for (var m = 0; m < mutations.length; m++) {
                    var mutation = mutations[m];
                    if (mutation.type === 'childList') {
                        for (var a = 0; a < mutation.addedNodes.length; a++) {
                        var node = mutation.addedNodes[a];
                        if (node.nodeType === 1) {
                        var tag = node.tagName ? node.tagName.toLowerCase() : '';
                        if (tag === 'video' || tag === 'audio' || tag === 'source' || tag === 'img' || tag === 'iframe') {
                        rewriteMediaElement(node);
                        }
                        if (node.querySelectorAll) {
                        var children = node.querySelectorAll('video, audio, source, img, iframe');
                        for (var c = 0; c < children.length; c++) {
                        rewriteMediaElement(children[c]);
                        }
                        }
                        }
                        }
                    }
                }
            });
            observer.observe(document.documentElement || document, { childList: true, subtree: true });
        }
    } catch (_) {}
})();
(function() {
    if (window.__htmlToApkDownloadInterceptor) return;
    window.__htmlToApkDownloadInterceptor = true;
    // Popup'larda AndroidAPI yoktur; yalniz indirme yapabilen dar kopru verilir.
    function dlBridge() {
        if (typeof AndroidAPI !== 'undefined') return AndroidAPI;
        if (typeof HtmlToApkDownloads !== 'undefined') return HtmlToApkDownloads;
        return null;
    }
    var blobCache = (typeof Map !== 'undefined') ? new Map() : null;
    if (typeof URL !== 'undefined' && URL.createObjectURL) {
        var origCreate = URL.createObjectURL;
        URL.createObjectURL = function(blob) {
            var url = origCreate.call(URL, blob);
            if (blobCache && blob instanceof Blob) {
                try {
                    blobCache.set(url, blob);
                    setTimeout(function() {
                        try { blobCache.delete(url); } catch (_) {}
                    }, 60000);
                } catch (_) {}
            }
            return url;
        };
        if (URL.revokeObjectURL) {
            var origRevoke = URL.revokeObjectURL;
            URL.revokeObjectURL = function(url) {
                setTimeout(function() {
                    try {
                        origRevoke.call(URL, url);
                    } catch (_) {}
                }, 30000);
            };
        }
    }
    function guessFileName(href, name, mimeType) {
        if (name && typeof name === 'string' && name.trim() && name.trim() !== 'download') {
            var cleanName = name.trim();
            if (cleanName.indexOf('.') !== -1) return cleanName;
            var extFromName = (mimeType && mimeType.indexOf('json') !== -1) ? 'json' : 'bin';
            return cleanName + '.' + extFromName;
        }
        var ext = 'json';
        if (mimeType) {
            if (mimeType.indexOf('json') !== -1) ext = 'json';
            else if (mimeType.indexOf('csv') !== -1) ext = 'csv';
            else if (mimeType.indexOf('text') !== -1) ext = 'txt';
            else if (mimeType.indexOf('pdf') !== -1) ext = 'pdf';
            else if (mimeType.indexOf('zip') !== -1) ext = 'zip';
            else if (mimeType.indexOf('png') !== -1) ext = 'png';
            else if (mimeType.indexOf('jpeg') !== -1 || mimeType.indexOf('jpg') !== -1) ext = 'jpg';
        }
        return 'export_' + Date.now() + '.' + ext;
    }
    function handleDownload(href, filename, mimeType) {
        if (!href || typeof href !== 'string') return;
        var isBlob = href.startsWith('blob:');
        var isData = href.startsWith('data:');
        if (!isBlob && !isData) return;
        var safeName = guessFileName(href, filename, mimeType);
        if (isBlob && blobCache && blobCache.has(href)) {
            try {
                var cachedBlob = blobCache.get(href);
                var actualMime = (cachedBlob && cachedBlob.type) ? cachedBlob.type : (mimeType || 'application/json');
                var reader = new FileReader();
                reader.onloadend = function() {
                    if (reader.result && dlBridge() && dlBridge().onBlobDownloaded) {
                        dlBridge().onBlobDownloaded(reader.result, safeName, actualMime);
                    }
                };
                reader.onerror = function() {
                    if (dlBridge() && dlBridge().downloadBlobUrl) {
                        dlBridge().downloadBlobUrl(href, safeName, actualMime);
                    }
                };
                reader.readAsDataURL(cachedBlob);
                return;
            } catch (_) {}
        }
        var bridge = dlBridge();
        if (bridge) {
            if (isData && bridge.downloadDataUri) {
                bridge.downloadDataUri(href, safeName, mimeType || 'application/json');
            } else if (bridge.downloadBlobUrl) {
                bridge.downloadBlobUrl(href, safeName, mimeType || 'application/json');
            }
        }
    }
    document.addEventListener('click', function(e) {
        var a = e.target && e.target.closest ? e.target.closest('a') : null;
        if (!a || typeof a.href !== 'string') return;
        if (a.href.startsWith('blob:') || a.href.startsWith('data:')) {
            e.preventDefault();
            e.stopPropagation();
            var dl = a.getAttribute('download') || a.download || '';
            handleDownload(a.href, dl, a.type || '');
        }
    }, true);
    try {
        var origClick = HTMLAnchorElement.prototype.click;
        HTMLAnchorElement.prototype.click = function() {
            var href = this.href;
            if (typeof href === 'string' && (href.startsWith('blob:') || href.startsWith('data:'))) {
                var dl = this.getAttribute('download') || this.download || '';
                handleDownload(href, dl, this.type || '');
                return;
            }
            return origClick.apply(this, arguments);
        };
    } catch (_) {}
    try {
        var origDispatch = HTMLAnchorElement.prototype.dispatchEvent;
        HTMLAnchorElement.prototype.dispatchEvent = function(event) {
            if (event && (event.type === 'click' || event === 'click')) {
                var href = this.href;
                if (typeof href === 'string' && (href.startsWith('blob:') || href.startsWith('data:'))) {
                    var dl = this.getAttribute('download') || this.download || '';
                    handleDownload(href, dl, this.type || '');
                    return false;
                }
            }
            return origDispatch.apply(this, arguments);
        };
    } catch (_) {}
    try {
        var origOpen = window.open;
        window.open = function(url) {
            if (typeof url === 'string' && (url.startsWith('blob:') || url.startsWith('data:'))) {
                handleDownload(url, 'download', '');
                return null;
            }
            return origOpen.apply(this, arguments);
        };
    } catch (_) {}
    if (!window.showSaveFilePicker) {
        window.showSaveFilePicker = async function(options) {
            var suggestedName = (options && options.suggestedName) ? options.suggestedName : 'export.json';
            var mime = 'application/json';
            if (options && options.types && options.types[0] && options.types[0].accept) {
                var keys = Object.keys(options.types[0].accept);
                if (keys.length > 0) mime = keys[0];
            }
            return {
                name: suggestedName,
                createWritable: async function() {
                    var chunks = [];
                    return {
                        write: async function(data) {
                        if (data instanceof Blob) {
                        chunks.push(data);
                        } else if (typeof data === 'string') {
                        chunks.push(new Blob([data], { type: mime }));
                        } else if (data instanceof ArrayBuffer || ArrayBuffer.isView(data)) {
                        chunks.push(new Blob([data], { type: mime }));
                        }
                        },
                        close: async function() {
                        var finalBlob = new Blob(chunks, { type: mime });
                        var reader = new FileReader();
                        reader.onloadend = function() {
                        if (reader.result && dlBridge() && dlBridge().onBlobDownloaded) {
                        dlBridge().onBlobDownloaded(reader.result, suggestedName, mime);
                        }
                        };
                        reader.readAsDataURL(finalBlob);
                        },
                        abort: async function() {
                        chunks = [];
                        }
                    };
                }
            };
        };
    }
})();
window.print = function() { AndroidAPI.printPage(); };
(function() {
    if (window.__htmlToApkWebApiShim || typeof AndroidAPI === 'undefined') return;
    window.__htmlToApkWebApiShim = true;
    var seq = 0;
    function nextId(prefix) { seq += 1; return prefix + '_' + Date.now() + '_' + seq; }
    function once(eventName, match, timeoutMs) {
        return new Promise(function(resolve, reject) {
            var timer = null;
            function handler(e) {
                var d = e && e.detail;
                if (match && !match(d)) return;
                window.removeEventListener(eventName, handler);
                if (timer) clearTimeout(timer);
                resolve(d || {});
            }
            window.addEventListener(eventName, handler);
            if (timeoutMs) {
                timer = setTimeout(function() {
                    window.removeEventListener(eventName, handler);
                    reject(new DOMException('Timed out', 'AbortError'));
                }, timeoutMs);
            }
        });
    }
    // --- navigator.share ---
    if (!navigator.share) {
        var safeFileName = function(name) {
            var n = String(name || 'file').replace(/[\\\\/:*?"<>|]/g, '_').replace(/\.\.+/g, '_');
            return n || 'file';
        };
        var shareResult = function() {
            return once('shareResult', null, 30000).then(function(d) {
                if (!d.success) throw new DOMException(d.error || 'share_failed', 'AbortError');
            });
        };
        navigator.canShare = function(data) {
            if (!data) return false;
            if (data.files && data.files.length) return data.files.length === 1;
            return !!(data.title || data.text || data.url);
        };
        navigator.share = function(data) {
            data = data || {};
            if (!navigator.canShare(data)) {
                return Promise.reject(new TypeError('navigator.share: nothing to share'));
            }
            if (data.files && data.files.length) {
                var file = data.files[0];
                return new Promise(function(resolve, reject) {
                    var reader = new FileReader();
                    reader.onload = function() {
                        var pending = shareResult();
                        AndroidAPI.shareFileFromBase64(safeFileName(file.name), String(reader.result),
                        file.type || 'application/octet-stream');
                        pending.then(resolve, reject);
                    };
                    reader.onerror = function() { reject(new DOMException('read_failed', 'DataError')); };
                    reader.readAsDataURL(file);
                });
            }
            var text = [data.title, data.text, data.url].filter(function(v) { return !!v; }).join('\n');
            var pending = shareResult();
            AndroidAPI.shareText(text);
            return pending;
        };
    }
    // --- Notification ---
    if (typeof window.Notification === 'undefined') {
        var NOTIF_PERMISSION = 'android.permission.POST_NOTIFICATIONS';
        var cachedPermission = null;
        var currentPermission = function() {
            if (cachedPermission) return cachedPermission;
            try { return AndroidAPI.hasPermission(NOTIF_PERMISSION) ? 'granted' : 'default'; } catch (_) { return 'default'; }
        };
        var ShimNotification = function(title, options) {
            if (!(this instanceof ShimNotification)) throw new TypeError("Failed to construct 'Notification'");
            options = options || {};
            this.title = String(title);
            this.body = options.body ? String(options.body) : '';
            this.tag = options.tag || '';
            this.data = options.data;
            this.onclick = null; this.onshow = null; this.onerror = null; this.onclose = null;
            this._listeners = {};
            this._id = null;
            var self = this;
            if (currentPermission() !== 'granted') {
                setTimeout(function() { self._fire('error'); }, 0);
                return;
            }
            once('notificationResult', function(d) { return d && d.action === 'post'; }, 10000).then(function(d) {
                if (d.success) { self._id = d.notificationId; self._fire('show'); } else { self._fire('error'); }
            }, function() { self._fire('error'); });
            AndroidAPI.showNotification(this.title, this.body, '');
        };
        ShimNotification.prototype.addEventListener = function(type, fn) {
            (this._listeners[type] = this._listeners[type] || []).push(fn);
        };
        ShimNotification.prototype.removeEventListener = function(type, fn) {
            var list = this._listeners[type] || [];
            var i = list.indexOf(fn);
            if (i >= 0) list.splice(i, 1);
        };
        ShimNotification.prototype._fire = function(type) {
            var ev = { type: type, target: this };
            var handler = this['on' + type];
            if (typeof handler === 'function') { try { handler.call(this, ev); } catch (_) {} }
            (this._listeners[type] || []).slice().forEach(function(fn) { try { fn.call(this, ev); } catch (_) {} }, this);
        };
        ShimNotification.prototype.close = function() {
            if (this._id !== null) { try { AndroidAPI.cancelNotification(this._id); } catch (_) {} }
            this._fire('close');
        };
        Object.defineProperty(ShimNotification, 'permission', { get: currentPermission });
        ShimNotification.maxActions = 0;
        ShimNotification.requestPermission = function(callback) {
            var id = nextId('webNotification');
            var pending = once('permissionResult', function(d) { return d && d.callbackId === id; }, 120000)
                .then(function(d) { cachedPermission = d.granted ? 'granted' : 'denied'; return cachedPermission; },
                      function() { return currentPermission(); });
            AndroidAPI.requestNotificationPermission(id);
            if (typeof callback === 'function') pending.then(callback);
            return pending;
        };
        window.Notification = ShimNotification;
    }
    // --- screen.orientation.lock / unlock ---
    if (typeof HtmlToApkWebCompat !== 'undefined' && window.screen && screen.orientation) {
        try {
            screen.orientation.lock = function(type) {
                var error = HtmlToApkWebCompat.lockOrientation(String(type));
                if (error) {
                    return Promise.reject(error === 'invalid_orientation'
                        ? new TypeError('Invalid orientation: ' + type)
                        : new DOMException(error, 'NotSupportedError'));
                }
                return Promise.resolve();
            };
            screen.orientation.unlock = function() { HtmlToApkWebCompat.unlockOrientation(); };
        } catch (_) {}
    }
})();
</script>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Cine Tung</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=Instrument+Serif:ital@0;1&display=swap');:root {
--bg: #0a0a0b;--bg2: #111113;--panel: #161618;--panel2: #1c1c1f;--line: rgba(255,255,255,.08);--line2: rgba(255,255,255,.12);--text: #f5f5f4;--muted: #a1a1aa;--accent: #e8a87c;--accent-dim: rgba(232,168,124,.15);--accent2: #c4a574;--danger: #f07178;--ok: #7fd99a;--shadow: 0 12px 40px rgba(0,0,0,.45);--radius: 16px;--radius-sm: 10px;--ease: cubic-bezier(.22,1,.36,1);--font: "DM Sans", system-ui, sans-serif;--serif: "Instrument Serif", Georgia, serif;}
* {
 box-sizing: border-box; }
html {
 -webkit-tap-highlight-color: transparent; }
body {
margin: 0;font-family: var(--font);background: var(--bg);color: var(--text);min-height: 100vh;line-height: 1.45;background-image:radial-gradient(ellipse 80% 50% at 50% -20%, rgba(232,168,124,.07), transparent),radial-gradient(ellipse 60% 40% at 100% 0%, rgba(120,100,80,.04), transparent);background-attachment: fixed;}
button, input, textarea, select {
 font-family: inherit; }
.hidden {
 display: none !important; }
#splash {
position: fixed; inset: 0; z-index: 99999;background: #050505;display: flex; flex-direction: column; align-items: center; justify-content: center;transition: opacity .6s var(--ease), visibility .6s;}
#splash.hide {
 opacity: 0; visibility: hidden; pointer-events: none; }
#splashLogo {
font-family: var(--serif);font-size: clamp(42px, 11vw, 72px);letter-spacing: .28em;color: #fff;min-height: 1.2em;font-weight: 400;text-indent: .28em;}
#splashCredit {
margin-top: 18px; font-size: 12px; letter-spacing: .2em;text-transform: uppercase; color: var(--muted);opacity: 0; transition: opacity 1.1s var(--ease);}
#splashCredit.show {
 opacity: 1; }
#splashHint {
 margin-top: 28px; color: #555; font-size: 12px; letter-spacing: .06em; }
#viewApp {
 padding-bottom: 88px; display: none; }
#viewApp.on {
 display: block !important; visibility: visible !important; animation: fadeUp .45s var(--ease); min-height: 100vh; }
@keyframes fadeUp {
from {
 opacity: 0; transform: translateY(8px); }
to {
 opacity: 1; transform: none; }
}
@keyframes fadeIn {
from {
 opacity: 0; }
 to {
 opacity: 1; }
}
@keyframes scaleIn {
from {
 opacity: 0; transform: scale(.96); }
to {
 opacity: 1; transform: none; }
}
.top {
position: sticky; top: 0; z-index: 20;background: rgba(10,10,11,.82);backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px);border-bottom: 1px solid var(--line);padding: 14px 18px 12px;}
.top-row {
display: flex; align-items: center; justify-content: space-between;gap: 10px; margin-bottom: 12px;}
.brand {
font-family: var(--serif);font-weight: 400;font-size: 22px;letter-spacing: .04em;}
.brand span {
 color: var(--accent); font-style: italic; }
.top-right {
 display: flex; align-items: center; gap: 10px; }
.crown-btn {
width: 38px; height: 38px; border-radius: 50%;border: 1px solid rgba(232,168,124,.35);background: linear-gradient(145deg, #2a2218, #14110c);color: var(--accent);font-size: 16px; cursor: pointer;display: flex; align-items: center; justify-content: center;transition: transform .25s var(--ease), box-shadow .25s;}
.crown-btn:active {
 transform: scale(.92); }
.ico-crown, .ico-lib, .ico-movie, .ico-series, .ico-user, .ico-search {
display: inline-block; width: 18px; height: 18px; vertical-align: -3px;background: currentColor; opacity: .9;}
.ico-crown {
width: 16px; height: 14px;mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M3 19h18v2H3v-2zm2-6l3-7 4 5 4-5 3 7H5z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M3 19h18v2H3v-2zm2-6l3-7 4 5 4-5 3 7H5z'/%3E%3C/svg%3E") center/contain no-repeat;background: var(--accent, #e8b86d);}
.ico-lib {
mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M4 4h5v16H4V4zm6 0h5v16h-5V4zm6 0h4v16h-4V4z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M4 4h5v16H4V4zm6 0h5v16h-5V4zm6 0h4v16h-4V4z'/%3E%3C/svg%3E") center/contain no-repeat;}
.ico-movie {
mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M18 4l2 4h-3l-2-4h-2l2 4h-3l-2-4H8l2 4H7L5 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V4h-4z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M18 4l2 4h-3l-2-4h-2l2 4h-3l-2-4H8l2 4H7L5 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V4h-4z'/%3E%3C/svg%3E") center/contain no-repeat;}
.ico-series {
mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M21 3H3c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h5v2h8v-2h5c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 14H3V5h18v12z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M21 3H3c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h5v2h8v-2h5c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 14H3V5h18v12z'/%3E%3C/svg%3E") center/contain no-repeat;}
.ico-user {
mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M12 12c2.7 0 4.8-2.1 4.8-4.8S14.7 2.4 12 2.4 7.2 4.5 7.2 7.2 9.3 12 12 12zm0 2.4c-3.2 0-9.6 1.6-9.6 4.8v2.4h19.2v-2.4c0-3.2-6.4-4.8-9.6-4.8z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M12 12c2.7 0 4.8-2.1 4.8-4.8S14.7 2.4 12 2.4 7.2 4.5 7.2 7.2 9.3 12 12 12zm0 2.4c-3.2 0-9.6 1.6-9.6 4.8v2.4h19.2v-2.4c0-3.2-6.4-4.8-9.6-4.8z'/%3E%3C/svg%3E") center/contain no-repeat;}
.ico-search {
width: 16px; height: 16px; opacity: .55;mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M15.5 14h-.79l-.28-.27A6.47 6.47 0 0016 9.5 6.5 6.5 0 109.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zM9.5 14C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z'/%3E%3C/svg%3E") center/contain no-repeat;-webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M15.5 14h-.79l-.28-.27A6.47 6.47 0 0016 9.5 6.5 6.5 0 109.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zM9.5 14C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z'/%3E%3C/svg%3E") center/contain no-repeat;}
.bottom-nav button .sym {
display: block; margin: 0 auto 2px; width: 22px; height: 22px;}
.card .poster {
background-color: #141416 !important;background-size: cover !important;background-position: center !important;position: relative;}
.card .poster.no-art::after {
content: attr(data-initials);position: absolute; inset: 0;display: flex; align-items: center; justify-content: center;font-family: var(--serif, Georgia, serif);font-size: 28px; letter-spacing: .04em;color: rgba(232,184,109,.55);background: linear-gradient(160deg, #1a1a1e 0%, #0e0e10 100%);}
.card .info b {
 font-weight: 600; }
.dot {
display: inline-block; width: 6px; height: 6px; border-radius: 50%;background: var(--accent, #e8b86d); margin-right: 8px; vertical-align: middle;}
.top-avatar {
width: 38px; height: 38px; border-radius: 50%; object-fit: cover;border: 2px solid var(--line2); background: var(--panel2); cursor: pointer;transition: border-color .25s;}
.top-avatar:hover {
 border-color: var(--accent); }
.search {
display: flex; align-items: center; gap: 10px;background: var(--panel);border: 1px solid var(--line);border-radius: 14px;padding: 0 14px;transition: border-color .25s, box-shadow .25s;}
.search:focus-within {
border-color: rgba(232,168,124,.4);box-shadow: 0 0 0 3px var(--accent-dim);}
.search input {
flex: 1; border: 0; background: transparent; color: var(--text);padding: 12px 0; outline: none; font-size: 14px;}
.search input::placeholder {
 color: #71717a; }
main.view-main {
 max-width: 1100px; margin: 0 auto; padding: 20px 16px; }
.page {
 display: none; }
.page.active {
 display: block; animation: fadeUp .4s var(--ease); }
.page-title {
margin: 0 0 6px;font-family: var(--serif);font-size: 28px;font-weight: 400;letter-spacing: .02em;}
.muted {
 color: var(--muted); font-size: 12px; }
.grid {
display: grid;grid-template-columns: repeat(auto-fill, minmax(var(--card-min, 148px), 1fr));gap: 16px;margin-top: 16px;}
.card {
background: var(--panel);border: 1px solid var(--line);border-radius: var(--radius);overflow: hidden;cursor: pointer;transition: transform .35s var(--ease), border-color .3s, box-shadow .35s;animation: scaleIn .45s var(--ease) both;}
.card:nth-child(1) {
 animation-delay: .02s; }
.card:nth-child(2) {
 animation-delay: .05s; }
.card:nth-child(3) {
 animation-delay: .08s; }
.card:nth-child(4) {
 animation-delay: .11s; }
.card:nth-child(5) {
 animation-delay: .14s; }
.card:nth-child(6) {
 animation-delay: .17s; }
.card:hover {
transform: translateY(-4px);border-color: var(--line2);box-shadow: var(--shadow);}
.card:active {
 transform: scale(.98); }
.poster {
aspect-ratio: 2/3;background: var(--panel2) center/cover no-repeat;transition: transform .5s var(--ease);}
.card:hover .poster {
 transform: scale(1.03); }
.info {
 padding: 12px; }
.info b {
display: block; font-size: 13px; font-weight: 600;white-space: nowrap; overflow: hidden; text-overflow: ellipsis;}
.info small {
 color: var(--muted); font-size: 11px; }
.badge {
display: inline-block; font-size: 10px; padding: 3px 8px;border-radius: 999px; margin-top: 8px;background: var(--accent-dim); color: var(--accent);font-weight: 600; letter-spacing: .03em;}
.empty {
color: var(--muted); padding: 48px 16px; text-align: center;font-size: 14px;}
.bottom-nav {
position: fixed; left: 0; right: 0; bottom: 0; z-index: 100; pointer-events: auto;background: rgba(10,10,11,.92);backdrop-filter: blur(18px); -webkit-backdrop-filter: blur(18px);border-top: 1px solid var(--line);display: grid; grid-template-columns: repeat(4, 1fr);padding: 8px 6px calc(10px + env(safe-area-inset-bottom));}
.bottom-nav button {
border: 0; background: transparent; color: var(--muted); cursor: pointer;padding: 8px 4px; border-radius: 12px; font-size: 10px; font-weight: 600;letter-spacing: .04em; text-transform: uppercase;display: flex; flex-direction: column; align-items: center; gap: 4px;transition: color .25s, background .25s, transform .2s;}
.bottom-nav button .sym {
 font-size: 20px; transition: transform .25s var(--ease); }
.bottom-nav button.active {
color: var(--text);background: rgba(255,255,255,.06);}
.bottom-nav button.active .sym {
 transform: scale(1.08); color: var(--accent); }
.profile-box {
background: var(--panel);border: 1px solid var(--line);border-radius: 20px;padding: 24px 20px;max-width: 420px;animation: scaleIn .4s var(--ease);}
.avatar-wrap {
display: flex; gap: 18px; align-items: center; margin-bottom: 20px;padding-bottom: 18px; border-bottom: 1px solid var(--line);}
.avatar {
width: 96px; height: 96px; min-width: 96px; border-radius: 50%;object-fit: cover; background: var(--panel2);border: 3px solid var(--line2);}
.file-btn {
display: inline-block; position: relative; overflow: hidden;border-radius: 12px; background: var(--panel2); color: #fff;font-weight: 600; font-size: 13px; padding: 10px 14px; cursor: pointer;border: 1px solid var(--line); transition: border-color .25s;}
.file-btn:hover {
 border-color: var(--accent); }
.file-btn input {
 position: absolute; inset: 0; opacity: 0; cursor: pointer; }
.field {
 margin: 0 0 16px; }
.field label {
display: block; color: var(--muted); font-size: 12px;margin-bottom: 8px; font-weight: 600; letter-spacing: .03em;}
.btn {
border: 0; border-radius: 14px; padding: 14px 16px; cursor: pointer;background: var(--accent); color: #1a120a; font-weight: 700;width: 100%; margin-top: 10px; font-size: 15px;transition: transform .2s var(--ease), filter .2s, box-shadow .25s;}
.btn:hover {
 filter: brightness(1.05); box-shadow: 0 8px 24px rgba(232,168,124,.25); }
.btn:active {
 transform: scale(.98); }
.btn.secondary {
 background: var(--panel2); color: var(--text); }
.btn.secondary:hover {
 box-shadow: none; filter: brightness(1.1); }
.btn.block {
 width: 100%; }
.btn.small {
 width: auto; padding: 10px 14px; margin: 0; font-size: 13px; }
.btn.ok {
 background: var(--ok); color: #0a1a10; }
.btn.danger {
 background: var(--danger); color: #fff; }
input[type="text"], input[type="password"], input[type="datetime-local"], textarea, select {
width: 100%; background: var(--bg2); color: var(--text);border: 1px solid var(--line); border-radius: 12px;padding: 12px 14px; outline: none;transition: border-color .25s, box-shadow .25s;}
input:focus, textarea:focus, select:focus {
border-color: rgba(232,168,124,.45);box-shadow: 0 0 0 3px var(--accent-dim);}
textarea {
min-height: 90px; resize: vertical;-webkit-user-select: text; user-select: text;}
textarea#msgText, #msgText {
-webkit-user-select: text !important;user-select: text !important;pointer-events: auto !important;}
#crownGate {
display: none; position: fixed; inset: 0; z-index: 400;background: rgba(0,0,0,.65);align-items: center; justify-content: center; padding: 16px;backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);animation: fadeIn .25s ease;}
#crownGate.open {
 display: flex; }
.crown-box {
width: min(360px, 100%);background: var(--panel);border: 1px solid rgba(232,168,124,.25);border-radius: 20px; padding: 22px;box-shadow: var(--shadow);animation: scaleIn .35s var(--ease);}
.crown-box h3 {
margin: 0 0 8px; font-size: 18px; font-family: var(--serif);font-weight: 400; color: var(--accent);}
.crown-box .err {
 color: var(--danger); font-size: 12px; min-height: 16px; margin-top: 8px; }
.panel {
display: none; position: fixed; inset: 0; z-index: 60;background: var(--bg); flex-direction: column;}
.panel.open {
 display: flex; animation: fadeUp .35s var(--ease); }
.panel-head {
display: flex; align-items: center; gap: 12px;padding: 14px 16px; border-bottom: 1px solid var(--line);background: rgba(10,10,11,.9);backdrop-filter: blur(12px);}
.panel-body {
 flex: 1; overflow: auto; padding: 16px; }
.icon-btn {
border: 0; border-radius: 10px; padding: 8px 12px; cursor: pointer;background: var(--panel2); color: var(--text); font-weight: 600;transition: background .2s;}
.icon-btn:hover {
 background: #252528; }
.hist-card {
display: flex; gap: 12px; align-items: center;background: var(--panel); border: 1px solid var(--line);border-radius: 14px; padding: 12px; margin-bottom: 10px;transition: border-color .25s, transform .25s var(--ease);animation: fadeUp .35s var(--ease) both;}
.hist-card:hover {
 border-color: var(--line2); }
.hist-card img {
width: 56px; height: 80px; object-fit: cover;border-radius: 8px; background: #222;}
.hist-card .meta {
 flex: 1; min-width: 0; }
.hist-card .meta b {
 display: block; font-size: 14px; }
.hist-card .meta small {
 color: var(--muted); display: block; margin-top: 4px; }
.overlay {
position: fixed; inset: 0; background: rgba(0,0,0,.9);display: none; z-index: 70; padding: 12px;align-items: center; justify-content: center;animation: fadeIn .25s ease;}
.overlay.open {
 display: flex; }
.player-box {
width: min(1100px, 100%);background: var(--bg2);border: 1px solid var(--line);border-radius: 18px; overflow: hidden;box-shadow: var(--shadow);animation: scaleIn .35s var(--ease);}
.player-top {
display: flex; justify-content: space-between; gap: 10px; align-items: center;padding: 14px 16px; border-bottom: 1px solid var(--line);}
.player-top h3 {
 margin: 0; font-size: 16px; font-weight: 600; }
.stage {
 position: relative; background: #000; aspect-ratio: 16/9; }
.stage iframe, .stage video {
position: absolute; inset: 0; width: 100%; height: 100%; border: 0;}
.ctrl {
display: flex; flex-wrap: wrap; gap: 8px; padding: 12px;border-top: 1px solid var(--line); align-items: center;}
.ctrl button {
border: 0; border-radius: 10px; padding: 8px 12px; cursor: pointer;background: var(--panel2); color: var(--text); font-weight: 600;transition: background .2s;}
.ctrl button:hover {
 background: #2a2a2e; }
.ep-list {
max-height: 160px; overflow: auto; border-top: 1px solid var(--line);padding: 10px; display: none;}
.ep-list.show {
 display: block; }
.ep {
padding: 10px 12px; border-radius: 8px; cursor: pointer;border: 1px solid transparent; transition: background .2s, border-color .2s;}
.ep:hover {
 background: rgba(255,255,255,.04); }
.ep.active {
border-color: rgba(232,168,124,.4);background: var(--accent-dim);}
#adminApp {
display: none !important;min-height: 100vh;background:radial-gradient(900px 500px at 0% -10%, rgba(232,168,124,.06), transparent),var(--bg);}
#adminApp.on {
display: block !important;animation: fadeUp .4s var(--ease);}
.admin-header {
display: flex; align-items: center; justify-content: space-between; gap: 12px;padding: 16px 22px; border-bottom: 1px solid var(--line);background: rgba(10,10,11,.9);backdrop-filter: blur(14px);position: sticky; top: 0; z-index: 10;}
.wrap {
max-width: 1200px; margin: 0 auto; padding: 20px;display: grid; grid-template-columns: 320px 1fr; gap: 18px;}
@media (max-width: 900px) {
 .wrap {
 grid-template-columns: 1fr; }
 }
.card2 {
background: var(--panel);border: 1px solid var(--line);border-radius: 18px; padding: 18px;}
.list {
display: flex; flex-direction: column; gap: 8px;max-height: 70vh; overflow: auto;}
.item {
display: flex; gap: 10px; align-items: center; padding: 10px;border: 1px solid var(--line); border-radius: 12px;background: var(--bg2); cursor: pointer;transition: border-color .25s, transform .2s;}
.item:hover {
 border-color: var(--line2); }
.item.active {
 border-color: rgba(232,168,124,.5); box-shadow: 0 0 0 1px var(--accent-dim); }
.item img {
width: 48px; height: 68px; object-fit: cover;border-radius: 6px; background: #222;}
.item .meta {
 flex: 1; min-width: 0; }
.item .meta b {
display: block; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;}
.badge2 {
font-size: 10px; padding: 3px 7px; border-radius: 999px; font-weight: 700;background: var(--panel2); color: var(--muted);}
.badge2.on {
 background: rgba(127,217,154,.15); color: var(--ok); }
.banner-preview {
width: 100%; max-height: 200px; object-fit: cover; border-radius: 12px;background: var(--bg2); border: 1px solid var(--line); margin-top: 8px; min-height: 100px;}
.row {
 display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.btns {
 display: flex; flex-wrap: wrap; gap: 8px; margin-top: 12px; }
.ep-box {
border: 1px dashed var(--line); border-radius: 12px;padding: 12px; margin-top: 10px;}
.toast {
position: fixed; right: 16px; bottom: 90px;background: var(--panel); border: 1px solid var(--line);padding: 12px 16px; border-radius: 12px;opacity: 0; transform: translateY(10px);transition: .3s var(--ease); z-index: 500;pointer-events: none !important;box-shadow: var(--shadow); font-size: 13px; font-weight: 500;}
.toast.show {
 opacity: 1; transform: none; }
#updateBanner {
display: none; position: fixed; left: 12px; right: 12px; top: 12px; z-index: 95;pointer-events: none;background: var(--panel); border: 1px solid rgba(232,168,124,.35);border-radius: 14px; padding: 12px 14px;box-shadow: var(--shadow);align-items: center; justify-content: space-between; gap: 10px;animation: fadeUp .4s var(--ease);}
#updateBanner.show {
 display: flex; pointer-events: auto; }
#updateBanner b {
 font-size: 13px; }
#updateBanner .muted {
 margin: 2px 0 0; }
label.lbl {
display: block; font-size: 12px; color: var(--muted);margin: 10px 0 6px; font-weight: 600;}
.genre-fab {
width: 38px; height: 38px; border-radius: 50%;border: 1px solid var(--line2);background: var(--panel2);color: var(--text);font-size: 16px; cursor: pointer;display: flex; align-items: center; justify-content: center;}
.drawer-backdrop {
position: fixed; inset: 0; z-index: 220;background: rgba(0,0,0,.55);opacity: 0; visibility: hidden; pointer-events: none;transition: opacity .3s var(--ease), visibility .3s;}
.drawer-backdrop.open {
 opacity: 1; visibility: visible; pointer-events: auto; }
.genre-drawer {
position: fixed; top: 0; left: 0; bottom: 0; z-index: 230;width: min(300px, 86vw);background: var(--panel);border-right: 1px solid var(--line);transform: translateX(-105%);transition: transform .35s var(--ease);display: flex; flex-direction: column;box-shadow: 8px 0 40px rgba(0,0,0,.4);pointer-events: none;}
.genre-drawer.open {
 transform: translateX(0); pointer-events: auto; }
body.drawer-open .bottom-nav {
 pointer-events: none !important; opacity: .4; }
body.drawer-open #viewApp {
 pointer-events: none; }
body.drawer-open .genre-drawer,body.drawer-open .drawer-backdrop {
 pointer-events: auto !important; }
.drawer-head {
display: flex; align-items: center; justify-content: space-between;padding: 16px 14px; border-bottom: 1px solid var(--line);}
.drawer-head b {
 font-size: 15px; }
.drawer-body {
 flex: 1; overflow-y: auto; padding: 10px 0; }
.drawer-item {
display: flex; align-items: center; gap: 10px;width: 100%; border: 0; background: transparent;color: var(--text); text-align: left;padding: 12px 18px; font-size: 14px; cursor: pointer;transition: background .2s;}
.drawer-item:hover, .drawer-item.active {
background: var(--accent-dim); color: var(--accent);}
.drawer-item.active {
 font-weight: 700; }
.drawer-foot {
padding-bottom: calc(16px + env(safe-area-inset-bottom));border-top: 1px solid var(--line);padding: 10px 0 calc(12px + env(safe-area-inset-bottom));}
.drawer-settings {
padding: 12px 16px 4px;}
.drawer-settings .lbl {
 margin-top: 10px; }
.drawer-credits {
margin-top: 14px; padding-top: 12px; border-top: 1px solid var(--line);font-size: 12px; color: var(--muted); line-height: 1.5;}
.drawer-credits b {
 color: var(--text); }
.genre-chip-row {
display: flex; flex-wrap: wrap; gap: 8px; margin: 8px 0 4px;}
.genre-chip {
display: inline-flex; align-items: center; gap: 4px;padding: 6px 10px; border-radius: 999px;border: 1px solid var(--line2); background: var(--panel2);font-size: 12px; cursor: pointer; color: var(--muted);user-select: none;}
.genre-chip.on {
background: var(--accent-dim); color: var(--accent);border-color: rgba(232,168,124,.45); font-weight: 600;}
.genre-filter-bar {
display: flex; align-items: center; gap: 8px;margin: 8px 0 0; flex-wrap: wrap;}
.genre-filter-bar .chip-active {
font-size: 12px; color: var(--accent); font-weight: 600;}
#copyModal {
display: none; position: fixed; inset: 0; z-index: 200;background: rgba(0,0,0,.72); align-items: center; justify-content: center;padding: 16px; pointer-events: auto;}
#copyModal.open {
 display: flex; }
#copyModal .box {
width: min(560px, 100%); max-height: 88vh;background: var(--panel); border: 1px solid var(--line2);border-radius: 16px; padding: 16px; display: flex; flex-direction: column; gap: 10px;box-shadow: var(--shadow);}
#copyModal textarea {
width: 100%; min-height: 220px; max-height: 50vh; resize: vertical;background: var(--bg2); color: var(--text); border: 1px solid var(--line);border-radius: 12px; padding: 12px; font-size: 11px; font-family: ui-monospace, monospace;-webkit-user-select: text; user-select: text;}
.obliry-splash{
position:fixed; inset:0; z-index:2147483000;background:#000;display:flex; align-items:center; justify-content:center;overflow:hidden;transition: opacity .95s ease, visibility .95s ease;}
.obliry-splash.hide{
 opacity:0; visibility:hidden; pointer-events:none; }
.obliry-splash.gone{
 display:none !important; }
.ob-stage{
position:relative; width:100%; height:100%;display:flex; flex-direction:column; align-items:center; justify-content:center;transform-origin: 18% 48%; will-change: transform;}
.ob-word{
position:relative;font-family: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;font-weight: 800;font-size: clamp(44px, 15vw, 96px);letter-spacing: 0.22em;color: #f3f3f3;line-height: 1;user-select: none;transform: scale(0.78);opacity: 0;filter: blur(10px);}
.ob-letter{
 display:inline-block; opacity:0; transform: translateY(22px); }
.ob-hole{
position:absolute;left: 0;top: 50%;width: 0.78em; height: 0.78em;margin-left: -0.02em;transform: translateY(-50%) scale(0.12);border-radius: 50%;pointer-events: none;opacity: 0;background:radial-gradient(circle at 50% 42%,rgba(30,30,32,0.2) 0%,rgba(0,0,0,0.65) 42%,#000 100%);box-shadow:0 0 0 1px rgba(232,184,109,0.4),0 0 28px 6px rgba(232,184,109,0.22),inset 0 0 24px rgba(0,0,0,0.9);will-change: transform, opacity;}
.ob-credit{
margin-top: 32px;font-size: 12.5px;letter-spacing: 0.14em;color: #9a9a9a;opacity: 0;transform: translateY(12px);font-weight: 500;text-transform: none;}
.ob-vignette{
position:absolute; inset:0; pointer-events:none;background: radial-gradient(ellipse at 18% 48%, transparent 20%, rgba(0,0,0,0.55) 55%, rgba(0,0,0,0.92) 100%);opacity: 1;transition: opacity 1.4s ease;}
.obliry-splash.phase-in .ob-word{
opacity: 1; filter: blur(0); transform: scale(1);transition: opacity 1.25s ease, filter 1.35s ease, transform 1.55s cubic-bezier(.2,.75,.2,1);}
.obliry-splash.phase-in .ob-letter{
opacity: 1; transform: translateY(0);transition: opacity .75s ease, transform .9s cubic-bezier(.2,.75,.2,1);}
.obliry-splash.phase-in .ob-letter:nth-child(1){
 transition-delay: .18s; }
.obliry-splash.phase-in .ob-letter:nth-child(2){
 transition-delay: .32s; }
.obliry-splash.phase-in .ob-letter:nth-child(3){
 transition-delay: .46s; }
.obliry-splash.phase-in .ob-letter:nth-child(4){
 transition-delay: .60s; }
.obliry-splash.phase-in .ob-letter:nth-child(5){
 transition-delay: .74s; }
.obliry-splash.phase-in .ob-letter:nth-child(6){
 transition-delay: .88s; }
.obliry-splash.phase-in .ob-credit{
opacity: 1; transform: translateY(0);transition: opacity 1.1s ease 1.05s, transform 1.1s ease 1.05s;}
.obliry-splash.phase-hole .ob-hole{
opacity: 1;transform: translateY(-50%) scale(0.95);transition: opacity .9s ease, transform 1.6s cubic-bezier(.15,.7,.1,1);}
.obliry-splash.phase-dive .ob-stage{
animation: obDive 2.85s cubic-bezier(.1,.55,.1,1) forwards;}
.obliry-splash.phase-dive .ob-hole{
transform: translateY(-50%) scale(22);opacity: 1;transition: transform 2.85s cubic-bezier(.1,.55,.1,1), opacity 1.4s ease 1.3s;}
.obliry-splash.phase-dive .ob-letter{
opacity: 0;transition: opacity 1s ease .25s;}
.obliry-splash.phase-dive .ob-credit{
opacity: 0;transition: opacity .8s ease;}
.obliry-splash.phase-dive .ob-vignette{
opacity: 0;transition: opacity 1.6s ease .35s;}
@keyframes obDive{
0% {
 transform: scale(1) translate(0,0); }
35% {
 transform: scale(1.45) translate(-6%, 1%); }
100% {
 transform: scale(3.2) translate(-14%, 2%); }
}
#appBg {
position: fixed; inset: 0; z-index: 0;pointer-events: none !important;background-color: #0a0a0b;background-size: cover;background-position: center;background-repeat: no-repeat;}
#viewApp, #adminApp {
 position: relative; z-index: 1; }
</style>

<style id="homeHeroCss">
.home-hero-img {
position: absolute; inset: 0; width: 100%; height: 100%;object-fit: cover; display: block; pointer-events: none;border-radius: inherit; z-index: 0;}
.home-hero-slide {
 position: relative; overflow: hidden; }
.home-hero-slide .cap {
 position: relative; z-index: 1; }
.card, .home-hero-slide, .bottom-nav button, .btn, .icon-btn, .genre-fab {
-webkit-tap-highlight-color: rgba(232,168,124,.25);touch-action: manipulation;cursor: pointer;}
.home-hero {
position: relative;width: 100%;height: 42vw;min-height: 140px;max-height: 220px;border-radius: 14px;overflow: hidden;background: #14141a;margin: 0 0 14px;border: 1px solid #22222c;box-shadow: 0 8px 24px rgba(0,0,0,.35);}
.home-hero-track {
display: flex;height: 100%;transition: transform .45s ease;}
.home-hero-slide {
min-width: 100%;height: 100%;background: #1a1a22 center/cover no-repeat;position: relative;cursor: pointer;}
.home-hero-slide .cap {
position: absolute; left: 0; right: 0; bottom: 0;padding: 16px 14px 12px;background: linear-gradient(transparent, rgba(0,0,0,.9));font-size: 15px; font-weight: 700;text-shadow: 0 1px 3px #000;}
.home-hero-dots {
position: absolute; bottom: 10px; right: 12px;display: flex; gap: 6px; z-index: 2;}
.home-hero-dots span {
width: 7px; height: 7px; border-radius: 50%;background: rgba(255,255,255,.35);transition: background .2s;}
.home-hero-dots span.on {
 background: #e8b86d; }
.home-hero-nav {
position: absolute; top: 50%; transform: translateY(-50%);width: 32px; height: 32px; border-radius: 50%;border: 0; background: rgba(0,0,0,.45); color: #fff;font-size: 18px; cursor: pointer; z-index: 3;display: flex; align-items: center; justify-content: center;backdrop-filter: blur(4px);}
.home-hero-nav:active {
 background: rgba(0,0,0,.7); }
.home-hero-nav.prev {
 left: 8px; }
.home-hero-nav.next {
 right: 8px; }
.home-hero-empty {
display: flex; align-items: center; justify-content: center;height: 100%; color: #666; font-size: 13px;}
</style>

<style id="cineTungAdaptCss">
/* cobre marca d'água "Built with HTML to APK Builder" */
#apkWatermarkMask{
  position:fixed!important;left:0!important;right:0!important;bottom:0!important;
  height:28px!important;z-index:2147483645!important;
  background:#0a0a0b!important;pointer-events:none!important;
}
.bottom-nav{z-index:2147483646!important}
.drawer-credits{position:relative;z-index:5;padding-bottom:28px}

/* ===== LANDSCAPE: nav lateral + topo bem compacto ===== */
@media (orientation: landscape){
  #viewApp.on{
    padding-bottom:0!important;
    padding-left:48px!important;
  }
  .bottom-nav{
    top:0!important;bottom:0!important;left:0!important;right:auto!important;
    width:48px!important;height:100%!important;
    grid-template-columns:1fr!important;
    grid-template-rows:repeat(4,minmax(0,1fr))!important;
    border-top:0!important;
    border-right:1px solid var(--line)!important;
    padding:4px 2px!important;
    gap:0!important;
  }
  .bottom-nav button{
    padding:4px 0!important;
    font-size:0!important;
    letter-spacing:0!important;
    gap:0!important;
    min-height:0!important;
  }
  .bottom-nav button .nav-ico,
  .bottom-nav button svg,
  .bottom-nav button span{
    font-size:16px!important;
    width:18px!important;height:18px!important;
  }
  .top{
    padding:4px 8px 4px!important;
  }
  .top-row{
    margin-bottom:4px!important;
    gap:6px!important;
  }
  .brand{font-size:14px!important}
  .top-avatar{width:28px!important;height:28px!important}
  .icon-btn,.genre-fab{
    width:30px!important;height:30px!important;
    min-width:30px!important;
  }
  .search{
    padding:4px 8px!important;
    min-height:0!important;
  }
  .search input{
    font-size:12px!important;
    padding:4px 0!important;
  }
  .view-main{padding:6px 10px!important}
  .page-title{font-size:16px!important;margin:0 0 2px!important}
  .page > .muted{display:none!important}
  .grid{gap:10px!important;margin-top:8px!important}
  #apkWatermarkMask{
    left:0!important;width:48px!important;right:auto!important;
    height:auto!important;bottom:0!important;top:auto!important;
    background:#0a0a0b!important;
  }
}
</style>
</head>
<body style="background-color:#0a0a0b;" class="">
<div id="appBg" aria-hidden="true" style="background-color: rgb(10, 10, 11); background-image: linear-gradient(rgba(8, 8, 10, 0.75) 0%, rgba(8, 8, 10, 0.5) 45%, rgba(8, 8, 10, 0.85) 100%), url(); background-size: cover, cover; background-position: center center, center center; background-repeat: no-repeat, no-repeat; pointer-events: none;"></div>

<!-- OBLIRY INTRO -->
<style id="obliryIntroCss">
#splash.obliry-splash{
position:fixed!important; inset:0!important; z-index:2147483646!important;background:#000!important; margin:0!important; padding:0!important;display:flex!important; align-items:center!important; justify-content:center!important;overflow:hidden!important;transition:opacity .55s ease, visibility .55s ease;pointer-events:auto!important;touch-action:none!important;}
#splash.obliry-splash.hide,#splash.obliry-splash.gone{
opacity:0!important; visibility:hidden!important; pointer-events:none!important;display:none!important; z-index:-1!important;}
#splash.obliry-splash.hide{
 display:none!important; }
#splash.obliry-splash.gone{
 display:none!important; }
#splash .ob-stage{
position:relative; width:100%; height:100%;display:flex; flex-direction:column; align-items:center; justify-content:center;transform-origin: 16% 50%;will-change: transform;}
#splash .ob-word{
position:relative;font-family: system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;font-weight:800;font-size: clamp(48px, 16vw, 100px);letter-spacing: 0.2em;color:#f5f5f5;line-height:1;opacity:0;transform: scale(0.85);filter: blur(12px);}
#splash .ob-letter{
display:inline-block;opacity:0;transform: translateY(24px);}
#splash .ob-hole{
position:absolute;left:0; top:50%;width:0.82em; height:0.82em;margin-left:-0.04em;transform: translateY(-50%) scale(0.08);border-radius:50%;opacity:0;pointer-events:none;background: radial-gradient(circle at 50% 40%, rgba(40,40,42,.25) 0%, rgba(0,0,0,.7) 40%, #000 100%);box-shadow: 0 0 0 2px rgba(232,184,109,.45), 0 0 30px 8px rgba(232,184,109,.2), inset 0 0 30px #000;}
#splash .ob-credit{
margin-top:36px;font-size:13px;letter-spacing:0.16em;color:#a0a0a0;opacity:0;transform: translateY(14px);font-weight:500;}
#splash .ob-vignette{
position:absolute; inset:0; pointer-events:none;background: radial-gradient(ellipse at 16% 50%, transparent 18%, rgba(0,0,0,.5) 50%, rgba(0,0,0,.95) 100%);}
#splash.phase-in .ob-word{
opacity:1; filter:blur(0); transform:scale(1);transition: opacity 1.3s ease, filter 1.4s ease, transform 1.6s cubic-bezier(.2,.75,.2,1);}
#splash.phase-in .ob-letter{
opacity:1; transform:translateY(0);transition: opacity .8s ease, transform .95s cubic-bezier(.2,.75,.2,1);}
#splash.phase-in .ob-letter:nth-child(1){
 transition-delay:.2s; }
#splash.phase-in .ob-letter:nth-child(2){
 transition-delay:.35s; }
#splash.phase-in .ob-letter:nth-child(3){
 transition-delay:.5s; }
#splash.phase-in .ob-letter:nth-child(4){
 transition-delay:.65s; }
#splash.phase-in .ob-letter:nth-child(5){
 transition-delay:.8s; }
#splash.phase-in .ob-letter:nth-child(6){
 transition-delay:.95s; }
#splash.phase-in .ob-credit{
opacity:1; transform:translateY(0);transition: opacity 1.2s ease 1.15s, transform 1.2s ease 1.15s;}
#splash.phase-hole .ob-hole{
opacity:1;transform: translateY(-50%) scale(1);transition: opacity 1s ease, transform 1.7s cubic-bezier(.15,.7,.1,1);}
#splash.phase-dive .ob-stage{
animation: obCamDive 1.55s cubic-bezier(.12,.6,.12,1) forwards;}
#splash.phase-dive .ob-hole{
transform: translateY(-50%) scale(24);transition: transform 1.55s cubic-bezier(.12,.6,.12,1), opacity .7s ease .5s;}
#splash.phase-dive .ob-letter{
 opacity:0; transition: opacity .45s ease .05s; }
#splash.phase-dive .ob-credit{
 opacity:0; transition: opacity .4s ease; }
#splash.phase-dive .ob-vignette{
 opacity:0; transition: opacity .7s ease .15s; }
@keyframes obCamDive{
0%{
 transform: scale(1) translate(0,0); }
30%{
 transform: scale(1.5) translate(-5%,0); }
100%{
 transform: scale(3.4) translate(-12%,1%); }
}
</style>

<div id="splash" class="obliry-splash">
  <div class="ob-stage">
    <div class="ob-word">
      <span class="ob-letter">O</span><span class="ob-letter">B</span><span class="ob-letter">L</span><span class="ob-letter">I</span><span class="ob-letter">R</span><span class="ob-letter">Y</span>
      <div class="ob-hole"></div>
    </div>
    <div class="ob-credit">Feito por Raphael Rocha</div>
  </div>
  <div class="ob-vignette"></div>
</div>

<script>
(function () {
  var splash = document.getElementById("splash");
  if (!splash) return;
  var done = false;
  try { window.__obliryIntroBlocking = true; window.__obliryIntroDone = false; } catch (e) {}
  function finish() {
    if (done) return;
    done = true;
    try {
      window.__obliryIntroDone = true;
      window.__obliryIntroBlocking = false;
    } catch (e) {}
    try {
      splash.style.pointerEvents = "none";
      splash.classList.add("hide");
      splash.style.display = "none";
    } catch (e) {}
    setTimeout(function () {
      try {
        splash.classList.add("gone");
        if (splash.parentNode) splash.parentNode.removeChild(splash);
      } catch (e) {}
    }, 560);
  }
  // timeline da intro
  setTimeout(function () { try { splash.classList.add("phase-in"); } catch (e) {} }, 180);
  setTimeout(function () { try { splash.classList.add("phase-hole"); } catch (e) {} }, 1750);
  setTimeout(function () { try { splash.classList.add("phase-dive"); } catch (e) {} }, 2350);
  setTimeout(finish, 4000);
  // pular intro: bloqueia toque de passar para o filme por baixo
  function skip(ev) {
    try {
      if (ev) {
        if (ev.preventDefault) ev.preventDefault();
        if (ev.stopPropagation) ev.stopPropagation();
        if (ev.stopImmediatePropagation) ev.stopImmediatePropagation();
      }
    } catch (e) {}
    // trava imediatamente qualquer clique nos cards
    try {
      splash.style.pointerEvents = "none";
      window.__obliryIntroBlocking = true;
    } catch (e) {}
    try { splash.classList.add("phase-hole"); } catch (e) {}
    try { splash.classList.add("phase-dive"); } catch (e) {}
    setTimeout(finish, 650);
  }
  splash.addEventListener("click", skip, true);
  splash.addEventListener("touchstart", function (ev) {
    try {
      if (ev.preventDefault) ev.preventDefault();
      if (ev.stopPropagation) ev.stopPropagation();
      if (ev.stopImmediatePropagation) ev.stopImmediatePropagation();
    } catch (e) {}
  }, { capture: true, passive: false });
  splash.addEventListener("touchend", skip, { capture: true, passive: false });
  window.__obliryEndSplash = finish;
})();
</script>

<script>
/* CineTung emergency boot — só libera bloqueios; NÃO fecha o player */
(function () {
  var forceLibDone = false;
  function unlockBlockers() {
    try {
      document.body.classList.remove("drawer-open");
      document.body.style.pointerEvents = "auto";
      var splash = document.getElementById("splash");
      if (splash) {
        splash.style.pointerEvents = "none";
        if (window.__obliryIntroDone) {
          try { splash.style.display = "none"; } catch (e) {}
        }
      }
      var db = document.getElementById("drawerBackdrop");
      if (db && !db.classList.contains("open")) {
        db.style.pointerEvents = "none";
        db.style.opacity = "0";
        db.style.visibility = "hidden";
      }
      var gd = document.getElementById("genreDrawer");
      if (gd && !gd.classList.contains("open")) {
        gd.style.pointerEvents = "none";
      }
      var nav = document.getElementById("bottomNav");
      if (nav) {
        nav.style.pointerEvents = "auto";
        nav.style.opacity = "1";
        nav.style.zIndex = "100";
      }
      var view = document.getElementById("viewApp");
      if (view) {
        view.style.pointerEvents = "auto";
        view.style.visibility = "visible";
        view.style.opacity = "1";
      }
    } catch (e) {}
  }
  function forceLibOnce() {
    if (forceLibDone) return;
    forceLibDone = true;
    try {
      var lib = document.getElementById("page-lib");
      if (!lib) return;
      var libOn = lib.classList.contains("active");
      var movies = document.getElementById("page-movies");
      var series = document.getElementById("page-series");
      var otherOn = (movies && movies.classList.contains("active")) || (series && series.classList.contains("active"));
      if (!libOn && !otherOn) {
        ["lib", "movies", "series", "profile"].forEach(function (id) {
          var el = document.getElementById("page-" + id);
          if (!el) return;
          if (id === "lib") el.classList.add("active");
          else el.classList.remove("active");
        });
        var nav = document.getElementById("bottomNav");
        if (nav) {
          nav.querySelectorAll("button[data-tab]").forEach(function (b) {
            if (b.getAttribute("data-tab") === "lib") b.classList.add("active");
            else b.classList.remove("active");
          });
        }
        var sb = document.getElementById("searchBar");
        if (sb) sb.style.display = "flex";
      }
    } catch (e) {}
  }
  function bindNavFallback() {
    var nav = document.getElementById("bottomNav");
    if (!nav || nav.__cineBound) return;
    nav.__cineBound = true;
    function go(name) {
      if (!name) return;
      if (typeof window.setTab === "function") {
        try { window.setTab(name); return; } catch (e) {}
      }
      ["lib", "movies", "series", "profile"].forEach(function (id) {
        var el = document.getElementById("page-" + id);
        if (!el) return;
        if (id === name) el.classList.add("active");
        else el.classList.remove("active");
      });
      nav.querySelectorAll("button[data-tab]").forEach(function (b) {
        if (b.getAttribute("data-tab") === name) b.classList.add("active");
        else b.classList.remove("active");
      });
      var sb = document.getElementById("searchBar");
      if (sb) sb.style.display = name === "profile" ? "none" : "flex";
    }
    nav.addEventListener("click", function (ev) {
      var t = ev.target;
      while (t && t !== nav && !(t.getAttribute && t.getAttribute("data-tab"))) t = t.parentNode;
      if (!t || !t.getAttribute) return;
      var name = t.getAttribute("data-tab");
      if (!name) return;
      try { ev.preventDefault(); } catch (e) {}
      go(name);
    }, true);
    nav.addEventListener("touchend", function (ev) {
      var t = ev.target;
      while (t && t !== nav && !(t.getAttribute && t.getAttribute("data-tab"))) t = t.parentNode;
      if (!t || !t.getAttribute) return;
      var name = t.getAttribute("data-tab");
      if (!name) return;
      try { ev.preventDefault(); } catch (e) {}
      go(name);
    }, { passive: false, capture: true });
  }
  function boot() {
    unlockBlockers();
    forceLibOnce();
    bindNavFallback();
  }
  boot();
  setTimeout(boot, 400);
  setTimeout(function () { unlockBlockers(); bindNavFallback(); }, 1500);
  setTimeout(function () { unlockBlockers(); }, 4000);
  document.addEventListener("DOMContentLoaded", boot);
  window.addEventListener("load", function () { unlockBlockers(); bindNavFallback(); });
  window.addEventListener("pageshow", function () { unlockBlockers(); });
})();
</script>

<!-- ========== VIEWER ========== -->
<div id="viewApp" class="on">
  <div class="top">
    <div class="top-row">
      <div class="brand">Cine <span>Tung</span></div>
      <div class="top-right">
        <button type="button" class="genre-fab" id="btnGenreDrawer" title="Gêneros">☰</button>
        <button type="button" class="icon-btn" id="crownBtn" title="Área restrita"><span class="ico-crown"></span></button>
        <img class="top-avatar" id="topAvatar" alt="" title="Perfil" src="">
      </div>
    </div>
    <div class="search" id="searchBar" style="display: flex;">
      <span class="ico-search"></span>
      <input id="q" placeholder="Pesquisar por nome...">
    </div>
  </div>

  <main class="view-main">
    <section class="page active" id="page-lib">
      <h1 class="page-title">Biblioteca</h1>
      <p class="muted" id="syncInfo">11 título(s) · 22/11/2026, 16:40:01</p>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelLib">Gênero: Todos</span></div>
      <div class="home-hero" id="homeHero" aria-label="Destaques">
        <div class="home-hero-track" id="homeHeroTrack"></div>
        <div class="home-hero-dots" id="homeHeroDots"></div>
        <button type="button" class="home-hero-nav prev" id="homeHeroPrev" aria-label="Anterior">‹</button>
        <button type="button" class="home-hero-nav next" id="homeHeroNext" aria-label="Próximo">›</button>
      </div>
      <div class="grid" id="grid-lib"><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>A Noiva Cadáver</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Os Croods</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Angry Birds Movie</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Verity</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-lib" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-movies">
      <h1 class="page-title">Filmes</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelMovies">Gênero: Todos</span></div>
      <div class="grid" id="grid-movies"><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>A Noiva Cadáver</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Os Croods</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Angry Birds Movie</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Verity</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-movies" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-series">
      <h1 class="page-title">Séries</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelSeries">Gênero: Todos</span></div>
      <div class="grid" id="grid-series"><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url("")"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div></div>
      <div class="empty" id="empty-series" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-profile">
      <h1 class="page-title">Perfil</h1>
      <div class="profile-box">
        <div class="avatar-wrap">
          <img class="avatar" id="avatarImg" alt="" src="">
          <div>
            <div class="muted" style="margin-bottom:10px">Trocar foto da galeria</div>
            <label class="file-btn">Escolher foto<input type="file" id="avatarFile" accept="image/*"></label>
          </div>
        </div>
        <div class="field">
          <label>Seu nome</label>
          <input type="text" id="profileName" placeholder="Como quer ser chamado">
        </div>
        <button class="btn block" type="button" id="btnSaveProfile">Salvar perfil</button>
        <button class="btn secondary block" type="button" id="btnHistory">⏱️ Histórico / Continuar</button>
        <button class="btn secondary block" type="button" id="btnMsg">✉️ Falar com o dono</button>

      </div>
    </section>
  <div id="apkWatermarkMask" aria-hidden="true"></div>
</main>

<nav class="bottom-nav" id="bottomNav" style="pointer-events: auto;">
    <button type="button" class="active" data-tab="lib"><span class="sym ico-lib"></span>Biblioteca</button>
    <button type="button" data-tab="movies"><span class="sym ico-movie"></span>Filmes</button>
    <button type="button" data-tab="series"><span class="sym ico-series"></span>Séries</button>
    <button type="button" data-tab="profile"><span class="sym ico-user"></span>Perfil</button>
  </nav>
</div>

<!-- genre drawer -->
<div class="drawer-backdrop" id="drawerBackdrop" style="pointer-events: none; opacity: 0; visibility: hidden;"></div>
<aside class="genre-drawer" id="genreDrawer" aria-hidden="true" style="">
  <div class="drawer-head">
    <b>Gêneros</b>
    <button type="button" class="icon-btn" id="drawerClose" style="width:36px;height:36px;border-radius:50%;border:1px solid var(--line);background:var(--panel2);color:var(--text);cursor:pointer">✕</button>
  </div>
  <div class="drawer-body" id="drawerGenreList"><button type="button" class="drawer-item active"><span>•</span> Todos</button><button type="button" class="drawer-item"><span>·</span> Ação</button><button type="button" class="drawer-item"><span>·</span> Aventura</button><button type="button" class="drawer-item"><span>·</span> Comédia</button><button type="button" class="drawer-item"><span>·</span> Drama</button><button type="button" class="drawer-item"><span>·</span> Terror</button><button type="button" class="drawer-item"><span>·</span> Romance</button><button type="button" class="drawer-item"><span>·</span> Ficção</button><button type="button" class="drawer-item"><span>·</span> Animação</button><button type="button" class="drawer-item"><span>·</span> Documentário</button><button type="button" class="drawer-item"><span>·</span> Suspense</button><button type="button" class="drawer-item"><span>·</span> Fantasia</button></div>
  <div class="drawer-foot">

        <button class="btn secondary block" type="button" id="btnDrawerHome"><span class="dot"></span> Início (Biblioteca)</button>
    <button type="button" class="drawer-item" id="btnDrawerSettings"><span>⚙️</span> Configurações</button>
    <div class="drawer-settings hidden" id="drawerSettingsPanel">
      <label class="lbl">Tamanho dos banners</label>
      <p class="muted" style="font-size:11px">Menor = mais filmes na tela. Salva só neste aparelho.</p>
      <input type="range" id="cardSizeRange" min="90" max="180" step="2" value="148" style="width:100%;margin-top:8px">
      <div class="muted" style="font-size:11px;margin-top:4px">Largura: <span id="cardSizeLabel">148</span>px</div>
      <button class="btn small block" type="button" id="btnSaveCardSize" style="margin-top:8px">Salvar tamanho</button>

      <label class="lbl">Fundo do app</label>
      <p class="muted" style="font-size:11px">Escolha uma imagem de fundo ou volte ao padrão.</p>
      <input type="file" id="bgImagePicker" accept="image/*" style="width:100%;font-size:12px;color:var(--muted)">
      <button class="btn small secondary block" type="button" id="btnResetBg" style="margin-top:8px">Fundo padrão</button>
      <label class="lbl">Rotação da tela</label>
      <p class="muted" style="font-size:11px">Trava na orientação atual (retrato ou paisagem).</p>
      <button class="btn small secondary block" type="button" id="btnLockOrient" style="margin-top:8px">🔓 Rotação livre</button>

      <div class="drawer-credits">
        <b style="font-size:16px;display:block;margin-bottom:6px">Créditos</b>
        <span style="font-size:18px;font-weight:700;color:var(--text);display:block;line-height:1.35">Raphael Rocha</span>
        <span style="font-size:18px;font-weight:700;color:var(--accent);display:block;line-height:1.35">Obliry</span>
      </div>
    </div>
  </div>
</aside>

<!-- crown password -->
<div id="crownGate" class="">
  <div class="crown-box" id="crownBox">
    <h3>Área do criador</h3>
    <p class="muted">Digite o código para abrir o criador de filmes.</p>
    <input type="password" id="crownCode" placeholder="Código" autocomplete="off">
    <div class="err" id="crownErr"></div>
    <div class="btns">
      <button class="btn small" type="button" id="crownEnter">Entrar</button>
      <button class="btn small secondary" type="button" id="crownCancel">Cancelar</button>
    </div>
  </div>
</div>

<!-- ========== ADMIN ========== -->
<div id="adminApp" class="">
  <div class="admin-header">
    <div class="brand">Cine <span>Tung</span> · Criador</div>
    <div class="btns" style="margin:0">
      <button class="btn small secondary" type="button" id="btnBackViewer">← Voltar ao Cine Tung</button>
      <button class="btn small secondary" type="button" id="btnAdminMsg">Mensagens</button>
      <button class="btn small ok" type="button" id="btnCopyFullHtml">Copiar HTML completo</button>
    </div>
  </div>
  <div class="wrap">
    <aside class="card2">
      <h2 style="margin:0 0 8px;font-size:16px">Repositório</h2>
      <p class="muted">Publicar salva e aparece no app. Apagar remove do app.</p>
      <div class="btns">
        <button class="btn small" type="button" id="btnNewMovie">+ Filme</button>
        <button class="btn small secondary" type="button" id="btnNewSeries">+ Série</button>
      </div>
      <div style="height:12px"></div>
      <div class="list" id="adminList"><div class="item"><img src="" alt=""><div class="meta"><b>A Noiva Cadáver</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Os Croods</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Angry Birds Movie</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Verity</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Dexter</b><small class="muted">AO AR · Série</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Sonic : O filme</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Cruella</b><small class="muted">AO AR · Filme</small></div></div><div class="item active"><img src="" alt=""><div class="meta"><b>Coraline e a Porta secreta</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>One Piece</b><small class="muted">AO AR · Série</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Homem-aranha: Um novo dia</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>JURASSIC WORLD</b><small class="muted">AO AR · Filme</small></div></div></div>
    </aside>
    <main class="card2" id="editor"><div style="display:flex;justify-content:space-between;gap:10px;align-items:center"><h2 style="margin:0;font-size:16px">Editar · Filme</h2><span class="badge2 on">AO AR</span></div><label class="lbl">Banner (URL ou deixe a galeria)</label><input id="f_poster" value="" placeholder="Capa da galeria já definida" data-has-data="1">
      <button class="btn small secondary" type="button" id="btnAutoPoster" style="margin-top:8px">🔎 Buscar capa pelo nome</button>
      <p class="muted" style="font-size:11px;margin-top:6px">A IA de capa busca pôster na Wikipedia pelo nome do filme (precisa de internet).</p><label class="lbl">Capa automática (pelo nome)</label><p class="muted" style="font-size:11px">Toque para buscar o pôster na Wikipedia pelo nome do filme. Não usa a foto da galeria (evita base64 gigante).</p><input id="f_poster_file" type="file" accept="image/*"><img class="banner-preview" id="bannerPreview" src=""><label class="lbl">Nome</label><input id="f_title" value="Coraline e a Porta secreta"><label class="lbl">Lançamento agendado (opcional)</label><input id="f_release" type="datetime-local" value=""><p class="muted">Vazio = libera na hora de publicar.</p><label class="lbl">Descrição</label><textarea id="f_desc"></textarea><label class="lbl">Gêneros (toque para marcar)</label><div class="genre-chip-row" id="f_genres" data-selected="Suspense|Terror|Aventura|Ficção|Fantasia|Animação"><span class="genre-chip" data-g="Ação">Ação</span><span class="genre-chip on" data-g="Aventura">Aventura</span><span class="genre-chip" data-g="Comédia">Comédia</span><span class="genre-chip" data-g="Drama">Drama</span><span class="genre-chip on" data-g="Terror">Terror</span><span class="genre-chip" data-g="Romance">Romance</span><span class="genre-chip on" data-g="Ficção">Ficção</span><span class="genre-chip on" data-g="Animação">Animação</span><span class="genre-chip" data-g="Documentário">Documentário</span><span class="genre-chip on" data-g="Suspense">Suspense</span><span class="genre-chip on" data-g="Fantasia">Fantasia</span></div><label class="lbl">URL do vídeo (YouTube, Seekee, Vimeo, mp4, m3u8 ou página)</label><input id="f_url" placeholder="YouTube, Seekee, mp4, m3u8 ou página do filme" value="https://ver.123pelicula.com/pt/detail/movie/dJOaDemkfHpWOgAihdiDM-Coraline"><label class="lbl">Legenda (.vtt opcional)</label><input id="f_sub" value=""><div class="btns"><button class="btn small ok" type="button" id="btnPublish">Publicar</button><button class="btn small secondary" type="button" id="btnDraft">Rascunho</button><button class="btn small danger" type="button" id="btnDelete">Excluir</button></div></main>
  </div>
</div>

<!-- history -->
<div class="panel" id="histPanel">
  <div class="panel-head">
    <button class="icon-btn" type="button" id="histBack">← Voltar</button>
    <div><b>Histórico</b><div class="muted">Continue de onde parou</div></div>
    <button class="btn small danger" type="button" id="btnClearHist" style="margin-left:auto">Limpar</button>
  </div>
  <div class="panel-body" id="histBody"></div>
</div>

<!-- messages user -->
<div class="panel" id="msgPanel">
  <div class="panel-head">
    <button class="icon-btn" type="button" id="msgBack">← Voltar</button>
    <div><b>Mensagem para o dono</b><div class="muted">Envia para o e-mail do dono (precisa de internet)</div></div>
  </div>
  <div class="panel-body">
    <textarea id="msgText" rows="4" autocomplete="off" autocorrect="on" autocapitalize="sentences" spellcheck="true" placeholder="Ex: Podem adicionar o filme X completo?" style="width:100%;min-height:110px;resize:vertical;-webkit-user-select:text;user-select:text;pointer-events:auto;"></textarea>
    <div class="btns" style="margin-top:10px">
      <button class="btn" type="button" id="msgSend" style="flex:1;width:auto;margin:0">Enviar ao dono</button>
      <button class="btn secondary" type="button" id="btnShareMsg" style="flex:1;width:auto;margin:0">📋 Copiar texto</button>
    </div>
    <h3 style="margin:18px 0 8px;font-size:15px">Suas mensagens</h3>
    <div id="myMsgList"></div>
  </div>
</div>

<!-- messages admin -->
<div class="panel" id="adminMsgPanel">
  <div class="panel-head">
    <button class="icon-btn" type="button" id="adminMsgBack">← Voltar</button>
    <div><b>Mensagens dos espectadores</b><div class="muted">Cole o código que eles enviaram</div></div>
  </div>
  <div class="panel-body">
    <label class="lbl">Colar mensagem recebida (código CTMSG:…)</label>
    <textarea id="adminMsgPaste" rows="3" placeholder="Cole aqui o texto que o espectador enviou…" style="width:100%;min-height:80px;resize:vertical"></textarea>
    <div class="btns" style="margin-top:8px">
      <button class="btn small ok" type="button" id="btnImportMsg">📥 Importar mensagem</button>
    </div>
    <p class="muted" id="adminMsgInfo" style="margin-top:8px;font-size:12px"></p>
    <h3 style="margin:16px 0 8px;font-size:15px">Todas as mensagens</h3>
    <div id="adminMsgList"></div>
  </div>
</div>

<!-- player -->
<div class="overlay" id="overlay">
  <div class="player-box">
    <div class="player-top">
      <div>
        <h3 id="pTitle">Título</h3>
        <div class="muted" id="pSub"></div>
      </div>
      <button class="icon-btn" type="button" id="playerClose">Fechar</button>
    </div>
    <div class="stage" id="stage"></div>
    <div class="ctrl">
      <button type="button" id="btnPlay">Play / Pause</button>
      <label class="muted">Velocidade
        <select id="speed">
          <option value="0.5">0.5x</option>
          <option value="0.75">0.75x</option>
          <option value="1" selected="">1x</option>
          <option value="1.25">1.25x</option>
          <option value="1.5">1.5x</option>
          <option value="2">2x</option>
        </select>
      </label>
      <button type="button" id="btnFs">⛶ Fullscreen</button>
      <button type="button" id="epBtn" style="display:none">Episódios</button>
    </div>
    <div class="ep-list" id="epList"></div>
  </div>
</div>

<div id="updateBanner">
  <div>
    <b>Biblioteca atualizada</b>
    <div class="muted" id="updateBannerMsg">Novos títulos do dono</div>
  </div>
  <button class="btn small" type="button" id="btnBannerOk">OK</button>
</div>
<div id="copyModal" class="">
  <div class="box">
    <b id="copyModalTitle">HTML completo Cine Tung · 11 filme(s) · 1 msg(s) · posters base64 omitidos</b>
    <p class="muted" id="copyModalHint" style="margin:0">HTML grande falha no clipboard do Android. Prefira <b>Baixar HTML</b> ou <b>Selecionar tudo</b> → Copiar. Só confie em “Copiado” se colar e ver o texto.</p>
    <textarea id="copyModalText"></textarea>
    <div class="btns" style="margin:0;flex-wrap:wrap">
      <button class="btn small" type="button" id="copyModalSelect">Selecionar tudo</button>
      <button class="btn small ok" type="button" id="copyModalCopy">📋 Copiar agora</button>
      <button class="btn small secondary" type="button" id="copyModalDownload">⬇ Baixar HTML</button>
      <button class="btn small secondary" type="button" id="copyModalClose">Fechar</button>
    </div>
  </div>
</div>
<div class="toast" id="toast" style="pointer-events: none;">Publicado</div>

<script>
(function () {
  var KEY = "cinetung_catalog_v1";
  var HIST_KEY = "cinetung_history_v1";
  var PROFILE_KEY = "cinetung_profile_v1";
  var MSG_KEY = "cinetung_messages_v1";
  var SYNC_URL_KEY = "cinetung_sync_url_v1";
  var MSG_SYNC_URL_KEY = "cinetung_msg_sync_url_v1";
  // URL padrão: HTML completo (ou catalog.json) no GitHub — todos os APKs puxam daqui
  var DEFAULT_SYNC_URL = "https://raw.githubusercontent.com/chipamzinni/Gerenciador-cine-tung/refs/heads/main/index.html";
  var lastLocalEdit = 0;
  var DELETED_KEY = "cinetung_deleted_v1";
  var THEME_KEY = "cinetung_bg_color_v1";
  var ADMIN_CODE = "OBLIRY"; // código da coroa
  var GENRES = ["Todos", "Ação", "Aventura", "Comédia", "Drama", "Terror", "Romance", "Ficção", "Animação", "Documentário", "Suspense", "Fantasia"];
  var currentGenre = "Todos";
  /* =========================================================
    CATÁLOGO + MENSAGENS EMBUTIDOS (edite aqui ou via criador)
    Ao gerar o APK, o que estiver abaixo já vem no app.
    Use o botão "Copiar HTML completo" no criador para atualizar.
    ========================================================= */
  var EMBEDDED_CATALOG = [{"id":"id_2a76kbamws5pw4p","type":"movie","title":"A Noiva Cadáver","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/a/a6/Corpse_Bride_film_poster.jpg","url":"https://peliculaplay.com/pt/detail/movie/SdvZ6sucvKvzAEgXh9Wpn-Corpse-Bride","subtitleUrl":"","published":true,"releaseAt":"","genres":["Aventura","Drama","Terror","Animação","Fantasia","Romance","Comédia"],"seasons":[],"updatedAt":1795371481265,"createdAt":1795371303913},{"id":"id_yy1zcv5mws5m2ej","type":"movie","title":"Os Croods","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/6/63/The_Croods.png","url":"https://es.cuevana4br.com/pt/detail/movie/FLo2N9wvYbX9VGkpFHEJL-The-Croods","subtitleUrl":"","published":true,"releaseAt":"","genres":["Ação","Aventura","Animação","Ficção","Fantasia","Comédia"],"seasons":[],"updatedAt":1795371218490,"createdAt":1795371125419},{"id":"id_b2iv7temws5k0ca","type":"movie","title":"Angry Birds Movie","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/f/f9/The_Angry_Birds_Movie_poster.png","url":"https://latino.solo-latino.com/pt/detail/movie/YONSwXAO83m9oYDWpPyKO-The-Angry-Birds-Movie","subtitleUrl":"","published":true,"releaseAt":"","genres":["Ação","Aventura","Animação","Fantasia"],"seasons":[],"updatedAt":1795371101871,"createdAt":1795371029434},{"id":"id_irkhfrpmws46bwf","type":"movie","title":"Verity","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/6/60/Verity_poster.jpg","url":"https://ww20.321moviesfree.com/pt/detail/movie/aqRk0pnDV1SAu7G7xI1qt-Verity","subtitleUrl":"","published":true,"releaseAt":"","genres":["Drama","Suspense","Ação"],"seasons":[],"updatedAt":1795369532720,"createdAt":1795368711615},{"id":"id_su7t8dsmwr6ugs9","type":"series","title":"Dexter","description":"","poster":"https://thumb.wikimedia.org/wikipedia/commons/thumb/b/bd/Dexter_Logo.svg/1920px-Dexter_Logo.svg.png","url":"","subtitleUrl":"","published":true,"releaseAt":"","genres":["Ação","Terror","Documentário","Suspense","Ficção"],"seasons":[{"name":"Temporada 1","episodes":[{"title":"Ep 1","url":"https://latino.solo-latino.com/pt/detail/drama/9HS3fgJL64irdVnmhcFEf-Dexter-Season-1/1","subtitleUrl":""}]}],"updatedAt":1795376336155,"createdAt":1795312730745,"_hadBase64Poster":true},{"id":"id_40f1o1wmwr6s93n","type":"movie","title":"Sonic : O filme","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/4/45/Sonic_the_Hedgehog_film_poster.jpg","url":"https://es.cuevana4br.com/pt/detail/movie/hch3oQtwgSCZQ67Kitib6-Sonic-O-Filme","subtitleUrl":"","published":true,"releaseAt":"","genres":["Ação","Aventura","Ficção","Animação","Fantasia","Suspense"],"seasons":[],"updatedAt":1795312887110,"createdAt":1795312627475},{"id":"id_v4ero27mwr6pmza","type":"movie","title":"Cruella","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/f/f6/Cruella_2021_film_poster.jpg","url":"https://ver.123pelicula.com/pt/detail/movie/Qs1nBk4HkERH83EMoPTeo-Cruella","subtitleUrl":"","published":true,"releaseAt":"","genres":["Ação","Suspense","Drama","Fantasia","Ficção"],"seasons":[],"updatedAt":1795376360659,"createdAt":1795312505494,"_hadBase64Poster":true},{"id":"id_cp4b9k1mwqyf6ke","type":"movie","title":"Coraline e a Porta secreta","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/3/36/Coraline_poster.jpg","url":"https://ver.123pelicula.com/pt/detail/movie/dJOaDemkfHpWOgAihdiDM-Coraline","subtitleUrl":"","published":true,"releaseAt":"","seasons":[],"updatedAt":1795376401490,"createdAt":1795298580734,"genres":["Suspense","Terror","Aventura","Ficção","Fantasia","Animação"],"_hadBase64Poster":true},{"id":"id_tkr31g0mwql19g3","type":"series","title":"One Piece","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/3/3f/One_Piece_Eternal_Log_-_East_Blue.png","url":"","subtitleUrl":"","published":true,"releaseAt":"","seasons":[{"name":"Temporada 1","episodes":[{"title":"Ep 1","url":"https://ver.123pelicula.com/pt/detail/drama/hKQr7WDgMEBe4uEbU3CNU-One-Piece/1","subtitleUrl":""}]},{"name":"Temporada 2","episodes":[{"title":"Ep 1","url":"https://ver.123pelicula.com/pt/detail/drama/hKQr7WDgMEBe4uEbU3CNU-One-Piece/2","subtitleUrl":""}]}],"updatedAt":1795376314536,"createdAt":1795276096275,"genres":["Ação","Aventura","Ficção","Animação","Fantasia"],"_hadBase64Poster":true},{"id":"id_mp0fas2mwqk3z5i","type":"movie","title":"Homem-aranha: Um novo dia","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/9/9a/Spider-Man_Brand_New_Day_poster.jpg","url":"https://es.cuevana4br.com/pt/detail/movie/iswnnqdfuZdhcRaFZNeuO-Spider-Man-Brand-New-Day","subtitleUrl":"","published":true,"releaseAt":"","seasons":[],"updatedAt":1795302369847,"createdAt":1795274543286,"genres":["Ficção","Ação","Aventura"]},{"id":"id_ayx45czmwqgp7lq","type":"movie","title":"JURASSIC WORLD","description":"","poster":"https://upload.wikimedia.org/wikipedia/en/6/6e/Jurassic_World_poster.jpg","url":"https://es.cuevana4br.com/pt/detail/movie/1h7LV1iNaWJ8J1Py6E2h1-Jurassic-World","subtitleUrl":"","published":true,"releaseAt":"","seasons":[],"updatedAt":1795302346121,"createdAt":1795268815550,"genres":["Ação","Ficção","Aventura","Fantasia"]}];
  var EMBEDDED_MESSAGES = [
  {
    "id": "m_mwqkhz91",
    "fromName": "Zé pilantra",
    "fromAvatar": "",
    "text": "bora trabaiar fidapulta",
    "at": 1795275196597
  }
];
  var DEMO = (EMBEDDED_CATALOG && EMBEDDED_CATALOG[0]) || null;
  var catalog = [];
  var currentId = null;
  var current = null;
  var currentEp = null;
  var tab = "lib";
  function $(id) { return document.getElementById(id); }
  function toast(msg) {
    var el = $("toast");
    el.textContent = msg;
    el.classList.add("show");
    setTimeout(function () { el.classList.remove("show"); }, 1800);
  }
  function uid() { return "id_" + Math.random().toString(36).slice(2, 9) + Date.now().toString(36); }
  function escapeHtml(s) {
    return String(s || "").replace(/[&<>"']/g, function (c) {
      return ({ "&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;","'":"&#39;" })[c];
    });
  }
  /* Comprime imagem da galeria p/ caber no HTML/GitHub (capa ~30–60KB) */
  function compressImageFile(file, maxW, quality) {
    maxW = maxW || 320;
    quality = quality || 0.62;
    return new Promise(function (resolve, reject) {
      try {
        var url = URL.createObjectURL(file);
        var img = new Image();
        img.onload = function () {
          try {
            var w = img.naturalWidth || img.width;
            var h = img.naturalHeight || img.height;
            if (!w || !h) { URL.revokeObjectURL(url); reject(new Error("img")); return; }
            function encode(mw, q) {
              var scale = Math.min(1, mw / w);
              var cw = Math.max(1, Math.round(w * scale));
              var ch = Math.max(1, Math.round(h * scale));
              var canvas = document.createElement("canvas");
              canvas.width = cw; canvas.height = ch;
              canvas.getContext("2d").drawImage(img, 0, 0, cw, ch);
              return canvas.toDataURL("image/jpeg", q);
            }
            // tenta qualidade boa e reduz até caber (~48KB de string ≈ capa leve)
            var out = encode(maxW, quality);
            var steps = [
              [300, 0.55], [280, 0.48], [240, 0.42], [220, 0.38], [200, 0.34]
            ];
            var si = 0;
            while (out.length > 65000 && si < steps.length) {
              out = encode(steps[si][0], steps[si][1]);
              si++;
            }
            URL.revokeObjectURL(url);
            resolve(out);
          } catch (e) {
            try { URL.revokeObjectURL(url); } catch (e2) {}
            reject(e);
          }
        };
        img.onerror = function () {
          try { URL.revokeObjectURL(url); } catch (e) {}
          reject(new Error("fail"));
        };
        img.src = url;
      } catch (e) { reject(e); }
    });
  }
  function compressDataUrl(dataUrl, maxW, quality) {
    maxW = maxW || 320;
    quality = quality || 0.62;
    return new Promise(function (resolve) {
      if (!dataUrl || dataUrl.indexOf("data:image") !== 0) { resolve(dataUrl || ""); return; }
      if (dataUrl.length < 65000) { resolve(dataUrl); return; }
      try {
        var img = new Image();
        img.onload = function () {
          try {
            var w = img.naturalWidth || img.width;
            var h = img.naturalHeight || img.height;
            function encode(mw, q) {
              var scale = Math.min(1, mw / Math.max(w, 1));
              var cw = Math.max(1, Math.round(w * scale));
              var ch = Math.max(1, Math.round(h * scale));
              var canvas = document.createElement("canvas");
              canvas.width = cw; canvas.height = ch;
              canvas.getContext("2d").drawImage(img, 0, 0, cw, ch);
              return canvas.toDataURL("image/jpeg", q);
            }
            var out = encode(maxW, quality);
            var steps = [[280, 0.5], [240, 0.42], [200, 0.36], [180, 0.32]];
            var si = 0;
            while (out.length > 65000 && si < steps.length) {
              out = encode(steps[si][0], steps[si][1]);
              si++;
            }
            resolve(out);
          } catch (e) { resolve(dataUrl); }
        };
        img.onerror = function () { resolve(dataUrl); };
        img.src = dataUrl;
      } catch (e) { resolve(dataUrl); }
    });
  }
  var memoryCatalog = null;
  var memoryMsgs = null;
  /* Capas multi-dispositivo:
    - URL http(s) → funciona em qualquer aparelho
    - Galeria → comprime e embute no HTML (Copiar HTML completo → GitHub/APK)
    - Mapa cinetung_posters_v1 guarda capas por id (sobreviu a limpezas leves)
  */
  var POSTER_KEY = "cinetung_posters_v1";
  function loadPosterMap() {
    try {
      var m = JSON.parse(localStorage.getItem(POSTER_KEY) || "{}");
      return m && typeof m === "object" ? m : {};
    } catch (e) { return {}; }
  }
  function savePosterMap(map) {
    try { localStorage.setItem(POSTER_KEY, JSON.stringify(map || {})); return true; }
    catch (e) {
      // se estourar, tira as maiores
      try {
        var ids = Object.keys(map || {});
        ids.sort(function (a, b) { return String(map[b]||"").length - String(map[a]||"").length; });
        var slim = {};
        for (var i = 0; i < ids.length; i++) {
          slim[ids[i]] = map[ids[i]];
          try { localStorage.setItem(POSTER_KEY, JSON.stringify(slim)); }
          catch (e2) { delete slim[ids[i]]; }
        }
        return true;
      } catch (e3) { return false; }
    }
  }
  function rememberPoster(id, poster) {
    if (!id || !poster) return;
    var map = loadPosterMap();
    map[id] = poster;
    savePosterMap(map);
  }
  function titlePosterDataUrl(title) {
    var t = String(title || "Filme").slice(0, 28);
    var svg =
      '<svg xmlns="http://www.w3.org/2000/svg" width="400" height="600" viewBox="0 0 400 600">' +
      '<defs><linearGradient id="g" x1="0" y1="0" x2="1" y2="1">' +
      '<stop offset="0%" stop-color="#1a1a22"/><stop offset="100%" stop-color="#0c0c10"/>' +
      '</linearGradient></defs>' +
      '<rect width="400" height="600" fill="url(#g)"/>' +
      '<rect x="24" y="24" width="352" height="552" rx="16" fill="none" stroke="#e8b86d" stroke-opacity="0.25" stroke-width="2"/>' +
      '<text x="200" y="300" text-anchor="middle" fill="#e8b86d" font-family="system-ui,sans-serif" font-size="22" font-weight="600">' +
      t.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/"/g,"&quot;") +
      '</text></svg>';
    return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg);
  }
  function resolvePoster(item) {
    if (!item) return "";
    var p = (item.poster || "").trim();
    if (p) return p;
    try {
      var map = loadPosterMap();
      if (item.id && map[item.id]) return map[item.id];
    } catch (e) {}
    // fallback visual (não apaga capas reais já existentes)
    return titlePosterDataUrl(item.title || "");
  }
  function attachPosters(list) {
    var map = loadPosterMap();
    return (list || []).map(function (it) {
      if (!it) return it;
      if ((!it.poster || !String(it.poster).trim()) && it.id && map[it.id]) {
        var o = {};
        for (var k in it) if (Object.prototype.hasOwnProperty.call(it, k)) o[k] = it[k];
        o.poster = map[it.id];
        return o;
      }
      return it;
    });
  }
  function bootstrapCatalogFromEmbedded() {
    try {
      hydratePostersFromEmbedded();
      var emb = (typeof EMBEDDED_CATALOG !== "undefined" && EMBEDDED_CATALOG) ? EMBEDDED_CATALOG : [];
      if (!emb || !emb.length) return;
      var local = [];
      try { local = JSON.parse(localStorage.getItem(KEY) || "[]"); } catch (e) { local = []; }
      if (!Array.isArray(local)) local = [];
      var map = {};
      local.forEach(function (it) { if (it && it.id) map[it.id] = it; });
      var changed = false;
      emb.forEach(function (it) {
        if (!it || !it.id) return;
        if (!map[it.id]) {
          map[it.id] = it;
          changed = true;
        } else {
          // preenche capa/url se local estiver vazio e remoto tiver
          var L = map[it.id];
          if ((!L.poster || !String(L.poster).trim()) && it.poster) {
            L.poster = it.poster;
            changed = true;
          }
          if ((!L.url || !String(L.url).trim()) && it.url) {
            L.url = it.url;
            changed = true;
          }
        }
        if (it.poster && it.id) rememberPoster(it.id, it.poster);
      });
      if (changed) {
        var out = Object.keys(map).map(function (k) { return map[k]; });
        try {
          localStorage.setItem(KEY, JSON.stringify(out));
          memoryCatalog = out;
        } catch (e) {
          // se estourar, grava sem posters base64 enormes
          try {
            var lite = out.map(function (it) {
              var o = {};
              for (var k in it) if (Object.prototype.hasOwnProperty.call(it, k)) o[k] = it[k];
              if (o.poster && String(o.poster).indexOf("data:") === 0 && String(o.poster).length > 80000) {
                rememberPoster(o.id, o.poster);
                o.poster = "";
              }
              return o;
            });
            localStorage.setItem(KEY, JSON.stringify(lite));
            memoryCatalog = attachPosters(lite);
          } catch (e2) {}
        }
      }
    } catch (e) {}
  }
  function hydratePostersFromEmbedded() {
    try {
      var emb = (typeof EMBEDDED_CATALOG !== "undefined" && EMBEDDED_CATALOG) ? EMBEDDED_CATALOG : [];
      if (!emb || !emb.length) return;
      var map = loadPosterMap();
      var changed = false;
      emb.forEach(function (it) {
        if (!it || !it.id || !it.poster || !String(it.poster).trim()) return;
        var p = String(it.poster).trim();
        if (map[it.id] !== p) { map[it.id] = p; changed = true; }
      });
      if (changed) savePosterMap(map);
      try {
        var list = JSON.parse(localStorage.getItem(KEY) || "[]");
        if (Array.isArray(list) && list.length) {
          var lc = false;
          list.forEach(function (it) {
            if (!it || !it.id) return;
            var embItem = null;
            for (var k = 0; k < emb.length; k++) if (emb[k] && emb[k].id === it.id) { embItem = emb[k]; break; }
            if (embItem && embItem.poster && String(embItem.poster).trim() && it.poster !== embItem.poster) {
              it.poster = embItem.poster;
              lc = true;
            }
          });
          if (lc) {
            localStorage.setItem(KEY, JSON.stringify(list));
            memoryCatalog = list;
          }
        }
      } catch (e2) {}
    } catch (e) {}
  }
  function loadRaw() {
    if (memoryCatalog && Array.isArray(memoryCatalog) && memoryCatalog.length) return attachPosters(memoryCatalog);
    try {
      var list = JSON.parse(localStorage.getItem(KEY) || "[]");
      if (Array.isArray(list) && list.length) {
        memoryCatalog = attachPosters(list);
        return memoryCatalog;
      }
    } catch (e) {}
    try {
      var bak = JSON.parse(localStorage.getItem(KEY + "_backup") || "[]");
      if (Array.isArray(bak) && bak.length) {
        memoryCatalog = bak;
        return bak;
      }
    } catch (e) {}
    if (EMBEDDED_CATALOG && EMBEDDED_CATALOG.length) {
      memoryCatalog = EMBEDDED_CATALOG.slice();
      return memoryCatalog;
    }
    return memoryCatalog || [];
  }
  function saveRaw(list, msg) {
    catalog = Array.isArray(list) ? list : [];
    memoryCatalog = catalog;
    lastLocalEdit = Date.now();
    var fullJson = JSON.stringify(catalog);
    var saved = false;
    try {
      localStorage.setItem(KEY, fullJson);
      localStorage.setItem(KEY + "_updated", String(Date.now()));
      saved = true;
    } catch (e) {
      // quota exceeded (common with data:image posters) — try lite version without huge base64
      try {
        var lite = catalog.map(function (item) {
          var o = {};
          for (var k in item) if (Object.prototype.hasOwnProperty.call(item, k)) o[k] = item[k];
          if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 90000) {
            o.poster = "";
            o._hadBase64Poster = true;
          }
          return o;
        });
        localStorage.setItem(KEY, JSON.stringify(lite));
        localStorage.setItem(KEY + "_updated", String(Date.now()));
        saved = true;
        try { toast("Salvo (posters base64 grandes omitidos no armazenamento · ficam nesta sessão)"); } catch (e2) {}
      } catch (e2) {
        try { toast("Sem espaço no aparelho · lista só nesta sessão. Use URL de poster em vez de galeria."); } catch (e3) {}
      }
    }
    try { localStorage.setItem(KEY + "_backup", fullJson); } catch (e) {
      try { localStorage.setItem(KEY + "_backup", JSON.stringify(catalog.map(function(it){ var o={}; for(var k in it)o[k]=it[k]; if(o.poster&&o.poster.length>120000&&o.poster.indexOf("data:")===0)o.poster=""; return o; }))); } catch (e2) {}
    }
    try {
      (catalog || []).forEach(function (it) {
        if (it && it.id && it.poster) rememberPoster(it.id, it.poster);
      });
    } catch (e) {}
    if (msg) toast(msg);
    else if (saved) { /* silent ok */ }
    try { renderAdminList(); } catch (e) {}
    try { renderGrid(); } catch (e) {}
  }
  function getSyncUrl() {
    var u = (localStorage.getItem(SYNC_URL_KEY) || "").trim();
    return u || DEFAULT_SYNC_URL;
  }
  function setSyncStatus(msg) {
    var el = $("syncStatus");
    if (el) el.textContent = msg || "";
  }
  function showUpdateBanner(msg) {
    var b = $("updateBanner");
    if (!b) return;
    $("updateBannerMsg").textContent = msg || "Novos títulos disponíveis";
    b.classList.add("show");
  }
  /* Gera URLs alternativas (raw + jsDelivr) — mais estável em WebView/APK */
  function expandCatalogUrls(url) {
    var list = [];
    url = (url || "").trim();
    if (!url) return list;
    list.push(url);
    try {
      // raw.githubusercontent.com/user/repo/branch/path
      var m = url.match(/^https?:\/\/raw\.githubusercontent\.com\/([^\/]+)\/([^\/]+)\/([^\/]+)\/(.+)$/i);
      if (m) {
        var user = m[1], repo = m[2], branch = m[3], filePath = m[4];
        list.push("https://cdn.jsdelivr.net/gh/" + user + "/" + repo + "@" + branch + "/" + filePath);
        list.push("https://cdn.jsdelivr.net/gh/" + user + "/" + repo + "/" + filePath);
      }
      // github.com/user/repo/blob/branch/path → raw + jsdelivr
      var m2 = url.match(/^https?:\/\/github\.com\/([^\/]+)\/([^\/]+)\/blob\/([^\/]+)\/(.+)$/i);
      if (m2) {
        var u2 = m2[1], r2 = m2[2], b2 = m2[3], p2 = m2[4];
        list.push("https://raw.githubusercontent.com/" + u2 + "/" + r2 + "/" + b2 + "/" + p2);
        list.push("https://cdn.jsdelivr.net/gh/" + u2 + "/" + r2 + "@" + b2 + "/" + p2);
      }
      // github.com/user/repo/raw/branch/path
      var m3 = url.match(/^https?:\/\/github\.com\/([^\/]+)\/([^\/]+)\/raw\/([^\/]+)\/(.+)$/i);
      if (m3) {
        list.push("https://raw.githubusercontent.com/" + m3[1] + "/" + m3[2] + "/" + m3[3] + "/" + m3[4]);
        list.push("https://cdn.jsdelivr.net/gh/" + m3[1] + "/" + m3[2] + "@" + m3[3] + "/" + m3[4]);
      }
    } catch (e) {}
    // unique
    var seen = {}, out = [];
    list.forEach(function (u) {
      var k = u.split("?")[0];
      if (!seen[k]) { seen[k] = 1; out.push(u); }
    });
    return out;
  }
  /* Extrai lista de filmes de: JSON puro []  OU  HTML com var EMBEDDED_CATALOG = [...] */
  function parseCatalogPayload(text) {
    text = String(text || "").trim();
    if (!text) throw new Error("resposta vazia");
    // 1) JSON direto
    if (text.charAt(0) === "[") {
      var arr = JSON.parse(text);
      if (!Array.isArray(arr)) throw new Error("JSON precisa ser lista []");
      return arr;
    }
    // 2) HTML com EMBEDDED_CATALOG
    var needle = "var EMBEDDED_CATALOG";
    var i = text.indexOf(needle);
    if (i < 0) {
      // tenta só o nome sem var
      i = text.indexOf("EMBEDDED_CATALOG");
      if (i < 0) throw new Error("Sem EMBEDDED_CATALOG no HTML remoto");
    }
    var eq = text.indexOf("=", i);
    if (eq < 0) throw new Error("EMBEDDED_CATALOG sem =");
    var j = eq + 1;
    while (j < text.length && /\s/.test(text.charAt(j))) j++;
    if (text.charAt(j) !== "[") throw new Error("EMBEDDED_CATALOG não é array");
    var depth = 0, inStr = false, strQ = "", esc = false, start = j;
    for (; j < text.length; j++) {
      var ch = text.charAt(j);
      if (inStr) {
        if (esc) { esc = false; continue; }
        if (ch === "\\") { esc = true; continue; }
        if (ch === strQ) inStr = false;
        continue;
      }
      if (ch === '"' || ch === "'" || ch === "`") { inStr = true; strQ = ch; continue; }
      if (ch === "[") { depth++; continue; }
      if (ch === "]") {
        depth--;
        if (depth === 0) {
          var json = text.slice(start, j + 1);
          var parsed = JSON.parse(json);
          if (!Array.isArray(parsed)) throw new Error("catálogo remoto inválido");
          return parsed;
        }
      }
    }
    throw new Error("Não fechou EMBEDDED_CATALOG");
  }
  function fetchCatalogFirstOk(urls) {
    var i = 0;
    function next() {
      if (i >= urls.length) return Promise.reject(new Error("Nenhuma URL respondeu"));
      var base = urls[i++];
      var u = base + (base.indexOf("?") >= 0 ? "&" : "?") + "t=" + Date.now();
      return fetch(u, { cache: "no-store", mode: "cors" })
        .then(function (r) {
          if (!r.ok) throw new Error("HTTP " + r.status);
          return r.text();
        })
        .then(function (txt) { return parseCatalogPayload(txt); })
        .catch(function () {
          // fallback XHR (alguns WebViews se comportam melhor)
          return new Promise(function (resolve, reject) {
            try {
              var xhr = new XMLHttpRequest();
              xhr.open("GET", u, true);
              xhr.setRequestHeader("Cache-Control", "no-cache");
              xhr.onload = function () {
                if (xhr.status >= 200 && xhr.status < 300) {
                  try { resolve(parseCatalogPayload(xhr.responseText)); }
                  catch (e) { reject(e); }
                } else reject(new Error("HTTP " + xhr.status));
              };
              xhr.onerror = function () { reject(new Error("rede")); };
              xhr.send();
            } catch (e) { reject(e); }
          });
        })
        .catch(function () { return next(); });
    }
    return next();
  }
  function getDeletedIds() {
    try {
      var d = JSON.parse(localStorage.getItem(DELETED_KEY) || "[]");
      return Array.isArray(d) ? d : [];
    } catch (e) { return []; }
  }
  function addDeletedId(id) {
    if (!id) return;
    var d = getDeletedIds();
    if (d.indexOf(id) < 0) {
      d.push(id);
      if (d.length > 200) d = d.slice(-200);
      try { localStorage.setItem(DELETED_KEY, JSON.stringify(d)); } catch (e) {}
    }
  }
  function clearDeletedId(id) {
    var d = getDeletedIds().filter(function (x) { return x !== id; });
    try { localStorage.setItem(DELETED_KEY, JSON.stringify(d)); } catch (e) {}
  }
  function mergeCatalog(localList, remoteList) {
    var deleted = {};
    getDeletedIds().forEach(function (id) { deleted[id] = true; });
    var map = {};
    (localList || []).forEach(function (m) { if (m && m.id) map[m.id] = m; });
    (remoteList || []).forEach(function (m) {
      if (!m || !m.id) return;
      if (deleted[m.id]) return; // respeita exclusão local
      var loc = map[m.id];
      if (!loc) {
        map[m.id] = m; // novo do remoto
      } else {
        var ru = Number(m.updatedAt) || 0;
        var lu = Number(loc.updatedAt) || 0;
        if (ru > lu) map[m.id] = m;
      }
    });
    return Object.keys(map).map(function (k) { return map[k]; });
  }
  function pullCatalogFromGitHub(silent) {
    var url = getSyncUrl();
    if (!url) {
      if (!silent) toast("Configure a URL do catalog.json no GitHub");
      setSyncStatus("Sem URL configurada.");
      return Promise.resolve(false);
    }
    // não sobrescreve edições locais recentes (publicar/remover)
    if (silent && lastLocalEdit && (Date.now() - lastLocalEdit < 5 * 60 * 1000)) {
      setSyncStatus("Sync pausado (edição local recente)");
      return Promise.resolve(false);
    }
    var localNow = loadRaw();
    var before = JSON.stringify(localNow);
    var urls = expandCatalogUrls(url);
    setSyncStatus("Baixando catálogo (" + urls.length + " rota(s))…");
    return fetchCatalogFirstOk(urls)
      .then(function (data) {
        if (!Array.isArray(data)) throw new Error("JSON precisa ser uma lista []");
        var merged = silent ? mergeCatalog(localNow, data) : data; // manual pull = force remote
        // se for pull manual e o usuário quiser, usa remote puro; se silent, mescla
        if (!silent) merged = data;
        var after = JSON.stringify(merged);
        var changed = after !== before;
        if (changed) {
          // ao salvar do sync, não marca como "local edit" forte — mas saveRaw marca; ajustamos
          var prevEdit = lastLocalEdit;
          saveRaw(merged, (!silent && changed) ? "Catálogo atualizado do GitHub" : null);
          if (silent) lastLocalEdit = prevEdit; // não conta sync como edição local
        }
        setSyncStatus("OK: " + merged.length + " item(ns) · " + new Date().toLocaleString());
        if (changed && silent) {
          showUpdateBanner(merged.length + " título(s) na biblioteca");
        }
        if (changed && !silent) toast("Biblioteca atualizada");
        if (!changed && !silent) toast("Já está atualizado");
        return changed;
      })
      .catch(function (e) {
        var msg = (e && e.message) ? e.message : String(e);
        setSyncStatus("Falha: " + msg + " · confira se o repo é público e a URL está certa");
        if (!silent) toast("Falha ao puxar catálogo");
        return false;
      });
  }
  function ensureDemo() {
    try {
      var list = loadRaw();
      if (!Array.isArray(list)) list = [];
      if (list.length === 0 && EMBEDDED_CATALOG && EMBEDDED_CATALOG.length) {
        list = EMBEDDED_CATALOG.slice();
        memoryCatalog = list;
        try {
          localStorage.setItem(KEY, JSON.stringify(list));
          localStorage.setItem(KEY + "_updated", String(Date.now()));
        } catch (e) {}
      }
      catalog = list;
      memoryCatalog = list;
      try {
        var msgs = [];
        try { msgs = JSON.parse(localStorage.getItem(MSG_KEY) || "[]"); } catch (e) { msgs = []; }
        if ((!msgs || !msgs.length) && EMBEDDED_MESSAGES && EMBEDDED_MESSAGES.length) {
          msgs = EMBEDDED_MESSAGES.slice();
          memoryMsgs = msgs;
          try { localStorage.setItem(MSG_KEY, JSON.stringify(msgs)); } catch (e) {}
        }
      } catch (e) {}
    } catch (e) {
      catalog = (EMBEDDED_CATALOG || []).slice();
      memoryCatalog = catalog;
    }
  }
  function isReleased(item) {
    if (!item || !item.published) return false;
    if (!item.releaseAt) return true;
    var t = new Date(item.releaseAt).getTime();
    if (isNaN(t)) return true;
    return Date.now() >= t;
  }
  function publishedList() { return loadRaw().filter(isReleased); }
  /* splash: controlada pelo script isolado no body */
  function endSplash() {
    try { if (window.__obliryEndSplash) window.__obliryEndSplash(); } catch (e) {}
    var splash = document.getElementById("splash");
    if (!splash) return;
    splash.classList.add("hide");
    setTimeout(function () {
      try { splash.classList.add("gone"); if (splash.parentNode) splash.parentNode.removeChild(splash); } catch (e) {}
    }, 900);
  }
  function runSplash() {
    /* intro já inicia sozinha no script do body */
  }
  /* mode */
  function showViewer() {
    try {
      var a = $("adminApp");
      var v = $("viewApp");
      if (a) {
        a.classList.remove("on");
        a.style.display = "none";
      }
      if (v) {
        v.classList.add("on");
        v.style.display = "block";
      }
      localStorage.setItem("cinetung_ui", "viewer");
      // scroll pro topo do espectador
      try { window.scrollTo(0, 0); } catch (e) {}
      try { document.documentElement.scrollTop = 0; document.body.scrollTop = 0; } catch (e) {}
      try { setTab("lib"); } catch (e) {}
      renderGrid();
      loadProfileUI();
    } catch (e) {
      try { toast("Erro ao voltar"); } catch (e2) {}
    }
  }
  function showAdmin() {
    window.__cineAdminUnlocked = true;
    try {
      var a = $("adminApp");
      var v = $("viewApp");
      if (v) {
        v.classList.remove("on");
        v.style.display = "none";
      }
      if (a) {
        a.classList.add("on");
        a.style.display = "block";
      }
      localStorage.setItem("cinetung_ui", "admin");
      try { window.scrollTo(0, 0); } catch (e) {}
      catalog = loadRaw();
      try { renderAdminList(); } catch (e) {}
      try { renderEditor(); } catch (e) {}
    } catch (e) {
      try { toast("Erro ao abrir criador"); } catch (e2) {}
    }
  }
  /* crown gate */
  function openCrown() {
    $("crownGate").classList.add("open");
    $("crownErr").textContent = "";
    $("crownCode").value = "";
    setTimeout(function () { $("crownCode").focus(); }, 50);
  }
  function closeCrown() {
    $("crownGate").classList.remove("open");
  }
  function tryCrown() {
    var code = ($("crownCode").value || "").trim();
    if (code === ADMIN_CODE) {
      closeCrown();
      window.__cineAdminUnlocked = true;
      showAdmin();
      toast("Bem-vindo ao criador");
    } else {
      $("crownErr").textContent = "Código incorreto";
    }
  }
  if ($("crownBtn")) $("crownBtn").onclick = openCrown;
  if ($("crownEnter")) $("crownEnter").onclick = tryCrown;
  if ($("crownCancel")) $("crownCancel").onclick = closeCrown;
  if ($("crownCode")) $("crownCode").addEventListener("keydown", function (e) {
    if (e.key === "Enter") tryCrown();
  });
  // clicar fora da caixa fecha
  if ($("crownGate")) $("crownGate").addEventListener("click", function (e) {
    if (e.target === $("crownGate")) closeCrown();
  });
  if ($("btnBackViewer")) {
    $("btnBackViewer").onclick = function (ev) {
      try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
      try { if (ev && ev.stopPropagation) ev.stopPropagation(); } catch (e) {}
      showViewer();
    };
  }
  /* profile */
  function getProfile() {
    var p = { name: "Espectador", avatar: "" };
    try { p = Object.assign(p, JSON.parse(localStorage.getItem(PROFILE_KEY) || "{}")); } catch (e) {}
    return p;
  }
  function applyAvatar(src) {
    var a = $("avatarImg"), t = $("topAvatar");
    if (a) { if (src) a.src = src; else a.removeAttribute("src"); }
    if (t) { if (src) t.src = src; else t.removeAttribute("src"); }
  }
  function loadProfileUI() {
    var p = getProfile();
    applyAvatar(p.avatar || "");
    if ($("profileName")) $("profileName").value = p.name || "";
  }
  if ($("btnSaveProfile")) $("btnSaveProfile").onclick = function () {
    var name = (($("profileName") && $("profileName").value) || "Espectador").trim();
    var avatar = ($("avatarImg") && $("avatarImg").getAttribute("src")) || "";
    localStorage.setItem(PROFILE_KEY, JSON.stringify({ name: name, avatar: avatar }));
    applyAvatar(avatar);
    toast("Perfil salvo");
  };
  if ($("avatarFile")) $("avatarFile").onchange = function (ev) {
    var f = ev.target.files && ev.target.files[0];
    if (!f) return;
    var r = new FileReader();
    r.onload = function () {
      if ($("avatarImg")) $("avatarImg").src = r.result;
      applyAvatar(r.result);
    };
    r.readAsDataURL(f);
  };
  if ($("topAvatar")) {
    $("topAvatar").onclick = function (ev) {
      try { if (ev) { ev.preventDefault(); ev.stopPropagation(); } } catch (e) {}
      try { closeDrawer(); } catch (e) {}
      setTab("profile");
    };
  }
  /* tabs */
  function setTab(name) {
    try { window.setTab = setTab; } catch (e) {}
    try {
      // fecha overlays que possam bloquear cliques
      try { closeDrawer(); } catch (e) {}
      try {
        var cm = $("copyModal");
        if (cm) cm.classList.remove("open");
      } catch (e) {}
      try {
        document.body.classList.remove("drawer-open");
        if ($("bottomNav")) $("bottomNav").style.pointerEvents = "auto";
        if ($("viewApp")) $("viewApp").style.pointerEvents = "auto";
      } catch (e) {}
      name = String(name || "lib");
      tab = name;
      ["lib", "movies", "series", "profile"].forEach(function (id) {
        var el = $("page-" + id);
        if (el) {
          if (id === name) el.classList.add("active");
          else el.classList.remove("active");
        }
      });
      var nav = $("bottomNav");
      if (nav) {
        nav.style.pointerEvents = "auto";
        nav.querySelectorAll("button[data-tab]").forEach(function (b) {
          if (b.getAttribute("data-tab") === name) b.classList.add("active");
          else b.classList.remove("active");
        });
      }
      var sb = $("searchBar");
      if (sb) sb.style.display = name === "profile" ? "none" : "flex";
      try { renderGrid(); } catch (e) {}
    } catch (e) {
      try { console.log("setTab error", e); } catch (e2) {}
    }
  }
  try { window.setTab = setTab; } catch (e) {}
  // liga cada botão da barra inferior individualmente (mais confiável no WebView)
  (function () {
    var nav = $("bottomNav");
    if (!nav) return;
    nav.querySelectorAll("button[data-tab]").forEach(function (b) {
      b.onclick = function (ev) {
        try { if (ev) { ev.preventDefault(); ev.stopPropagation(); } } catch (e) {}
        setTab(b.getAttribute("data-tab"));
      };
    });
  })();
  if ($("q")) $("q").oninput = function () { renderGrid(); };
  /* ——— gaveta de gêneros + configurações ——— */
  function openDrawer() {
    try { document.body.classList.add("drawer-open"); } catch (e) {}
    if ($("genreDrawer")) {
      $("genreDrawer").classList.add("open");
      $("genreDrawer").style.pointerEvents = "auto";
      $("genreDrawer").setAttribute("aria-hidden", "false");
    }
    if ($("drawerBackdrop")) {
      $("drawerBackdrop").classList.add("open");
      $("drawerBackdrop").style.pointerEvents = "auto";
    }
    renderDrawerGenres();
  }
  function closeDrawer() {
    try { document.body.classList.remove("drawer-open"); } catch (e) {}
    if ($("genreDrawer")) {
      $("genreDrawer").classList.remove("open");
      $("genreDrawer").style.pointerEvents = "";
      $("genreDrawer").setAttribute("aria-hidden", "true");
    }
    if ($("drawerBackdrop")) {
      $("drawerBackdrop").classList.remove("open");
      $("drawerBackdrop").style.pointerEvents = "none";
    }
    if ($("drawerSettingsPanel")) $("drawerSettingsPanel").classList.add("hidden");
    try {
      var nav = $("bottomNav");
      if (nav) nav.style.pointerEvents = "auto";
      if ($("viewApp")) $("viewApp").style.pointerEvents = "";
    } catch (e) {}
  }
  function renderDrawerGenres() {
    var box = $("drawerGenreList");
    if (!box) return;
    box.innerHTML = "";
    GENRES.forEach(function (g) {
      var b = document.createElement("button");
      b.type = "button";
      b.className = "drawer-item" + (g === currentGenre ? " active" : "");
      b.innerHTML = "<span>" + (g === "Todos" ? "•" : "·") + "</span> " + escapeHtml(g);
      b.onclick = function () {
        currentGenre = g;
        renderGrid();
        closeDrawer();
        toast("Gênero: " + g);
      };
      box.appendChild(b);
    });
  }
  var __bgIdb = null;
  function bgIdb() {
    return new Promise(function (resolve) {
      try {
        if (__bgIdb) { resolve(__bgIdb); return; }
        if (!window.indexedDB) { resolve(null); return; }
        var req = indexedDB.open("cinetung_bg_db", 1);
        req.onupgradeneeded = function () {
          try {
            if (!req.result.objectStoreNames.contains("kv")) req.result.createObjectStore("kv");
          } catch (e) {}
        };
        req.onsuccess = function () { __bgIdb = req.result; resolve(__bgIdb); };
        req.onerror = function () { resolve(null); };
      } catch (e) { resolve(null); }
    });
  }
  function bgIdbSet(key, val) {
    return bgIdb().then(function (db) {
      if (!db) return false;
      return new Promise(function (resolve) {
        try {
          var tx = db.transaction("kv", "readwrite");
          tx.objectStore("kv").put(val, key);
          tx.oncomplete = function () { resolve(true); };
          tx.onerror = function () { resolve(false); };
        } catch (e) { resolve(false); }
      });
    });
  }
  function bgIdbGet(key) {
    return bgIdb().then(function (db) {
      if (!db) return null;
      return new Promise(function (resolve) {
        try {
          var tx = db.transaction("kv", "readonly");
          var r = tx.objectStore("kv").get(key);
          r.onsuccess = function () { resolve(r.result || null); };
          r.onerror = function () { resolve(null); };
        } catch (e) { resolve(null); }
      });
    });
  }
  function clearBgImage() {
    try {
      var bg = document.getElementById("appBg");
      if (bg) {
        bg.style.backgroundImage = "";
        bg.style.backgroundColor = "#0a0a0b";
      }
      document.body.style.backgroundImage = "";
      localStorage.removeItem(BG_IMAGE_KEY);
      try { bgIdbSet("bg", ""); } catch (e) {}
    } catch (e) {}
  }
  function paintBg(dataUrl) {
    var bg = document.getElementById("appBg");
    if (!bg) return;
    if (!dataUrl) {
      bg.style.backgroundImage = "";
      bg.style.backgroundColor = "#0a0a0b";
      return;
    }
    var safe = String(dataUrl).replace(/'/g, "%27").replace(/\n/g, "");
    // vinheta + imagem (só visual; layer não captura clique)
    bg.style.backgroundColor = "#0a0a0b";
    bg.style.backgroundImage =
      "linear-gradient(180deg, rgba(8,8,10,.75) 0%, rgba(8,8,10,.5) 45%, rgba(8,8,10,.85) 100%), url('" + safe + "')";
    bg.style.backgroundSize = "cover, cover";
    bg.style.backgroundPosition = "center, center";
    bg.style.backgroundRepeat = "no-repeat, no-repeat";
    bg.style.pointerEvents = "none";
  }
  function applyBgColor(hex) {
    if (!hex) hex = "#0a0a0b";
    try {
      document.documentElement.style.setProperty("--bg", hex);
      document.body.style.backgroundColor = hex;
      var bg = document.getElementById("appBg");
      if (bg) bg.style.backgroundColor = hex;
      localStorage.setItem(THEME_KEY, hex);
    } catch (e) {}
  }
  function persistBgImage(dataUrl) {
    paintBg(dataUrl);
    // 1) IndexedDB (sobrevive fechar APK / desligar)
    return bgIdbSet("bg", dataUrl).then(function (okIdb) {
      // 2) localStorage backup bem comprimido
      function tryLS(str) {
        try { localStorage.setItem(BG_IMAGE_KEY, str); return true; } catch (e) { return false; }
      }
      if (tryLS(dataUrl)) return true;
      var steps = [[400, 0.35], [300, 0.28], [220, 0.22], [160, 0.18]];
      return steps.reduce(function (chain, step) {
        return chain.then(function (done) {
          if (done) return true;
          return compressDataUrl(dataUrl, step[0], step[1]).then(function (small) {
            if (tryLS(small)) return true;
            return false;
          });
        });
      }, Promise.resolve(false)).then(function (okLs) {
        return okIdb || okLs;
      });
    });
  }
  function applyBgImage(dataUrl) {
    if (!dataUrl) return;
    paintBg(dataUrl);
    persistBgImage(dataUrl).then(function (ok) {
      if (ok) { try { toast("Fundo salvo neste aparelho"); } catch (e) {} }
    });
  }
  function loadTheme() {
    // IndexedDB primeiro (permanente)
    bgIdbGet("bg").then(function (img) {
      if (img) { paintBg(img); return; }
      try {
        var ls = localStorage.getItem(BG_IMAGE_KEY);
        if (ls) { paintBg(ls); return; }
        var c = localStorage.getItem(THEME_KEY);
        if (c) applyBgColor(c);
      } catch (e) {}
    }).catch(function () {
      try {
        var ls2 = localStorage.getItem(BG_IMAGE_KEY);
        if (ls2) paintBg(ls2);
      } catch (e) {}
    });
  }
  if ($("btnGenreDrawer")) $("btnGenreDrawer").onclick = function () { openDrawer(); };
  if ($("drawerClose")) $("drawerClose").onclick = function () { closeDrawer(); };
  if ($("drawerBackdrop")) $("drawerBackdrop").onclick = function () { closeDrawer(); };
  if ($("btnDrawerHome")) $("btnDrawerHome").onclick = function () {
    currentGenre = "Todos";
    setTab("lib");
    closeDrawer();
  };
  if ($("btnDrawerSettings")) $("btnDrawerSettings").onclick = function () {
    var p = $("drawerSettingsPanel");
    if (!p) return;
    p.classList.toggle("hidden");
  };
  if ($("bgImagePicker")) {
    $("bgImagePicker").onchange = function () {
      var f = this.files && this.files[0];
      if (!f) return;
      // comprime forte antes de aplicar (fundo precisa caber no localStorage)
      compressImageFile(f, 480, 0.4).then(function (dataUrl) {
        applyBgImage(dataUrl);
      }).catch(function () {
        var r = new FileReader();
        r.onload = function () {
          compressDataUrl(String(r.result || ""), 480, 0.4).then(function (d) {
            applyBgImage(d);
          });
        };
        r.readAsDataURL(f);
      });
    };
  }
  if ($("btnResetBg")) {
    $("btnResetBg").onclick = function () {
      clearBgImage();
      applyBgColor("#0a0a0b");
      toast("Fundo padrão");
    };
  }
  try { loadTheme(); } catch (e) {}
  /* grids */
  function itemGenres(item) {
    if (!item) return [];
    if (Array.isArray(item.genres)) return item.genres;
    if (typeof item.genre === "string" && item.genre) return [item.genre];
    return [];
  }
  function filtered(type) {
    var q = ($("q").value || "").toLowerCase().trim();
    var list = publishedList();
    if (type === "movie") list = list.filter(function (x) { return x.type === "movie"; });
    if (type === "series") list = list.filter(function (x) { return x.type === "series"; });
    if (currentGenre && currentGenre !== "Todos") {
      list = list.filter(function (x) {
        var g = itemGenres(x);
        if (!g.length) return false;
        return g.indexOf(currentGenre) >= 0;
      });
    }
    if (q) {
      list = list.filter(function (x) {
        return (x.title || "").toLowerCase().indexOf(q) >= 0 ||
          (x.description || "").toLowerCase().indexOf(q) >= 0;
      });
    }
    return list;
  }
  function fillGrid(gridId, emptyId, list) {
    var grid = $(gridId), empty = $(emptyId);
    grid.innerHTML = "";
    if (!list.length) {
      empty.style.display = "block";
      empty.textContent = "Nenhum título nesta aba.";
      return;
    }
    empty.style.display = "none";
    list.forEach(function (item) {
      var el = document.createElement("div");
      el.className = "card";
      var openIt = function () { openPlayer(item); };
      // toque: só abre se for tap (pouco movimento), não ao deslizar a lista
      var tx = 0, ty = 0, moved = false;
      el.addEventListener("touchstart", function (ev) {
        moved = false;
        try {
          var t = ev.changedTouches && ev.changedTouches[0];
          if (t) { tx = t.clientX; ty = t.clientY; }
        } catch (e) {}
      }, { passive: true });
      el.addEventListener("touchmove", function (ev) {
        try {
          var t = ev.changedTouches && ev.changedTouches[0];
          if (!t) return;
          if (Math.abs(t.clientX - tx) > 12 || Math.abs(t.clientY - ty) > 12) moved = true;
        } catch (e) {}
      }, { passive: true });
      el.addEventListener("touchend", function (ev) {
        if (moved) return;
        try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
        openIt();
      }, { passive: false });
      el.addEventListener("click", function (ev) {
        // desktop / fallback; no mobile o touchend já abriu
        if (ev && ev.detail === 0) return;
        openIt();
      });
      var poster = String(resolvePoster(item) || item.poster || "").trim();
      var initials = (item.title || "?").replace(/[^A-Za-z0-9À-ÿ ]/g, " ").trim().split(/\s+/).slice(0, 2).map(function (w) { return w.charAt(0); }).join("").toUpperCase() || "?";
      var posterHtml;
      if (poster) {
        posterHtml = '<div class="poster" style="background-image:url(\'' + poster.replace(/'/g, "%27").replace(/"/g, "&quot;") + '\')"></div>';
      } else {
        posterHtml = '<div class="poster no-art" data-initials="' + escapeHtml(initials) + '"></div>';
      }
      el.innerHTML =
        posterHtml +
        '<div class="info"><b>' + escapeHtml(item.title) + '</b>' +
        '<small>' + (item.type === "movie" ? "Filme" : "Série") + '</small>' +
        '<div class="badge">' + (item.type === "series" ? "Série" : "Completo") + '</div></div>';
      grid.appendChild(el);
    });
  }
  function renderGrid() {
    ensureDemo();
    var n = publishedList().length;
    var ts = Number(localStorage.getItem(KEY + "_updated") || Date.now());
    if ($("syncInfo")) $("syncInfo").textContent = n + " título(s) · " + new Date(ts).toLocaleString();
    var gtxt = "Gênero: " + (currentGenre || "Todos");
    if ($("genreLabelLib")) $("genreLabelLib").textContent = gtxt;
    if ($("genreLabelMovies")) $("genreLabelMovies").textContent = gtxt;
    if ($("genreLabelSeries")) $("genreLabelSeries").textContent = gtxt;
    fillGrid("grid-lib", "empty-lib", filtered("all"));
    fillGrid("grid-movies", "empty-movies", filtered("movie"));
    fillGrid("grid-series", "empty-series", filtered("series"));
    try { renderDrawerGenres(); } catch (e) {}
    try { renderHomeHero(); } catch (e) {}
  }
  /* player - multi site */
  function parseYouTubeId(url) {
    if (!url) return null;
    try {
      var u = new URL(url);
      var host = (u.hostname || "").toLowerCase();
      if (host.indexOf("youtu.be") >= 0) {
        var id = u.pathname.replace(/^\//, "").split("/")[0];
        return id && id.length >= 6 ? id : null;
      }
      if (host.indexOf("youtube.com") >= 0 || host.indexOf("youtube-nocookie.com") >= 0) {
        if (u.searchParams.get("v")) return u.searchParams.get("v");
        var parts = u.pathname.split("/").filter(Boolean);
        var idx = parts.indexOf("embed");
        if (idx >= 0 && parts[idx + 1]) return parts[idx + 1];
        idx = parts.indexOf("shorts");
        if (idx >= 0 && parts[idx + 1]) return parts[idx + 1];
        idx = parts.indexOf("live");
        if (idx >= 0 && parts[idx + 1]) return parts[idx + 1];
      }
    } catch (e) {}
    return null;
  }
  function parseVimeoId(url) {
    if (!url) return null;
    try {
      var u = new URL(url);
      var host = (u.hostname || "").toLowerCase();
      if (host.indexOf("vimeo.com") < 0) return null;
      var parts = u.pathname.split("/").filter(Boolean);
      if (parts[0] === "video" && parts[1]) return parts[1];
      if (/^\d+$/.test(parts[0])) return parts[0];
    } catch (e) {}
    return null;
  }
  function isDirectMediaUrl(url) {
    if (!url) return false;
    var clean = String(url).split("?")[0].split("#")[0].toLowerCase();
    return /\.(mp4|webm|ogg|ogv|m4v|mov|mkv|m3u8|mpd|ts|aac|mp3)$/i.test(clean) ||
          /\/(playlist|manifest|master)\.m3u8/i.test(url) ||
          /format=m3u8|type=m3u8|ext=m3u8/i.test(url);
  }
  function isWebPlayerPage(url) {
    if (!url) return false;
    try {
      var host = (new URL(url).hostname || "").toLowerCase();
      if (host.indexOf("seekee.ai") >= 0) return true;
      if (host.indexOf("aiseekee.com") >= 0) return true;
      var known = [
        "streamtape", "doodstream", "dood.", "mixdrop", "upstream", "voe.sx", "voe.",
        "filemoon", "vidmoly", "ok.ru", "vk.com", "vkvideo", "drive.google.com",
        "mega.nz", "mediafire.com", "pixeldrain", "krakenfiles",
        "streamwish", "filelions", "vidhide", "lulustream", "lulu.st",
        "mp4upload", "uqload", "vidoza", "streamlare", "streamvid",
        "yourupload", "vidcloud", "vidguard", "vgfplay", "d0o0d",
        "rabbitstream", "megacloud", "vidsrc", "embed.su", "2embed",
        "player.vimeo.com", "dailymotion.com", "dai.ly",
        "archive.org", "rumble.com", "bitchute.com",
        "dropbox.com", "onedrive.live.com", "1drv.ms",
        "ibbfa", "smoothpre", "streamhub", "streamable.com"
      ];
      for (var i = 0; i < known.length; i++) {
        if (host.indexOf(known[i]) >= 0) return true;
      }
      if (host.indexOf("h5.") >= 0 && /\/(drive\/s|share|play|watch|v\/)/i.test(url)) return true;
      // caminhos típicos de player embed
      if (/\/(e|embed|v|play|player|watch)\//i.test(url)) return true;
    } catch (e) {}
    return false;
  }
  function isLikelyIframeBlocked(url) {
    if (!url) return false;
    try {
      var host = (new URL(url).hostname || "").toLowerCase();
      var blocked = [
        "cuevana", "cuevana3", "cuevana4", "cuevana2", "cuevana5",
        "pelisplushd", "pelisplus", "repelis", "seriesyonkis", "seriesflix",
        "gnula", "lapse", "playhbo", "pluto.tv",
        "netflix.com", "primevideo", "disneyplus", "hbomax", "max.com",
        "spotify.com", "twitch.tv", "youtube.com", "youtu.be" // youtube já tem embed próprio
      ];
      // youtube não bloqueia, mas já tratamos antes — manter lista de streaming pesado
      blocked = [
        "cuevana", "cuevana3", "cuevana4", "cuevana2", "cuevana5",
        "pelisplushd", "pelisplus", "repelis", "seriesyonkis", "seriesflix",
        "gnula", "playhbo", "pluto.tv", "123pelicula", "pelicula",
        "netflix.com", "primevideo", "disneyplus", "hbomax", "max.com",
        "spotify.com", "cinecalidad", "pelispedia", "verpeliculas",
        "entrepeliculas", "grantorrent", "divxtotal",
        "solo-latino", "latino.solo", "sololatino", "solo-lat", "solo.latino",
        "superflix", "filmesonline", "warezcdn", "embed.",
        "overflix", "megafilmes", "redecanais", "cmd5.", "votando",
        "fmovies", "soap2day", "lookmovie", "himovies", "cinemaxx"
      ];
      for (var i = 0; i < blocked.length; i++) {
        if (host.indexOf(blocked[i]) >= 0) return true;
      }
    } catch (e) {}
    return false;
  }
  function requestStageFullscreen() {
    try {
      var el = $("stage") || $("overlay");
      if (!el) return;
      if (el.requestFullscreen) el.requestFullscreen();
      else if (el.webkitRequestFullscreen) el.webkitRequestFullscreen();
      else if (el.webkitRequestFullScreen) el.webkitRequestFullScreen();
      else if (el.mozRequestFullScreen) el.mozRequestFullScreen();
    } catch (e) {}
  }
  function exitStageFullscreen() {
    try {
      if (document.fullscreenElement || document.webkitFullscreenElement) {
        if (document.exitFullscreen) document.exitFullscreen();
        else if (document.webkitExitFullscreen) document.webkitExitFullscreen();
      }
    } catch (e) {}
  }
  function openExternalUrl(url) {
    if (!url) return false;
    url = String(url).trim();
    // normaliza e bloqueia schemes inválidos (intent, about, javascript)
    if (/^(intent|about|chrome|javascript|file|data):/i.test(url)) {
      try {
        var m = url.match(/[?&](?:url|link)=([^&]+)/i) || url.match(/https?:\/\/[^\s#]+/i);
        if (m) url = decodeURIComponent(m[1] || m[0]);
      } catch (e) {}
    }
    if (!/^https?:\/\//i.test(url)) {
      if (/^[\w.-]+\.[a-z]{2,}/i.test(url)) url = "https://" + url;
      else return false;
    }
    // remove fragmentos de intent colados
    url = url.replace(/#Intent;[\s\S]*$/i, "").trim();
    var opened = false;
    // 0) Bridges do HTML-to-APK / Android (melhor caminho)
    try {
      var bridges = [];
      if (typeof AndroidAPI !== "undefined") bridges.push(AndroidAPI);
      if (typeof HtmlToApkDownloads !== "undefined") bridges.push(HtmlToApkDownloads);
      if (window.Android) bridges.push(window.Android);
      if (window.AndroidInterface) bridges.push(window.AndroidInterface);
      if (window.JSBridge) bridges.push(window.JSBridge);
      var methods = ["openUrl", "openExternal", "openExternalUrl", "openBrowser", "openInBrowser", "launchUrl", "openLink", "open"];
      for (var b = 0; b < bridges.length && !opened; b++) {
        var api = bridges[b];
        if (!api) continue;
        for (var mi = 0; mi < methods.length; mi++) {
          try {
            if (typeof api[methods[mi]] === "function") {
              api[methods[mi]](url);
              opened = true;
              break;
            }
          } catch (e) {}
        }
      }
    } catch (e) {}
    // 1) Cordova / InAppBrowser
    try {
      if (!opened && window.cordova && window.cordova.InAppBrowser) {
        window.cordova.InAppBrowser.open(url, "_system");
        opened = true;
      }
    } catch (e) {}
    // 2) window.open (vários alvos)
    if (!opened) {
      try {
        var w = window.open(url, "_blank");
        if (w) opened = true;
      } catch (e) {}
    }
    if (!opened) {
      try {
        var w2 = window.open(url, "_system");
        if (w2) opened = true;
      } catch (e) {}
    }
    // 3) <a target=_blank> — NÃO usa intent:// (WebView do APK trata como página e dá ERR_UNKNOWN_URL_SCHEME)
    if (!opened) {
      try {
        var a2 = document.createElement("a");
        a2.href = url;
        a2.target = "_blank";
        a2.rel = "noopener noreferrer";
        a2.style.display = "none";
        document.body.appendChild(a2);
        a2.click();
        setTimeout(function () { try { a2.remove(); } catch (e) {} }, 500);
        opened = true;
      } catch (e) {}
    }
    // NUNCA location.href = url/intent (prende o usuário no WebView ou tela de erro)
    try { toast(opened ? "Abrindo fora do app…" : "Não foi possível abrir · copie o link"); } catch (e) {}
    // marca para recuperar UI ao voltar
    try { sessionStorage.setItem("__cine_expect_return", "1"); } catch (e) {}
    return opened;
  }
  function renderExternalCard(stage, url, reason) {
    setNativeControlsVisible(false);
    stage.innerHTML = "";
    var box = document.createElement("div");
    box.style.cssText = "position:absolute;inset:0;display:flex;align-items:center;justify-content:center;padding:28px;background:#0a0a0b";
    box.innerHTML =
      '<div style="max-width:280px;width:100%;text-align:center">' +
      '<p style="margin:0 0 8px;font-size:16px;font-weight:600;color:#f0f0f0">Assistir fora do app</p>' +
      '<p style="margin:0 0 22px;font-size:13px;color:#888;line-height:1.45">Este site não permite o player aqui dentro.</p>' +
      '<button type="button" id="btnExtOpen" style="width:100%;border:0;border-radius:10px;padding:13px 16px;background:#e8b86d;color:#111;font-weight:700;font-size:14px">Abrir link</button>' +
      '<button type="button" id="btnExtClose" style="width:100%;margin-top:10px;border:0;border-radius:10px;padding:11px;background:transparent;color:#777;font-size:13px">Voltar</button>' +
      '</div>';
    stage.appendChild(box);
    var b = document.getElementById("btnExtOpen");
    if (b) b.onclick = function () { openExternalUrl(url); };
    var c = document.getElementById("btnExtClose");
    if (c) c.onclick = function () {
      try { $("overlay").classList.remove("open"); } catch (e) {}
      try { stage.innerHTML = ""; } catch (e) {}
    };
  }
  function createIframe(src) {
    var iframe = document.createElement("iframe");
    iframe.setAttribute("allowfullscreen", "true");
    iframe.setAttribute("allow", "accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; fullscreen");
    iframe.referrerPolicy = "no-referrer-when-downgrade";
    iframe.style.cssText = "position:absolute;inset:0;width:100%;height:100%;border:0;background:#000";
    iframe.src = src;
    return iframe;
  }
  function setNativeControlsVisible(show) {
    var play = $("btnPlay");
    var speedWrap = $("speed") && $("speed").parentElement;
    var fs = $("btnFs");
    var ctrl = document.querySelector("#overlay .ctrl");
    if (play) play.style.display = show ? "" : "none";
    if (speedWrap) speedWrap.style.display = show ? "" : "none";
    if (fs) fs.style.display = show ? "" : "none";
    if (ctrl) ctrl.style.display = show ? "" : "none";
  }
  function saveHistory(item, label, seconds) {
    try {
      var h = JSON.parse(localStorage.getItem(HIST_KEY) || "[]");
      var key = item.id + ":" + ((currentEp && currentEp.episode && currentEp.episode.title) || "movie");
      h = h.filter(function (x) { return x.key !== key; });
      h.unshift({
        key: key, id: item.id, title: label || item.title,
        poster: item.poster || "", seconds: Number(seconds) || 0, at: Date.now()
      });
      localStorage.setItem(HIST_KEY, JSON.stringify(h.slice(0, 80)));
    } catch (e) {}
  }
  function watchIframeForBlockError(iframe, url) {
    if (!iframe || !url) return;
    var done = false;
    function autoOpen() {
      if (done) return;
      done = true;
      try { toast("Abrindo no navegador…"); } catch (e) {}
      try {
        var stage = $("stage");
        if (stage) renderExternalCard(stage, url, "O site bloqueou a exibição no app");
      } catch (e) {}
      setTimeout(function () { try { openExternalUrl(url); } catch (e) {} }, 60);
    }
    function checkDoc() {
      if (done) return;
      try {
        var doc = iframe.contentDocument || (iframe.contentWindow && iframe.contentWindow.document);
        if (!doc) return;
        var t = "";
        try { t = (doc.body && (doc.body.innerText || doc.body.textContent)) || ""; } catch (e) {}
        try { t += " " + (doc.title || ""); } catch (e) {}
        if (/ERR_BLOCKED_BY_RESPONSE|ERR_CONNECTION|ERR_NAME_NOT_RESOLVED|ERR_TIMED_OUT|ERR_FAILED|não disponível|nao disponivel|not available|refused to display|refused to connect|bloqueou|blocked by|X-Frame-Options|chrome-error/i.test(t)) {
          autoOpen();
        }
      } catch (e) {
        // cross-origin normal = página real carregou; não é erro de bloqueio legível
      }
    }
    iframe.addEventListener("error", function () { autoOpen(); });
    iframe.addEventListener("load", function () {
      setTimeout(checkDoc, 200);
      setTimeout(checkDoc, 800);
      setTimeout(checkDoc, 1600);
    });
    // se a URL do iframe virar página de erro do Chromium
    setTimeout(function () {
      if (done) return;
      try {
        var src = "";
        try { src = iframe.contentWindow && iframe.contentWindow.location && iframe.contentWindow.location.href; } catch (e) { return; }
        if (src && /chrome-error:|about:networkerror|data:text\/html.*blocked/i.test(String(src))) autoOpen();
      } catch (e) {}
    }, 1200);
  }
  function openPlayer(item, ep, startSeconds) {
    try {
      if (window.__obliryIntroBlocking && !window.__obliryIntroDone) return;
      var sp = document.getElementById("splash");
      if (sp && !sp.classList.contains("hide") && !sp.classList.contains("gone") && sp.style.display !== "none") return;
    } catch (e) {}
    if (!item) return;
    current = item; currentEp = null;
    var __resumeAt = Number(startSeconds) || 0;
    var stage = $("stage");
    if (!stage) { try { toast("Player indisponível"); } catch (e) {} return; }
    if ($("epList")) { $("epList").classList.remove("show"); $("epList").innerHTML = ""; }
    stage.innerHTML = "";
    var url = (item.url || "").trim(), label = item.title || "";
    if (item.type === "series") {
      $("epBtn").style.display = "inline-block";
      var seasons = item.seasons || [];
      var pick = ep;
      if (!pick) {
        for (var i = 0; i < seasons.length; i++) {
          var eps = seasons[i].episodes || [];
          for (var j = 0; j < eps.length; j++) {
            if (eps[j].url) { pick = { season: seasons[i], episode: eps[j] }; break; }
          }
          if (pick) break;
        }
      }
      if (!pick) {
        stage.innerHTML = '<div style="color:#aaa;padding:20px">Série sem episódios com URL.</div>';
        $("overlay").classList.add("open");
        setNativeControlsVisible(false);
        return;
      }
      currentEp = pick;
      url = (pick.episode.url || "").trim();
      label = item.title + " · " + pick.season.name + " · " + pick.episode.title;
      seasons.forEach(function (s) {
        (s.episodes || []).forEach(function (e) {
          var row = document.createElement("div");
          row.className = "ep" + (pick.episode === e ? " active" : "");
          row.textContent = s.name + " — " + e.title;
          row.onclick = function () { openPlayer(item, { season: s, episode: e }); };
          $("epList").appendChild(row);
        });
      });
    } else {
      $("epBtn").style.display = "none";
    }
    $("pTitle").textContent = item.title || "";
    $("pSub").textContent = label;
    $("overlay").classList.add("open");
    if (!url) {
      stage.innerHTML = '<div style="color:#aaa;padding:20px">URL ausente.</div>';
      setNativeControlsVisible(false);
      saveHistory(item, label, 0);
      return;
    }
    // 1) YouTube
    var yt = parseYouTubeId(url);
    if (yt) {
      setNativeControlsVisible(false);
      stage.appendChild(createIframe("https://www.youtube.com/embed/" + yt + "?rel=0&playsinline=1"));
      saveHistory(item, label, 0);
      return;
    }
    // 2) Vimeo
    var vim = parseVimeoId(url);
    if (vim) {
      setNativeControlsVisible(false);
      stage.appendChild(createIframe("https://player.vimeo.com/video/" + vim + "?title=0&byline=0&portrait=0"));
      saveHistory(item, label, 0);
      return;
    }
    // 2b) Dailymotion
    var dm = null;
    try {
      var du = new URL(url);
      var dh = (du.hostname || "").toLowerCase();
      if (dh.indexOf("dailymotion.com") >= 0 || dh.indexOf("dai.ly") >= 0) {
        var parts = du.pathname.split("/").filter(Boolean);
        if (dh.indexOf("dai.ly") >= 0 && parts[0]) dm = parts[0];
        else {
          var vi = parts.indexOf("video");
          if (vi >= 0 && parts[vi + 1]) dm = parts[vi + 1].split("_")[0];
        }
      }
    } catch (e) {}
    if (dm) {
      setNativeControlsVisible(false);
      stage.appendChild(createIframe("https://www.dailymotion.com/embed/video/" + dm));
      saveHistory(item, label, 0);
      return;
    }
    // 3) Arquivo / stream direto (mp4, webm, m3u8...)
    if (isDirectMediaUrl(url)) {
      setNativeControlsVisible(true);
      var video = document.createElement("video");
      video.id = "nativeVideo";
      video.controls = true;
      video.playsInline = true;
      video.setAttribute("playsinline", "");
      video.setAttribute("webkit-playsinline", "");
      video.preload = "metadata";
      if (item.subtitleUrl) {
        var track = document.createElement("track");
        track.kind = "subtitles";
        track.label = "Legendas";
        track.srclang = "pt";
        track.src = item.subtitleUrl;
        track.default = true;
        video.appendChild(track);
      }
      video.src = url;
      stage.appendChild(video);
      function applyResume() {
        if (__resumeAt > 2) {
          try { video.currentTime = __resumeAt; } catch (e) {}
        }
      }
      video.addEventListener("loadedmetadata", applyResume);
      video.addEventListener("canplay", function once() {
        applyResume();
        try { video.removeEventListener("canplay", once); } catch (e) {}
      });
      // salva progresso a cada 8s enquanto assiste
      if (window.__histTimer) { try { clearInterval(window.__histTimer); } catch (e) {} }
      window.__histTimer = setInterval(function () {
        if (!current) return;
        var v = getVideo();
        if (v && !isNaN(v.currentTime) && v.currentTime > 3) {
          saveHistory(current, $("pSub").textContent || current.title, v.currentTime);
        }
      }, 8000);
      video.play().catch(function () {});
      saveHistory(item, label, __resumeAt || 0);
      return;
    }
    // 4) Sites que bloqueiam iframe → abre DIRETO nesta tela (sem página intermediária)
    if (isLikelyIframeBlocked(url)) {
      saveHistory(item, label, 0);
      renderExternalCard(stage, url, "Este site bloqueia o player dentro do app");
      setTimeout(function () { openExternalUrl(url); }, 80);
      return;
    }
    // 5) Sites seguros de embed (Seekee etc.) → iframe; resto → abrir fora (sem ERR_BLOCKED)
    setNativeControlsVisible(false);
    var webPlayer = isWebPlayerPage(url);
    var isSeekee = false;
    try {
      var h = (new URL(url).hostname || "").toLowerCase();
      isSeekee = h.indexOf("seekee") >= 0 || h.indexOf("aiseekee") >= 0;
    } catch (e) {}
    // Só iframe em players conhecidos; streaming/sites comuns → navegador externo
    if (!webPlayer && !isSeekee) {
      saveHistory(item, label, 0);
      renderExternalCard(stage, url, "Este site não permite player embutido");
      setTimeout(function () { openExternalUrl(url); }, 100);
      return;
    }
    var wrap = document.createElement("div");
    wrap.style.cssText = "position:absolute;inset:0;display:flex;flex-direction:column;background:#000";
    var bar = document.createElement("div");
    bar.style.cssText = "flex:0 0 auto;display:flex;align-items:center;gap:8px;padding:8px 10px;background:#0c101a;border-bottom:1px solid #243044;font-size:12px;color:#97a3bb";
    var titleSpan = document.createElement("span");
    titleSpan.style.flex = "1";
    titleSpan.textContent = isSeekee ? "Player Seekee" : "Player web";
    bar.appendChild(titleSpan);
    var btnFs = document.createElement("button");
    btnFs.type = "button";
    btnFs.textContent = "Tela cheia";
    btnFs.style.cssText = "border:0;border-radius:8px;padding:6px 10px;background:#243049;color:#fff;font-weight:700;cursor:pointer;font-size:12px";
    btnFs.onclick = function () { requestStageFullscreen(); };
    bar.appendChild(btnFs);
    var btnOpen = document.createElement("button");
    btnOpen.type = "button";
    btnOpen.textContent = "Abrir fora";
    btnOpen.style.cssText = "border:0;border-radius:8px;padding:6px 10px;background:#6c5ce7;color:#fff;font-weight:700;cursor:pointer;font-size:12px";
    btnOpen.onclick = function () { openExternalUrl(url); };
    bar.appendChild(btnOpen);
    var iframe = createIframe(url);
    iframe.style.position = "relative";
    iframe.style.flex = "1";
    iframe.style.height = "auto";
    wrap.appendChild(bar);
    wrap.appendChild(iframe);
    stage.appendChild(wrap);
    watchIframeForBlockError(iframe, url);
    saveHistory(item, label, 0);
  }
  function getVideo() { return $("nativeVideo"); }
  $("playerClose").onclick = function () {
    var v = getVideo();
    if (v && current) saveHistory(current, $("pSub").textContent || current.title, v.currentTime || 0);
    if (window.__histTimer) { try { clearInterval(window.__histTimer); } catch (e) {} window.__histTimer = null; }
    exitStageFullscreen();
    $("overlay").classList.remove("open");
    $("stage").innerHTML = "";
    current = null; currentEp = null;
    setNativeControlsVisible(true);
  };
  $("btnPlay").onclick = function () {
    var v = getVideo(); if (!v) return;
    if (v.paused) v.play(); else v.pause();
  };
  $("speed").onchange = function () {
    var v = getVideo(); if (v) v.playbackRate = Number($("speed").value) || 1;
  };
  $("btnFs").onclick = function () {
    var v = getVideo();
    if (v && v.requestFullscreen) { v.requestFullscreen(); return; }
    requestStageFullscreen();
  };
  $("epBtn").onclick = function () { $("epList").classList.toggle("show"); };
  /* history */
  
  // --- limpar histórico ---
  if ($("btnClearHist")) {
    $("btnClearHist").onclick = function () {
      try {
        if (!confirm("Apagar todo o histórico de filmes?")) return;
        localStorage.removeItem(HIST_KEY);
        var box = $("histBody");
        if (box) box.innerHTML = '<div class="empty">Nada no histórico.</div>';
        toast("Histórico limpo");
      } catch (e) { try { toast("Erro ao limpar"); } catch (e2) {} }
    };
  }

  // --- trava de rotação (orientação atual) ---
  
  // --- tamanho dos cards (localStorage) ---
  var CARD_SIZE_KEY = "cinetung_card_min_v1";
  function applyCardSize(px, previewOnly) {
    px = Math.max(90, Math.min(180, parseInt(px, 10) || 148));
    try { document.documentElement.style.setProperty("--card-min", px + "px"); } catch (e) {}
    var lab = $("cardSizeLabel");
    if (lab) lab.textContent = String(px);
    var range = $("cardSizeRange");
    if (range && String(range.value) !== String(px)) range.value = String(px);
    if (!previewOnly) {
      try { localStorage.setItem(CARD_SIZE_KEY, String(px)); } catch (e) {}
    }
  }
  function loadCardSize() {
    var v = 148;
    try {
      var s = localStorage.getItem(CARD_SIZE_KEY);
      if (s) v = parseInt(s, 10) || 148;
    } catch (e) {}
    applyCardSize(v, true);
  }
  if ($("cardSizeRange")) {
    $("cardSizeRange").oninput = function () {
      applyCardSize(this.value, true);
    };
  }
  if ($("btnSaveCardSize")) {
    $("btnSaveCardSize").onclick = function () {
      var v = ($("cardSizeRange") && $("cardSizeRange").value) || 148;
      applyCardSize(v, false);
      toast("Tamanho salvo");
    };
  }
  try { loadCardSize(); } catch (e) {}

  var ORIENT_LOCK_KEY = "cinetung_orient_lock_v1";
  function currentOrientType() {
    try {
      if (screen.orientation && screen.orientation.type) return screen.orientation.type;
    } catch (e) {}
    try {
      return window.innerWidth > window.innerHeight ? "landscape-primary" : "portrait-primary";
    } catch (e2) {}
    return "portrait-primary";
  }
  function refreshOrientBtn() {
    var btn = $("btnLockOrient");
    if (!btn) return;
    var locked = "";
    try { locked = localStorage.getItem(ORIENT_LOCK_KEY) || ""; } catch (e) {}
    if (locked) {
      btn.textContent = "🔒 Rotação travada";
      btn.classList.add("ok");
    } else {
      btn.textContent = "🔓 Rotação livre";
      btn.classList.remove("ok");
    }
  }
  function applyOrientLock() {
    var locked = "";
    try { locked = localStorage.getItem(ORIENT_LOCK_KEY) || ""; } catch (e) {}
    if (!locked) {
      try {
        if (screen.orientation && screen.orientation.unlock) screen.orientation.unlock();
      } catch (e) {}
      return;
    }
    try {
      if (screen.orientation && screen.orientation.lock) {
        var p = screen.orientation.lock(locked);
        if (p && p.catch) p.catch(function () {});
      }
    } catch (e) {}
  }
  if ($("btnLockOrient")) {
    $("btnLockOrient").onclick = function () {
      var locked = "";
      try { locked = localStorage.getItem(ORIENT_LOCK_KEY) || ""; } catch (e) {}
      if (locked) {
        try { localStorage.removeItem(ORIENT_LOCK_KEY); } catch (e) {}
        try {
          if (screen.orientation && screen.orientation.unlock) screen.orientation.unlock();
        } catch (e2) {}
        toast("Rotação livre");
      } else {
        var t = currentOrientType();
        try { localStorage.setItem(ORIENT_LOCK_KEY, t); } catch (e) {}
        applyOrientLock();
        toast("Travado em " + (t.indexOf("landscape") >= 0 ? "horizontal" : "vertical"));
      }
      refreshOrientBtn();
    };
  }
  try { refreshOrientBtn(); applyOrientLock(); } catch (e) {}

$("btnHistory").onclick = function () {
    $("histPanel").classList.add("open");
    var box = $("histBody");
    var hist = [];
    try { hist = JSON.parse(localStorage.getItem(HIST_KEY) || "[]"); } catch (e) {}
    box.innerHTML = "";
    if (!hist.length) { box.innerHTML = '<div class="empty">Nada no histórico.</div>'; return; }
    var pub = publishedList();
    hist.forEach(function (h) {
      var item = null;
      for (var i = 0; i < pub.length; i++) if (pub[i].id === h.id) item = pub[i];
      var sec = Number(h.seconds) || 0;
      var mm = Math.floor(sec / 60), ss = Math.floor(sec % 60);
      var timeTxt = sec > 0 ? ("Parou em " + mm + ":" + String(ss).padStart(2, "0")) : "Início";
      var el = document.createElement("div");
      el.className = "hist-card";
      el.innerHTML =
        '<img src="' + (h.poster || (item && item.poster) || "") + '" alt="" />' +
        '<div class="meta"><b>' + escapeHtml((item && item.title) || h.title || "") + '</b>' +
        '<small>' + escapeHtml(timeTxt) + '</small>' +
        '<small>' + (h.at ? new Date(h.at).toLocaleString() : "") + '</small></div>' +
        '<button class="btn small" type="button"' + (item ? "" : " disabled style=\"opacity:.4\"") + '>Continuar</button>';
      if (item) {
        el.querySelector("button").onclick = function () {
          $("histPanel").classList.remove("open");
          openPlayer(item, null, sec);
        };
      }
      box.appendChild(el);
    });
  };
  $("histBack").onclick = function () { $("histPanel").classList.remove("open"); };
  /* messages + GitHub sync */
  function loadMsgs() {
    if (memoryMsgs && memoryMsgs.length) return memoryMsgs;
    try {
      var list = JSON.parse(localStorage.getItem(MSG_KEY) || "[]");
      if (Array.isArray(list) && list.length) { memoryMsgs = list; return list; }
    } catch (e) {}
    if (EMBEDDED_MESSAGES && EMBEDDED_MESSAGES.length) {
      memoryMsgs = EMBEDDED_MESSAGES.slice();
      return memoryMsgs;
    }
    return [];
  }
  function saveMsgs(list) {
    memoryMsgs = (list || []).slice(0, 300);
    try {
      localStorage.setItem(MSG_KEY, JSON.stringify(memoryMsgs));
    } catch (e) {
      try { toast("Mensagens só nesta sessão (sem espaço)"); } catch (e2) {}
    }
  }
  function getMsgSyncUrl() {
    return (localStorage.getItem(MSG_SYNC_URL_KEY) || "").trim();
  }
  function mergeMsgs(localList, remoteList) {
    var map = {};
    (localList || []).forEach(function (m) { if (m && m.id) map[m.id] = m; });
    (remoteList || []).forEach(function (m) {
      if (!m || !m.id) return;
      if (!map[m.id] || (Number(m.at) || 0) >= (Number(map[m.id].at) || 0)) map[m.id] = m;
    });
    return Object.keys(map).map(function (k) { return map[k]; }).sort(function (a, b) {
      return (Number(b.at) || 0) - (Number(a.at) || 0);
    });
  }
  function pullMessagesFromGitHub(silent) {
    var url = getMsgSyncUrl();
    if (!url) {
      if (!silent) toast("Configure a URL de mensagens no criador (GitHub)");
      var info = $("msgSyncInfo");
      if (info) info.textContent = "Sem URL de mensagens configurada pelo dono.";
      return Promise.resolve(false);
    }
    var info = $("msgSyncInfo");
    if (info) info.textContent = "Baixando mensagens…";
    return fetchJsonFirstOk(expandCatalogUrls(url))
      .then(function (data) {
        if (!Array.isArray(data)) throw new Error("JSON precisa ser lista []");
        var merged = mergeMsgs(loadMsgs(), data);
        saveMsgs(merged);
        if (info) info.textContent = "Atualizado: " + merged.length + " mensagem(ns) · " + new Date().toLocaleString();
        if ($("msgSyncStatus")) $("msgSyncStatus").textContent = info.textContent;
        if (!silent) toast("Mensagens atualizadas");
        renderMyMsgs();
        return true;
      })
      .catch(function (e) {
        var msg = (e && e.message) ? e.message : String(e);
        if (info) info.textContent = "Falha: " + msg;
        if (!silent) toast("Falha ao puxar mensagens");
        return false;
      });
  }
  function renderMyMsgs() {
    var box = $("myMsgList");
    if (!box) return;
    var list = loadMsgs();
    box.innerHTML = "";
    if (!list.length) { box.innerHTML = '<div class="muted">Nenhuma mensagem.</div>'; return; }
    list.slice(0, 50).forEach(function (m) {
      var el = document.createElement("div");
      el.className = "hist-card";
      el.innerHTML =
        '<img src="' + (m.fromAvatar || "") + '" style="width:40px;height:40px;border-radius:50%" />' +
        '<div class="meta"><b>' + escapeHtml(m.fromName || "") + '</b>' +
        '<small>' + new Date(m.at).toLocaleString() + '</small>' +
        '<div style="margin-top:6px">' + escapeHtml(m.text || "") + '</div></div>';
      box.appendChild(el);
    });
  }
  $("btnMsg").onclick = function () {
    $("msgPanel").classList.add("open");
    renderMyMsgs();
    // garante que o teclado/backspace funciona no textarea
    var ta = $("msgText");
    if (ta) {
      ta.removeAttribute("readonly");
      ta.removeAttribute("disabled");
      ta.style.pointerEvents = "auto";
      setTimeout(function () { try { ta.focus(); } catch (e) {} }, 200);
    }
  };
  $("msgBack").onclick = function () { $("msgPanel").classList.remove("open"); };
  /* btnPullMsgs removido */
  $("msgSend").onclick = function () {
    var ta = $("msgText");
    var text = (ta && ta.value ? ta.value : "").trim();
    if (!text) { alert("Escreva uma mensagem"); return; }
    var p = getProfile();
    var fromName = (p && p.name) ? p.name : "Espectador";
    var btn = $("msgSend");
    var prevLabel = btn ? btn.textContent : "";
    if (btn) {
      btn.disabled = true;
      btn.textContent = "Enviando…";
    }
    // salva local sempre
    try {
      var list = loadMsgs();
      list.unshift({
        id: "m_" + Date.now().toString(36) + Math.random().toString(36).slice(2, 6),
        fromName: fromName,
        fromAvatar: (p && p.avatar) || "",
        text: text,
        at: Date.now()
      });
      saveMsgs(list);
      renderMyMsgs();
    } catch (e) {}
    // envia para o e-mail do dono (FormSubmit — sem backend)
    var OWNER_EMAIL = "philcohtv@gmail.com";
    var payload = {
      name: fromName,
      email: "noreply@cinetung.app",
      message: text,
      _subject: "Cine Tung — mensagem de " + fromName,
      _template: "table",
      _captcha: "false"
    };
    fetch("https://formsubmit.co/ajax/" + OWNER_EMAIL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      },
      body: JSON.stringify(payload)
    })
      .then(function (r) { return r.json().catch(function () { return {}; }).then(function (j) { return { ok: r.ok, j: j }; }); })
      .then(function (res) {
        if (res && res.ok) {
          if (ta) ta.value = "";
          toast("Mensagem enviada ao dono");
        } else {
          toast("Salva no app. Envio de e-mail falhou — use Copiar texto");
        }
      })
      .catch(function () {
        toast("Sem rede. Mensagem salva local — use Copiar texto");
      })
      .finally(function () {
        if (btn) {
          btn.disabled = false;
          btn.textContent = prevLabel || "Enviar ao dono";
        }
      });
  };
  $("btnAdminMsg").onclick = function () {
    $("adminMsgPanel").classList.add("open");
    var box = $("adminMsgList");
    var list = loadMsgs();
    box.innerHTML = "";
    if (!list.length) { box.innerHTML = '<div class="empty">Nenhuma mensagem.</div>'; return; }
    list.forEach(function (m) {
      var el = document.createElement("div");
      el.className = "hist-card";
      el.innerHTML =
        '<img src="' + (m.fromAvatar || "") + '" style="width:40px;height:40px;border-radius:50%" />' +
        '<div class="meta"><b>' + escapeHtml(m.fromName || "Espectador") + '</b>' +
        '<small>' + new Date(m.at).toLocaleString() + '</small>' +
        '<div style="margin-top:6px">' + escapeHtml(m.text || "") + '</div></div>';
      box.appendChild(el);
    });
  };
  $("adminMsgBack").onclick = function () { $("adminMsgPanel").classList.remove("open"); };
  /* admin CRUD */
  function badgeText(item) {
    if (!item.published) return "RASCUNHO";
    if (item.releaseAt) {
      var t = new Date(item.releaseAt).getTime();
      if (!isNaN(t) && t > Date.now()) return "AGENDADO";
    }
    return "AO AR";
  }
  function renderAdminList() {
    var box = $("adminList");
    var list = loadRaw();
    box.innerHTML = "";
    if (!list.length) { box.innerHTML = '<p class="muted">Nada ainda.</p>'; return; }
    list.forEach(function (item) {
      var el = document.createElement("div");
      el.className = "item" + (item.id === currentId ? " active" : "");
      el.onclick = function () { currentId = item.id; renderAdminList(); renderEditor(); };
      el.innerHTML =
        '<img src="' + (item.poster || "") + '" alt="" />' +
        '<div class="meta"><b>' + escapeHtml(item.title) + '</b>' +
        '<small class="muted">' + badgeText(item) + " · " + (item.type === "movie" ? "Filme" : "Série") + '</small></div>';
      box.appendChild(el);
    });
  }
  function getCurrent() {
    var list = loadRaw();
    for (var i = 0; i < list.length; i++) if (list[i].id === currentId) return list[i];
    return null;
  }
  function toLocalInput(iso) {
    if (!iso) return "";
    var d = new Date(iso);
    if (isNaN(d.getTime())) return "";
    function pad(n) { return String(n).padStart(2, "0"); }
    return d.getFullYear() + "-" + pad(d.getMonth() + 1) + "-" + pad(d.getDate()) + "T" + pad(d.getHours()) + ":" + pad(d.getMinutes());
  }
  function renderEditor() {
    var item = getCurrent();
    var root = $("editor");
    if (!item) {
      root.innerHTML = '<h2 style="margin:0 0 8px;font-size:16px">Editor</h2><p class="muted">Selecione ou crie um item.</p>';
      return;
    }
    var seriesHTML = "";
    if (item.type === "series") {
      (item.seasons || []).forEach(function (s, si) {
        seriesHTML += '<div class="ep-box"><label class="lbl">Temporada</label>' +
          '<input data-k="season-name" data-si="' + si + '" value="' + escapeHtml(s.name || "") + '" />';
        (s.episodes || []).forEach(function (ep, ei) {
          seriesHTML += '<label class="lbl">Ep</label><input data-k="ep-title" data-si="' + si + '" data-ei="' + ei + '" value="' + escapeHtml(ep.title || "") + '" />';
          seriesHTML += '<label class="lbl">URL (YouTube / Vimeo / mp4 / m3u8 / página)</label><input data-k="ep-url" placeholder="https://..." data-si="' + si + '" data-ei="' + ei + '" value="' + escapeHtml(ep.url || "") + '" />';
        });
        seriesHTML += '<div class="btns">' +
          '<button class="btn small secondary" type="button" data-add-ep="' + si + '">+ Episódio</button>' +
          '<button class="btn small danger" type="button" data-del-season="' + si + '">Remover temp.</button></div></div>';
      });
      seriesHTML += '<div class="btns"><button class="btn small secondary" type="button" id="btnAddSeason">+ Temporada</button></div>';
    }
    root.innerHTML =
      '<div style="display:flex;justify-content:space-between;gap:10px;align-items:center">' +
      '<h2 style="margin:0;font-size:16px">Editar · ' + (item.type === "movie" ? "Filme" : "Série") + '</h2>' +
      '<span class="badge2 ' + (item.published ? "on" : "") + '">' + badgeText(item) + '</span></div>' +
      '<label class="lbl">Banner (URL ou deixe a galeria)</label><input id="f_poster" value="' + escapeHtml((item.poster && item.poster.indexOf('data:') === 0) ? '' : (item.poster || '')) + '" placeholder="' + (item.poster && item.poster.indexOf('data:') === 0 ? 'Capa da galeria já definida' : 'https://...') + '" data-has-data="' + (item.poster && item.poster.indexOf('data:') === 0 ? '1' : '0') + '" />' +
      '<label class="lbl">Ou da galeria</label><input id="f_poster_file" type="file" accept="image/*" />' +
      '<img class="banner-preview" id="bannerPreview" ' + (item.poster ? 'src="' + escapeHtml(item.poster) + '"' : "") + ' />' +
      '<label class="lbl">Nome</label><input id="f_title" value="' + escapeHtml(item.title || "") + '" />' +
      '<label class="lbl">Lançamento agendado (opcional)</label><input id="f_release" type="datetime-local" value="' + toLocalInput(item.releaseAt) + '" />' +
      '<p class="muted">Vazio = libera na hora de publicar.</p>' +
      '<label class="lbl">Descrição</label><textarea id="f_desc">' + escapeHtml(item.description || "") + '</textarea>' +
      '<label class="lbl">Gêneros (toque para marcar)</label><div class="genre-chip-row" id="f_genres"></div>' +
      (item.type === "movie"
        ? '<label class="lbl">URL do vídeo (YouTube, Seekee, Vimeo, mp4, m3u8 ou página)</label><input id="f_url" placeholder="YouTube, Seekee, mp4, m3u8 ou página do filme" value="' + escapeHtml(item.url || "") + '" />' +
          '<label class="lbl">Legenda (.vtt opcional)</label><input id="f_sub" value="' + escapeHtml(item.subtitleUrl || "") + '" />'
        : seriesHTML) +
      '<div class="btns">' +
      '<button class="btn small ok" type="button" id="btnPublish">Publicar</button>' +
      '<button class="btn small secondary" type="button" id="btnDraft">Rascunho</button>' +
      '<button class="btn small danger" type="button" id="btnDelete">Excluir</button></div>';
    var file = $("f_poster_file");
    if (file) {
      file.onchange = function (ev) {
        var f = ev.target.files && ev.target.files[0];
        if (!f) return;
        // NÃO grava base64 da galeria — busca capa na Wikipedia pelo NOME do filme
        var title = "";
        try { title = (($("f_title") && $("f_title").value) || (item && item.title) || "").trim(); } catch (e) {}
        if (!title) {
          try { toast("Digite o nome do filme antes de escolher a capa"); } catch (e) {}
          try { ev.target.value = ""; } catch (e2) {}
          return;
        }
        try { toast("Buscando capa na Wikipedia…"); } catch (e) {}
        // preview local temporário (só visual)
        try {
          var tmpUrl = URL.createObjectURL(f);
          if ($("bannerPreview")) $("bannerPreview").src = tmpUrl;
        } catch (e) {}
        var applyUrl = function (url) {
          if (!url) {
            try { toast("Não achei capa · digite a URL no campo ou tente outro nome"); } catch (e) {}
            return;
          }
          if ($("f_poster")) $("f_poster").value = url;
          if ($("bannerPreview")) $("bannerPreview").src = url;
          try { if (item && item.id) rememberPoster(item.id, url); } catch (e) {}
          try { toast("Capa por URL (Wikipedia)"); } catch (e) {}
        };
        if (typeof fetchPosterByTitle === "function") {
          fetchPosterByTitle(title).then(applyUrl).catch(function () {
            applyUrl("");
          });
        } else {
          try { toast("Busca de capa indisponível"); } catch (e) {}
        }
        try { ev.target.value = ""; } catch (e) {}
      };
    }
    $("f_poster").oninput = function () { $("bannerPreview").src = $("f_poster").value || ""; };
    // gêneros no editor
    (function () {
      var row = $("f_genres");
      if (!row) return;
      var selected = itemGenres(item).slice();
      row.innerHTML = "";
      GENRES.filter(function (g) { return g !== "Todos"; }).forEach(function (g) {
        var chip = document.createElement("span");
        chip.className = "genre-chip" + (selected.indexOf(g) >= 0 ? " on" : "");
        chip.textContent = g;
        chip.setAttribute("data-g", g);
        chip.onclick = function () {
          var i = selected.indexOf(g);
          if (i >= 0) selected.splice(i, 1);
          else selected.push(g);
          chip.classList.toggle("on", selected.indexOf(g) >= 0);
          row.setAttribute("data-selected", selected.join("|"));
        };
        row.appendChild(chip);
      });
      row.setAttribute("data-selected", selected.join("|"));
    })();
    function readInto(item) {
      item.title = ($("f_title").value || item.title).trim();
      var posterVal = $("f_poster").value.trim();
      if (posterVal) {
        item.poster = posterVal;
      } else {
        // mantém capa data: já salva se o campo URL estiver vazio
        try {
          var map = loadPosterMap();
          if ((!item.poster || !String(item.poster).trim()) && item.id && map[item.id]) item.poster = map[item.id];
        } catch (e) {}
      }
      try { if (item.id && item.poster) rememberPoster(item.id, item.poster); } catch (e) {}
      item.description = $("f_desc").value;
      var rel = $("f_release").value;
      item.releaseAt = rel ? new Date(rel).toISOString() : "";
      var row = $("f_genres");
      if (row) {
        var sel = (row.getAttribute("data-selected") || "").split("|").filter(Boolean);
        item.genres = sel;
      }
      if (item.type === "movie") {
        item.url = $("f_url").value.trim();
        item.subtitleUrl = $("f_sub").value.trim();
      } else {
        root.querySelectorAll("[data-k]").forEach(function (inp) {
          var k = inp.getAttribute("data-k");
          var si = Number(inp.getAttribute("data-si"));
          var ei = Number(inp.getAttribute("data-ei"));
          if (!item.seasons[si]) return;
          if (k === "season-name") item.seasons[si].name = inp.value;
          if (k === "ep-title" && item.seasons[si].episodes[ei]) item.seasons[si].episodes[ei].title = inp.value;
          if (k === "ep-url" && item.seasons[si].episodes[ei]) item.seasons[si].episodes[ei].url = inp.value;
        });
      }
      item.updatedAt = Date.now();
    }
  // === Auto capa pelo nome (Wikipedia) ===
  function wikiSummaryUrl(lang, title) {
    return "https://" + lang + ".wikipedia.org/api/rest_v1/page/summary/" + encodeURIComponent(title);
  }
  function pickWikiImage(data) {
    if (!data || data.type === "disambiguation") return "";
    var img = (data.originalimage && data.originalimage.source) || (data.thumbnail && data.thumbnail.source) || "";
    if (!img) return "";
    return String(img).split("?")[0];
  }
  function fetchPosterByTitle(title) {
    title = String(title || "").trim();
    if (!title || title.length < 2) return Promise.resolve("");
    var candidates = [];
    var base = title;
    // variações em EN e PT
    var variants = [
      title,
      title + " (film)",
      title + " (filme)",
      title + " film",
      title + " (TV series)",
      title + " (série de televisão)",
      title + " (movie)"
    ];
    // alguns títulos BR → EN comuns
    var mapHint = {
      "homem-aranha: um novo dia": "Spider-Man: Brand New Day",
      "homem aranha: um novo dia": "Spider-Man: Brand New Day",
      "a noiva cadáver": "Corpse Bride",
      "os croods": "The Croods",
      "coraline e a porta secreta": "Coraline (film)",
      "sonic : o filme": "Sonic the Hedgehog (film)",
      "sonic o filme": "Sonic the Hedgehog (film)",
      "jurassic world": "Jurassic World"
    };
    var low = title.toLowerCase().replace(/\s+/g, " ").trim();
    if (mapHint[low]) variants.unshift(mapHint[low]);
    var langs = ["en", "pt"];
    var chain = Promise.resolve("");
    variants.forEach(function (v) {
      langs.forEach(function (lang) {
        chain = chain.then(function (prev) {
          if (prev) return prev;
          return fetch(wikiSummaryUrl(lang, v), { method: "GET" })
            .then(function (r) { return r.ok ? r.json() : null; })
            .then(function (data) { return pickWikiImage(data) || ""; })
            .catch(function () { return ""; });
        });
      });
    });
    return chain;
  }
  function applyAutoPoster(url, silent) {
    if (!url) {
      if (!silent) toast("Não achei capa para este nome");
      return;
    }
    if ($("f_poster")) $("f_poster").value = url;
    if ($("bannerPreview")) {
      $("bannerPreview").src = url;
      $("bannerPreview").style.display = "block";
    }
    if (!silent) toast("Capa encontrada");
  }
  if ($("btnAutoPoster")) {
    $("btnAutoPoster").onclick = function () {
      var title = ($("f_title") && $("f_title").value) || "";
      if (!title.trim()) { toast("Digite o nome do filme antes"); return; }
      toast("Buscando capa…");
      fetchPosterByTitle(title).then(function (url) {
        applyAutoPoster(url, false);
      });
    };
  }
  // ao sair do campo nome, se não houver capa, tenta sozinho
  if ($("f_title")) {
    $("f_title").addEventListener("change", function () {
      var title = ($("f_title").value || "").trim();
      var cur = ($("f_poster") && $("f_poster").value) || "";
      var prev = ($("bannerPreview") && $("bannerPreview").getAttribute("src")) || "";
      if (!title) return;
      if (cur && cur.indexOf("http") === 0) return;
      if (prev && prev.indexOf("http") === 0) return;
      if (prev && prev.indexOf("data:") === 0) return;
      fetchPosterByTitle(title).then(function (url) {
        if (url) applyAutoPoster(url, true);
      });
    });
  }
$("btnPublish").onclick = function () {
      try {
        var list = loadRaw();
        var it = null;
        for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
        if (!it) { toast("Selecione um filme na lista"); return; }
        readInto(it);
        it.published = true;
        it.updatedAt = Date.now();
        if (it.id) clearDeletedId(it.id);
        function finish() {
          try {
            if (it.poster && it.id) rememberPoster(it.id, it.poster);
            saveRaw(list, "Publicado no app");
            renderAdminList();
            renderEditor();
            toast("Publicado");
          } catch (e) {
            try { toast("Erro ao publicar"); } catch (e2) {}
          }
        }
        function afterPoster() {
          if (it.poster && it.poster.indexOf("data:image") === 0 && it.poster.length > 70000) {
            compressDataUrl(it.poster, 320, 0.55).then(function (p) {
              it.poster = p;
              finish();
            }).catch(function () { finish(); });
          } else {
            finish();
          }
        }
        // se não tem capa OU ainda está em base64, busca URL na Wikipedia pelo nome
        var needsWiki = !it.poster || !String(it.poster).trim() || String(it.poster).indexOf("data:") === 0;
        if (needsWiki) {
          toast("Buscando capa automática…");
          fetchPosterByTitle(it.title || "").then(function (url) {
            if (url) {
              it.poster = url;
              if ($("f_poster")) $("f_poster").value = url;
              if ($("bannerPreview")) { $("bannerPreview").src = url; $("bannerPreview").style.display = "block"; }
              try { if (it.id) rememberPoster(it.id, url); } catch (e) {}
            }
            afterPoster();
          }).catch(function () { afterPoster(); });
        } else {
          afterPoster();
        }
      } catch (e) {
        try { toast("Erro ao publicar"); } catch (e2) {}
      }
    };
    $("btnDraft").onclick = function () {
      var list = loadRaw();
      var it = null;
      for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
      if (!it) return;
      readInto(it);
      it.published = false;
      function finish() {
        saveRaw(list, "Rascunho salvo");
        renderEditor();
      }
      if (it.poster && it.poster.indexOf("data:image") === 0 && it.poster.length > 60000) {
        compressDataUrl(it.poster, 360, 0.65).then(function (p) {
          it.poster = p;
          finish();
        }).catch(function () { finish(); });
      } else {
        finish();
      }
    };
    $("btnDelete").onclick = function () {
      if (!confirm("Excluir? Some do app de filmes.")) return;
      var delId = currentId;
      var list = loadRaw().filter(function (x) { return x.id !== currentId; });
      currentId = list[0] ? list[0].id : null;
      if (delId) addDeletedId(delId);
      saveRaw(list, "Removido do app");
      renderEditor();
    };
    var addSeason = $("btnAddSeason");
    if (addSeason) {
      addSeason.onclick = function () {
        var list = loadRaw();
        var it = null;
        for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
        if (!it) return;
        readInto(it);
        it.seasons.push({ name: "Temporada " + (it.seasons.length + 1), episodes: [{ title: "Ep 1", url: "", subtitleUrl: "" }] });
        saveRaw(list);
        renderEditor();
      };
    }
    root.querySelectorAll("[data-add-ep]").forEach(function (b) {
      b.onclick = function () {
        var si = Number(b.getAttribute("data-add-ep"));
        var list = loadRaw();
        var it = null;
        for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
        if (!it) return;
        readInto(it);
        var n = it.seasons[si].episodes.length + 1;
        it.seasons[si].episodes.push({ title: "Ep " + n, url: "", subtitleUrl: "" });
        saveRaw(list);
        renderEditor();
      };
    });
    root.querySelectorAll("[data-del-season]").forEach(function (b) {
      b.onclick = function () {
        var si = Number(b.getAttribute("data-del-season"));
        var list = loadRaw();
        var it = null;
        for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
        if (!it) return;
        readInto(it);
        it.seasons.splice(si, 1);
        saveRaw(list);
        renderEditor();
      };
    });
  }
  $("btnNewMovie").onclick = function () {
    var list = loadRaw();
    var item = {
      id: uid(), type: "movie", title: "Novo filme", description: "", poster: "", url: "",
      subtitleUrl: "", published: false, releaseAt: "", genres: [], seasons: [], updatedAt: Date.now(), createdAt: Date.now()
    };
    list.unshift(item); currentId = item.id; saveRaw(list, "Filme criado"); renderEditor();
  };
  $("btnNewSeries").onclick = function () {
    var list = loadRaw();
    var item = {
      id: uid(), type: "series", title: "Nova série", description: "", poster: "", url: "",
      subtitleUrl: "", published: false, releaseAt: "", genres: [],
      seasons: [{ name: "Temporada 1", episodes: [{ title: "Ep 1", url: "", subtitleUrl: "" }] }],
      updatedAt: Date.now(), createdAt: Date.now()
    };
    list.unshift(item); currentId = item.id; saveRaw(list, "Série criada"); renderEditor();
  };
  function bakeEmbeddedIntoHtmlSource(src) {
    src = String(src || "");
    // usa versão lite (sem base64 gigante) para o HTML exportado caber no clipboard
    var emb = null;
    try { emb = buildEmbeddedBlock(); } catch (e) { emb = null; }
    var catJs, msgJs;
    if (emb && emb.text) {
      // emb.text já tem as duas vars; extrai cada uma
      catJs = "var EMBEDDED_CATALOG = " + (function () {
        try {
          var list = loadRaw() || [];
          var lite = list.map(function (it) {
            var o = {};
            for (var k in it) if (Object.prototype.hasOwnProperty.call(it, k)) o[k] = it[k];
            if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 90000) {
              o.poster = "";
            }
            return o;
          });
          return JSON.stringify(lite, null, 2);
        } catch (e) { return "[]"; }
      })() + ";";
      msgJs = "var EMBEDDED_MESSAGES = " + (function () {
        try { return JSON.stringify(loadMsgs() || [], null, 2); } catch (e) { return "[]"; }
      })() + ";";
    } else {
      catJs = "var EMBEDDED_CATALOG = [];";
      msgJs = "var EMBEDDED_MESSAGES = [];";
    }
    // Troca por índice (rápido) em vez de regex no HTML gigante com base64
    function replaceVarBlock(source, varName, newCode) {
      var needle = "var " + varName;
      var i = source.indexOf(needle);
      if (i < 0) {
        // tenta com espaços extras
        i = source.search(new RegExp("var\\s+" + varName + "\\s*="));
        if (i < 0) return source;
      }
      // acha o '=' depois do nome
      var eq = source.indexOf("=", i);
      if (eq < 0) return source;
      // percorre a partir do valor até o ';' que fecha o statement no nível 0
      var depth = 0;
      var inStr = false;
      var strQ = "";
      var esc = false;
      var j = eq + 1;
      // pula espaços
      while (j < source.length && (source.charAt(j) === " " || source.charAt(j) === "\n" || source.charAt(j) === "\r" || source.charAt(j) === "\t")) j++;
      for (; j < source.length; j++) {
        var ch = source.charAt(j);
        if (inStr) {
          if (esc) { esc = false; continue; }
          if (ch === "\\") { esc = true; continue; }
          if (ch === strQ) inStr = false;
          continue;
        }
        if (ch === '"' || ch === "'" || ch === "`") { inStr = true; strQ = ch; continue; }
        if (ch === "[" || ch === "{") { depth++; continue; }
        if (ch === "]" || ch === "}") { depth--; continue; }
        if (ch === ";" && depth <= 0) {
          // substitui de i até j (inclusive ;)
          return source.slice(0, i) + newCode + source.slice(j + 1);
        }
      }
      // se não achou fim, não mexe
      return source;
    }
    if (src.indexOf("EMBEDDED_CATALOG") >= 0) {
      src = replaceVarBlock(src, "EMBEDDED_CATALOG", catJs);
    } else if (src.indexOf("var DEMO") >= 0) {
      src = src.replace("var DEMO", catJs + "\n  var DEMO");
    }
    if (src.indexOf("EMBEDDED_MESSAGES") >= 0) {
      src = replaceVarBlock(src, "EMBEDDED_MESSAGES", msgJs);
    } else if (src.indexOf(catJs) >= 0) {
      src = src.replace(catJs, catJs + "\n  " + msgJs);
    }
    // DEMO aponta pro catálogo
    if (src.indexOf("var DEMO") >= 0) {
      src = replaceVarBlock(src, "DEMO", "var DEMO = (EMBEDDED_CATALOG && EMBEDDED_CATALOG[0]) || null;");
    }
    return src;
  }
  function copyTextToClipboard(str, okMsg) {
    str = String(str || "");
    if (!str) {
      try { toast("Nada para copiar"); } catch (e) {}
      return false;
    }
    var done = false;
    function ok(msg) {
      if (done) return;
      done = true;
      try { toast(msg || okMsg || "Copiado na área de transferência"); } catch (e) {}
    }
    // 1) Android shareText — usuário escolhe "Copiar"
    try {
      if (typeof AndroidAPI !== "undefined" && AndroidAPI && typeof AndroidAPI.shareText === "function") {
        AndroidAPI.shareText(str);
        ok((okMsg || "Pronto") + " · toque em Copiar no menu");
        return true;
      }
    } catch (e) {}
    // 2) Clipboard API
    if (navigator.clipboard && navigator.clipboard.writeText) {
      try {
        navigator.clipboard.writeText(str).then(function () {
          ok(okMsg || "Copiado na área de transferência");
        }).catch(function () {
          fallback();
        });
        return true;
      } catch (e) {}
    }
    function fallback() {
      if (done) return false;
      // 3) execCommand
      try {
        var ta = document.createElement("textarea");
        ta.value = str;
        ta.setAttribute("readonly", "");
        ta.style.cssText = "position:fixed;left:0;top:0;width:1px;height:1px;opacity:0.01;z-index:999999";
        document.body.appendChild(ta);
        ta.focus();
        ta.select();
        ta.setSelectionRange(0, str.length);
        var copied = false;
        try { copied = document.execCommand("copy"); } catch (e) {}
        try { document.body.removeChild(ta); } catch (e) {}
        if (copied) {
          ok(okMsg || "Copiado na área de transferência");
          return true;
        }
      } catch (e) {}
      // 4) prompt (último recurso — usuário copia manualmente)
      try {
        if (str.length < 50000) {
          prompt("Toque e segure → Copiar:", str);
          ok("Use Copiar no campo acima");
          return true;
        }
      } catch (e) {}
      try { toast("Não foi possível copiar automaticamente"); } catch (e) {}
      return false;
    }
    return fallback();
  }
  function downloadTextFile(filename, content) {
    try {
      var blob = new Blob([content], { type: "text/html;charset=utf-8" });
      var a = document.createElement("a");
      a.href = URL.createObjectURL(blob);
      a.download = filename || "CineTung.html";
      a.style.display = "none";
      document.body.appendChild(a);
      a.click();
      setTimeout(function () {
        try { URL.revokeObjectURL(a.href); a.remove(); } catch (e) {}
      }, 3000);
    } catch (e) {
      // fallback: tenta copiar o conteúdo
      try { copyTextToClipboard(content, "Conteúdo grande · use o compartilhar/salvar"); } catch (e2) {}
    }
  }
  function forceCopyText(str) {
    str = String(str || "");
    if (!str) return false;
    // textarea VISÍVEL (WebView Android falha com opacity 0 / tamanho 0)
    var ta = document.createElement("textarea");
    ta.value = str;
    ta.setAttribute("readonly", "readonly");
    ta.style.cssText = "position:fixed;top:0;left:0;width:90%;height:40%;opacity:1;z-index:2147483647;font-size:12px;background:#111;color:#fff;";
    document.body.appendChild(ta);
    var ok = false;
    try {
      ta.focus();
      ta.select();
      ta.setSelectionRange(0, str.length);
      ok = document.execCommand("copy") === true;
    } catch (e) { ok = false; }
    try { document.body.removeChild(ta); } catch (e) {}
    return ok;
  }
  function downloadTextFile(filename, text) {
    try {
      var blob = new Blob([text], { type: "text/html;charset=utf-8" });
      var url = URL.createObjectURL(blob);
      var a = document.createElement("a");
      a.href = url;
      a.download = filename || "CineTung_APP.html";
      a.style.display = "none";
      document.body.appendChild(a);
      a.click();
      setTimeout(function () {
        try { document.body.removeChild(a); } catch (e) {}
        try { URL.revokeObjectURL(url); } catch (e) {}
      }, 800);
      return true;
    } catch (e) {
      return false;
    }
  }
  function shareOrCopyLarge(str) {
    try {
      if (typeof AndroidAPI !== "undefined" && AndroidAPI) {
        if (typeof AndroidAPI.shareText === "function") {
          AndroidAPI.shareText(str);
          return "share";
        }
        if (typeof AndroidAPI.copyToClipboard === "function") {
          AndroidAPI.copyToClipboard(str);
          return "android-copy";
        }
      }
    } catch (e) {}
    try {
      if (navigator.share) {
        navigator.share({ title: "Cine Tung HTML", text: str }).catch(function () {});
        return "share";
      }
    } catch (e) {}
    return false;
  }
  function buildEmbeddedBlock() {
    var catList = [], msgList = [];
    try { catList = loadRaw() || []; } catch (e) {}
    try { msgList = loadMsgs() || []; } catch (e) {}
    // versão leve: tira data:image gigante (estoura clipboard do Android e "copia vazio")
    var lite = catList.map(function (it) {
      var o = {};
      for (var k in it) if (Object.prototype.hasOwnProperty.call(it, k)) o[k] = it[k];
      if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 90000) {
        o.poster = "";
        o._posterOmitted = true;
      }
      return o;
    });
    var catJson = "[]", msgJson = "[]";
    try { catJson = JSON.stringify(lite, null, 2); } catch (e) {}
    try { msgJson = JSON.stringify(msgList, null, 2); } catch (e) {}
    var text = "var EMBEDDED_CATALOG = " + catJson + ";\n\n" +
              "var EMBEDDED_MESSAGES = " + msgJson + ";\n";
    // se ainda gigante, minifica
    if (text.length > 200000) {
      try {
        catJson = JSON.stringify(lite);
        msgJson = JSON.stringify(msgList);
        text = "var EMBEDDED_CATALOG = " + catJson + ";\nvar EMBEDDED_MESSAGES = " + msgJson + ";\n";
      } catch (e) {}
    }
    return {
      text: text,
      nFilms: catList.length,
      nMsgs: msgList.length,
      omittedPosters: lite.some(function (x) { return x._posterOmitted; })
    };
  }
  function showCopyModal(text, title) {
    var modal = $("copyModal");
    var ta = $("copyModalText");
    if (!modal || !ta) return;
    window.__cineCopyPayload = String(text || "");
    ta.value = window.__cineCopyPayload;
    if ($("copyModalTitle")) $("copyModalTitle").textContent = title || "Código pronto";
    modal.classList.add("open");
  }
  function generateBakedHtml() {
    if (window.__cineTungCopying) {
      try { toast("Aguarde…"); } catch (e) {}
      return;
    }
    window.__cineTungCopying = true;
    /* sem toast de preparação */
    var emb = buildEmbeddedBlock();
    var textToCopy = "";
    var mode = "bloco";
    // 1) Tenta HTML COMPLETO do app (código + filmes embutidos)
    try {
      var html = "";
      try {
        html = "<!DOCTYPE html>\n" + document.documentElement.outerHTML;
      } catch (e) { html = ""; }
      if (html && html.length > 3000) {
        // embute catálogo/mensagens atuais
        try { html = bakeEmbeddedIntoHtmlSource(html); } catch (e) {}
        try {
          html = html.replace(/id="adminApp" class="on"/g, 'id="adminApp" class=""');
          html = html.replace(/id="viewApp" class=""/g, 'id="viewApp" class="on"');
          html = html.replace(/id="appBg"[^>]*>/g, 'id="appBg">');
          html = html.replace(/data:image\/[a-zA-Z0-9+.-]+;base64,[A-Za-z0-9+\/=]{40000,}/g, "");
          if (html.indexOf('id="splash"') < 0) {
            var inj = '<div id="splash" class="obliry-splash"><div class="ob-stage"><div class="ob-word"><span class="ob-letter">O</span><span class="ob-letter">B</span><span class="ob-letter">L</span><span class="ob-letter">I</span><span class="ob-letter">R</span><span class="ob-letter">Y</span><div class="ob-hole"></div></div><div class="ob-credit">Feito por Raphael Rocha</div></div><div class="ob-vignette"></div></div>';
            html = html.replace(/<body([^>]*)>/i, "<body$1>" + inj);
          }
          try {
            var bRaw = localStorage.getItem("cinetung_brtv_banners_v1");
            if (bRaw && html.indexOf("EMBEDDED_CATALOG") >= 0) {
              if (html.indexOf("EMBEDDED_BRTV_BANNERS") < 0) {
                html = html.replace("var EMBEDDED_CATALOG", "var EMBEDDED_BRTV_BANNERS = " + bRaw + ";\nvar EMBEDDED_CATALOG");
              }
            }
          } catch (eB) {}
        } catch (eSan) {}
        // export sempre no modo espectador + aba Biblioteca (nunca Perfil)
        try {
          html = html.replace(/id="adminApp" class=""/g, 'id="adminApp" class=""');
          html = html.replace(/id="viewApp" class="on"/g, 'id="viewApp" class="on"');
          html = html.replace(/<section class="page" id="page-lib">/g, '<section class="page active" id="page-lib">');
          html = html.replace(/<section class="page active" id="page-profile">/g, '<section class="page" id="page-profile">');
          html = html.replace(/data-tab="profile" class="active"/g, 'data-tab="profile"');
          html = html.replace(/class="" data-tab="lib"/g, 'class="active" data-tab="lib"');
          html = html.replace(/id="searchBar" style="display: none;"/g, 'id="searchBar" style="display: flex;"');
          html = html.replace(/class="drawer-open"/g, 'class=""');
        } catch (e) {}
        // remove data:image enormes que sobraram em qualquer lugar (senão clipboard zera)
        try {
          html = html.replace(/data:image\/[a-zA-Z0-9+.-]+;base64,[A-Za-z0-9+\/=]{150000,}/g, "");
        } catch (e) {}
        // garante que o bloco EMBEDDED do bake (já lite) está presente
        if (html.indexOf("EMBEDDED_CATALOG") >= 0 && html.indexOf("<html") >= 0) {
          textToCopy = html;
          mode = "html";
        }
      }
    } catch (e) {
      try { console.log(e); } catch (e2) {}
    }
    // 2) Fallback: só o bloco de dados
    if (!textToCopy) {
      textToCopy = emb.text;
      mode = "bloco";
    }
    var title =
      (mode === "html" ? "HTML completo Cine Tung · " : "Só bloco de dados · ") +
      emb.nFilms + " filme(s) · " + emb.nMsgs + " msg(s)" +
      (emb.omittedPosters ? " · posters base64 omitidos" : "");
    showCopyModal(textToCopy, title);
    if (emb.omittedPosters) {
      try { toast("Posters da galeria omitidos (use URL). HTML do app incluso."); } catch (e) {}
    }
    var result = forceCopyText(textToCopy);
    if (result === true) {
      try { toast(mode === "html" ? "HTML completo copiado" : "Bloco copiado"); } catch (e) {}
    } else if (result === "share") {
      try { toast("No menu Android toque em Copiar"); } catch (e) {}
    } else {
      try { toast("Use Copiar agora no painel"); } catch (e) {}
    }
    setTimeout(function () { window.__cineTungCopying = false; }, 800);
  }
  // === botão único: copiar HTML completo com filmes + mensagens atuais ===
  if ($("btnCopyFullHtml")) {
    $("btnCopyFullHtml").onclick = function (ev) {
      try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
      try { if (ev && ev.stopPropagation) ev.stopPropagation(); } catch (e) {}
      generateBakedHtml();
    };
  }
  if ($("copyModalClose")) {
    $("copyModalClose").onclick = function () {
      var m = $("copyModal");
      if (m) m.classList.remove("open");
    };
  }
  if ($("copyModalSelect")) {
    $("copyModalSelect").onclick = function () {
      var ta = $("copyModalText");
      if (!ta) return;
      try {
        ta.removeAttribute("readonly");
        ta.focus();
        ta.select();
        ta.setSelectionRange(0, ta.value.length);
        toast("Segure o texto azul → Copiar");
      } catch (e) {}
    };
  }
  if ($("copyModalCopy")) {
    $("copyModalCopy").onclick = function (ev) {
      try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
      var ta = $("copyModalText");
      var str = (window.__cineCopyPayload || (ta && ta.value) || "");
      if (!String(str).trim()) { toast("Nada para copiar"); return; }
      var len = str.length;
      var btn = $("copyModalCopy");
      if (btn) { btn.disabled = true; btn.textContent = "Copiando…"; }
      function done(msg) {
        if (btn) { btn.disabled = false; btn.textContent = "📋 Copiar agora"; }
        toast(msg);
      }
      // Textos grandes: clipboard do WebView mente "ok" e fica vazio — não tenta copiar
      if (len > 20000) {
        if (downloadTextFile("CineTung_APP.html", str)) {
          done("HTML baixado · envie esse arquivo no GitHub (clipboard não segura texto grande)");
          return;
        }
        var sh = shareOrCopyLarge(str);
        if (sh === "share") {
          done("Compartilhar aberto · salve no Drive ou Copiar");
          return;
        }
        if (sh === "android-copy") {
          done("Pedido de cópia ao Android");
          return;
        }
        try {
          if (ta) {
            ta.focus();
            ta.select();
            ta.setSelectionRange(0, str.length);
          }
        } catch (e) {}
        done("Use Baixar HTML · clipboard não funciona com arquivo grande");
        return;
      }
      // 1) textarea do modal (visível)
      var ok = false;
      try {
        if (ta) {
          ta.removeAttribute("readonly");
          ta.focus();
          ta.select();
          ta.setSelectionRange(0, str.length);
          ok = document.execCommand("copy") === true;
        }
      } catch (e) { ok = false; }
      // 2) textarea auxiliar visível
      if (!ok) {
        try { ok = forceCopyText(str) === true; } catch (e) { ok = false; }
      }
      // 3) Clipboard API (só confirma no then)
      if (!ok && navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(str).then(function () {
          done("Copiado!");
        }).catch(function () {
          var sh2 = shareOrCopyLarge(str);
          if (sh2 === "share") done("Use Copiar no menu de compartilhar");
          else if (downloadTextFile("CineTung_APP.html", str)) done("Baixado como arquivo");
          else done("Falhou · Selecionar tudo e copie na mão");
        });
        return;
      }
      if (ok) {
        done("Tentou copiar · cole em outro app para CONFERIR (se vazio, use Baixar HTML)");
      } else {
        var sh3 = shareOrCopyLarge(str);
        if (sh3 === "share") done("Escolha Copiar no menu");
        else if (downloadTextFile("CineTung_APP.html", str)) done("Arquivo HTML baixado");
        else {
          try {
            if (ta) { ta.focus(); ta.select(); ta.setSelectionRange(0, str.length); }
          } catch (e) {}
          done("Selecione o texto e copie manualmente");
        }
      }
    };
  }
  if ($("copyModalDownload")) {
    $("copyModalDownload").onclick = function () {
      var str = (window.__cineCopyPayload || ($("copyModalText") && $("copyModalText").value) || "");
      if (!String(str).trim()) { toast("Nada para baixar"); return; }
      if (downloadTextFile("CineTung_APP.html", str)) toast("Download iniciado");
      else {
        var sh = shareOrCopyLarge(str);
        if (sh) toast("Use compartilhar / salvar");
        else toast("Não foi possível baixar neste WebView");
      }
    };
  }
  if ($("copyModal")) {
    $("copyModal").addEventListener("click", function (e) {
      if (e.target === $("copyModal")) $("copyModal").classList.remove("open");
    });
  }
  // === mensagens offline (sem servidor) ===
  // Espectador: compartilha um pacote CTMSG: base64/json via WhatsApp etc.
  function buildMsgSharePayload(m) {
    var obj = {
      v: 1,
      id: m.id,
      fromName: m.fromName || "Espectador",
      fromAvatar: m.fromAvatar || "",
      text: m.text || "",
      at: m.at || Date.now()
    };
    var json = JSON.stringify(obj);
    // prefixo fácil de reconhecer
    try {
      return "CTMSG:" + btoa(unescape(encodeURIComponent(json)));
    } catch (e) {
      return "CTMSG:" + json;
    }
  }
  function parseMsgSharePayload(raw) {
    raw = String(raw || "").trim();
    if (!raw) return null;
    // aceita com ou sem prefixo, e também JSON puro
    var body = raw;
    if (body.indexOf("CTMSG:") === 0) body = body.slice(6).trim();
    // tenta base64
    try {
      var decoded = decodeURIComponent(escape(atob(body)));
      var o = JSON.parse(decoded);
      if (o && (o.text || o.fromName)) return o;
    } catch (e) {}
    // tenta JSON direto
    try {
      var o2 = JSON.parse(body);
      if (o2 && (o2.text || o2.fromName)) return o2;
    } catch (e2) {}
    // se for só texto livre, cria mensagem
    if (raw.length > 2) {
      return { id: "m_imp_" + Date.now().toString(36), fromName: "Importado", text: raw, at: Date.now() };
    }
    return null;
  }
  if ($("btnShareMsg")) {
    $("btnShareMsg").onclick = function () {
      var ta = $("msgText");
      var text = (ta && ta.value ? ta.value : "").trim();
      if (!text) { toast("Escreva algo na caixa de texto"); return; }
      copyTextToClipboard(text, "Texto copiado · cole no WhatsApp");
    };
  }
  if ($("btnImportMsg")) {
    $("btnImportMsg").onclick = function () {
      var ta = $("adminMsgPaste");
      var raw = (ta && ta.value ? ta.value : "").trim();
      if (!raw) {
        // tenta ler da área de transferência
        if (navigator.clipboard && navigator.clipboard.readText) {
          navigator.clipboard.readText().then(function (clip) {
            doImport(clip);
          }).catch(function () {
            toast("Cole o código no campo e toque de novo");
          });
          return;
        }
        toast("Cole o código no campo acima");
        return;
      }
      doImport(raw);
      function doImport(src) {
        var m = parseMsgSharePayload(src);
        if (!m) { toast("Código inválido"); return; }
        if (!m.id) m.id = "m_imp_" + Date.now().toString(36);
        if (!m.at) m.at = Date.now();
        var list = loadMsgs();
        // evita duplicata pelo id
        var exists = false;
        for (var i = 0; i < list.length; i++) if (list[i].id === m.id) { exists = true; break; }
        if (!exists) {
          list.unshift(m);
          saveMsgs(list);
        }
        if (ta) ta.value = "";
        var info = $("adminMsgInfo");
        if (info) info.textContent = "Importada: " + (m.fromName || "") + " · " + new Date(m.at).toLocaleString();
        toast(exists ? "Já estava na lista" : "Mensagem importada");
        // re-render admin list
        var box = $("adminMsgList");
        if (box) {
          box.innerHTML = "";
          loadMsgs().forEach(function (mm) {
            var el = document.createElement("div");
            el.className = "hist-card";
            el.innerHTML =
              '<img src="' + (mm.fromAvatar || "") + '" style="width:40px;height:40px;border-radius:50%" />' +
              '<div class="meta"><b>' + escapeHtml(mm.fromName || "Espectador") + '</b>' +
              '<small>' + new Date(mm.at).toLocaleString() + '</small>' +
              '<div style="margin-top:6px">' + escapeHtml(mm.text || "") + '</div></div>';
            box.appendChild(el);
          });
        }
      }
    };
  }
  // handlers antigos de GitHub/JSON removidos de propósito (botões não existem mais)
  // banner OK
  if ($("btnBannerOk")) {
    $("btnBannerOk").onclick = function () {
      $("updateBanner").classList.remove("show");
      renderGrid();
    };
  }
  // ao voltar pro app / foco → puxa catálogo do GitHub + redesenha
  function onAppActive() {
    if ($("viewApp").classList.contains("on")) {
      try {
        pullCatalogFromGitHub(true).then(function () {
          try { renderGrid(); } catch (e) {}
        }).catch(function () {
          try { renderGrid(); } catch (e) {}
        });
      } catch (e) {
        try { renderGrid(); } catch (e2) {}
      }
    }
  }
  document.addEventListener("visibilitychange", function () {
    if (!document.hidden) onAppActive();
  });
  window.addEventListener("focus", onAppActive);
  window.addEventListener("online", onAppActive);
  /* Brasil TV */
  /* Brasil TV — layout compacto */
  var BRTV_KEY = "cinetung_brtv_banners_v1";
  var homeHeroIndex = 0;
  var homeHeroTimer = null;
  function loadBrtvBanners() {
    function hasImage(b) {
      return b && b.image && String(b.image).trim().length > 8;
    }
    function fromCatalog() {
      var list = [];
      try {
        list = publishedList().slice(0, 6).map(function (m) {
          return {
            id: "auto_" + (m.id || ""),
            title: m.title || "",
            image: m.poster || "",
            movieId: m.id || ""
          };
        }).filter(hasImage);
      } catch (e) {}
      return list;
    }
    try {
      var local = JSON.parse(localStorage.getItem(BRTV_KEY) || "[]");
      if (Array.isArray(local) && local.length) {
        var good = local.filter(hasImage);
        if (good.length) return good;
      }
    } catch (e) {}
    try {
      if (typeof EMBEDDED_BRTV_BANNERS !== "undefined" && EMBEDDED_BRTV_BANNERS && EMBEDDED_BRTV_BANNERS.length) {
        var emb = EMBEDDED_BRTV_BANNERS.filter(hasImage);
        if (emb.length) return emb.slice();
      }
    } catch (e) {}
    return fromCatalog();
  }
  function saveBrtvBanners(arr) {
    try { localStorage.setItem(BRTV_KEY, JSON.stringify(arr || [])); } catch (e) {}
  }
  function openBannerTarget(b) {
    if (!b) return;
    try {
      var mid = b.movieId || "";
      var all = publishedList();
      var item = null;
      if (mid) {
        for (var i = 0; i < all.length; i++) {
          if (all[i].id === mid) { item = all[i]; break; }
        }
      }
      if (!item && b.title) {
        var t = String(b.title).toLowerCase();
        for (var j = 0; j < all.length; j++) {
          if (String(all[j].title || "").toLowerCase() === t) { item = all[j]; break; }
        }
      }
      if (item) openPlayer(item);
    } catch (e) {}
  }
  function applyHomeHeroSlide() {
    var track = document.getElementById("homeHeroTrack");
    var dots = document.getElementById("homeHeroDots");
    if (track) track.style.transform = "translateX(-" + (homeHeroIndex * 100) + "%)";
    if (dots) {
      var spans = dots.querySelectorAll("span");
      for (var i = 0; i < spans.length; i++) spans[i].className = i === homeHeroIndex ? "on" : "";
    }
  }
  function startHomeHeroAuto() {
    if (homeHeroTimer) clearInterval(homeHeroTimer);
    homeHeroTimer = setInterval(function () {
      var bans = loadBrtvBanners();
      if (!bans.length) return;
      homeHeroIndex = (homeHeroIndex + 1) % bans.length;
      applyHomeHeroSlide();
    }, 5000);
  }
  function renderHomeHero() {
    var bans = loadBrtvBanners();
    var track = document.getElementById("homeHeroTrack");
    var dots = document.getElementById("homeHeroDots");
    var hero = document.getElementById("homeHero");
    if (!track || !hero) return;
    track.innerHTML = "";
    if (dots) dots.innerHTML = "";
    if (!bans.length) {
      track.innerHTML = '<div class="home-hero-slide"><div class="home-hero-empty">Adicione banners no painel do criador</div></div>';
      homeHeroIndex = 0;
      return;
    }
    if (homeHeroIndex >= bans.length) homeHeroIndex = 0;
    bans.forEach(function (b, i) {
      var slide = document.createElement("div");
      slide.className = "home-hero-slide";
      var imgUrl = b.image ? String(b.image).replace(/"/g, "%22") : "";
      if (imgUrl) {
        slide.style.backgroundImage = 'url("' + imgUrl + '")';
        slide.style.backgroundSize = "cover";
        slide.style.backgroundPosition = "center center";
        slide.innerHTML = '<img class="home-hero-img" src="' + imgUrl + '" alt="" loading="lazy" />' +
          '<div class="cap">' + escapeHtml(b.title || "Destaque") + '</div>';
      } else {
        slide.innerHTML = '<div class="cap">' + escapeHtml(b.title || "Destaque") + '</div>';
      }
      var openB = (function (banner) {
        return function () { openBannerTarget(banner); };
      })(b);
      var stx = 0, sty = 0, smoved = false;
      slide.addEventListener("touchstart", function (ev) {
        smoved = false;
        try {
          var t = ev.changedTouches && ev.changedTouches[0];
          if (t) { stx = t.clientX; sty = t.clientY; }
        } catch (e) {}
      }, { passive: true });
      slide.addEventListener("touchmove", function (ev) {
        try {
          var t = ev.changedTouches && ev.changedTouches[0];
          if (!t) return;
          if (Math.abs(t.clientX - stx) > 12 || Math.abs(t.clientY - sty) > 12) smoved = true;
        } catch (e) {}
      }, { passive: true });
      slide.addEventListener("touchend", function (ev) {
        if (smoved) return;
        try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
        openB();
      }, { passive: false });
      slide.addEventListener("click", function () { openB(); });
      track.appendChild(slide);
      if (dots) {
        var d = document.createElement("span");
        if (i === homeHeroIndex) d.className = "on";
        d.onclick = (function (idx) {
          return function (ev) {
            try { ev.stopPropagation(); } catch (e) {}
            homeHeroIndex = idx;
            applyHomeHeroSlide();
            startHomeHeroAuto();
          };
        })(i);
        dots.appendChild(d);
      }
    });
    applyHomeHeroSlide();
    startHomeHeroAuto();
  }
  function renderBrtvBannerEditor() {
    var box = $("brtvBannerEditor");
    if (!box) return;
    box.innerHTML = "";
    var arr = loadBrtvBanners();
    if (!arr.length) {
      box.innerHTML = '<p class="muted" style="font-size:12px">Nenhum banner. Toque em + Banner.</p>';
    }
    arr.forEach(function (b, idx) {
      var row = document.createElement("div");
      row.style.cssText = "margin-bottom:10px;padding:8px;border:1px solid var(--line);border-radius:10px";
      var t = document.createElement("input");
      t.placeholder = "Titulo";
      t.value = b.title || "";
      t.style.cssText = "width:100%;margin-bottom:6px";
      t.onchange = function () {
        var a = loadBrtvBanners();
        if (a[idx]) { a[idx].title = t.value.trim(); saveBrtvBanners(a); renderHomeHero(); }
      };
      var u = document.createElement("input");
      u.placeholder = "URL imagem";
      u.value = (b.image && String(b.image).indexOf("data:") === 0) ? "" : (b.image || "");
      u.style.cssText = "width:100%;margin-bottom:6px";
      u.onchange = function () {
        var a = loadBrtvBanners();
        if (a[idx] && u.value.trim()) { a[idx].image = u.value.trim(); saveBrtvBanners(a); renderHomeHero(); }
      };
      var f = document.createElement("input");
      f.type = "file";
      f.accept = "image/*";
      f.style.cssText = "width:100%;margin-bottom:6px;font-size:11px";
      f.onchange = function () {
        var file = f.files && f.files[0];
        if (!file) return;
        try {
          compressImageFile(file, 720, 0.55).then(function (dataUrl) {
            var a = loadBrtvBanners();
            if (a[idx]) { a[idx].image = dataUrl; saveBrtvBanners(a); renderBrtvBannerEditor(); renderHomeHero(); toast("Banner ok"); }
          });
        } catch (e) {}
      };
      var mid = document.createElement("input");
      mid.placeholder = "ID do filme (opcional)";
      mid.value = b.movieId || "";
      mid.style.cssText = "width:100%;margin-bottom:6px";
      mid.onchange = function () {
        var a = loadBrtvBanners();
        if (a[idx]) { a[idx].movieId = mid.value.trim(); saveBrtvBanners(a); }
      };
      var del = document.createElement("button");
      del.type = "button";
      del.className = "btn small danger";
      del.textContent = "Remover";
      del.onclick = function () {
        var a = loadBrtvBanners();
        a.splice(idx, 1);
        saveBrtvBanners(a);
        renderBrtvBannerEditor();
        renderHomeHero();
      };
      row.appendChild(t);
      row.appendChild(u);
      row.appendChild(f);
      row.appendChild(mid);
      row.appendChild(del);
      box.appendChild(row);
    });
  }
  // nav do carrossel
  (function bindHomeHeroNav() {
    var prev = document.getElementById("homeHeroPrev");
    var next = document.getElementById("homeHeroNext");
    if (prev) prev.onclick = function (e) {
      try { e.stopPropagation(); } catch (err) {}
      var bans = loadBrtvBanners();
      if (!bans.length) return;
      homeHeroIndex = (homeHeroIndex - 1 + bans.length) % bans.length;
      applyHomeHeroSlide();
      startHomeHeroAuto();
    };
    if (next) next.onclick = function (e) {
      try { e.stopPropagation(); } catch (err) {}
      var bans = loadBrtvBanners();
      if (!bans.length) return;
      homeHeroIndex = (homeHeroIndex + 1) % bans.length;
      applyHomeHeroSlide();
      startHomeHeroAuto();
    };
  })();
  if ($("btnBrtvAddBanner")) {
    $("btnBrtvAddBanner").onclick = function () {
      var arr = loadBrtvBanners();
      arr.push({ id: "b_" + Date.now().toString(36), title: "Novo banner", image: "", movieId: "" });
      saveBrtvBanners(arr);
      renderBrtvBannerEditor();
      renderHomeHero();
    };
  }
  if ($("btnDrawerSettings")) {
    var _heroOld = $("btnDrawerSettings").onclick;
    $("btnDrawerSettings").onclick = function () {
      try { if (typeof _heroOld === "function") _heroOld(); } catch (e) {}
      var p = $("drawerSettingsPanel");
      if (p && !p.classList.contains("hidden")) renderBrtvBannerEditor();
    };
  }
  // init (protegido — não trava na splash se algo falhar)
  try { clearClickBlockers(); } catch (e) {}
  // no boot: SEMPRE espectador
  try {
    window.__cineAdminUnlocked = false;
    localStorage.setItem("cinetung_ui", "viewer");
    var a0 = document.getElementById("adminApp");
    var v0 = document.getElementById("viewApp");
    if (a0) { a0.classList.remove("on"); a0.style.display = "none"; }
    if (v0) { v0.classList.add("on"); v0.style.display = "block"; }
  } catch (e) {}
  try { bootstrapCatalogFromEmbedded(); } catch (e) {}
  try { ensureDemo(); } catch (e) { catalog = attachPosters((EMBEDDED_CATALOG || []).slice()); memoryCatalog = catalog; }
  try { loadProfileUI(); } catch (e) {}
  try { setTab("lib"); } catch (e) {}
  try { renderGrid(); } catch (e) {}
  // sempre mostra o viewer (evita tela preta)
  try {
    if ($("adminApp")) { $("adminApp").classList.remove("on"); $("adminApp").style.display = "none"; }
    if ($("viewApp")) { $("viewApp").classList.add("on"); $("viewApp").style.display = "block"; $("viewApp").style.pointerEvents = "auto"; }
    localStorage.setItem("cinetung_ui", "viewer");
  } catch (e) {}
  try { showViewer(); } catch (e) {
    try { $("viewApp").classList.add("on"); $("adminApp").classList.remove("on"); } catch (e2) {}
  }
  try { setTab("lib"); } catch (e) {}
  try { window.setTab = setTab; } catch (e) {}
  // carrossel usa capas do catálogo (remove banners manuais antigos/errados no aparelho)
  try {
    localStorage.removeItem("cinetung_brtv_banners_v1");
  } catch (e) {}
  setTimeout(function () { try { setTab("lib"); } catch (e) {} }, 400);
  setTimeout(function () { try { clearClickBlockers(); setTab("lib"); } catch (e) {} }, 2000);
  function clearClickBlockers() {
    try {
      document.body.classList.remove("drawer-open");
      // NÃO mexe na splash/intro — ela se remove sozinha
      var ids = ["drawerBackdrop", "genreDrawer", "crownGate", "copyModal", "updateBanner"];
      ids.forEach(function (id) {
        var el = document.getElementById(id);
        if (!el) return;
        el.classList.remove("open", "show", "hide");
        if (id === "drawerBackdrop") {
          el.style.pointerEvents = "none";
          el.style.opacity = "0";
          el.style.visibility = "hidden";
        }
        if (id === "genreDrawer") {
          el.classList.remove("open");
          el.style.pointerEvents = "";
        }
      });
      var nav = document.getElementById("bottomNav");
      if (nav) nav.style.pointerEvents = "auto";
      var view = document.getElementById("viewApp");
      if (view) view.style.pointerEvents = "";
      var toast = document.getElementById("toast");
      if (toast) { toast.classList.remove("show"); toast.style.pointerEvents = "none"; }
    } catch (e) {}
  }
  function recoverAppUi() {
    try {
      clearClickBlockers();
      // splash/intro: não remover aqui (só depois que a intro terminar sozinha)
      if (window.__obliryIntroDone) {
        var sp = document.getElementById("splash");
        if (sp) {
          try { sp.style.display = "none"; sp.style.pointerEvents = "none"; } catch (e) {}
          try { if (sp.parentNode) sp.parentNode.removeChild(sp); } catch (e) {}
        }
      }
      // NÃO expulsa o criador se já desbloqueou
      if (!window.__cineAdminUnlocked) {
        var admin = document.getElementById("adminApp");
        var view = document.getElementById("viewApp");
        if (admin && admin.classList.contains("on") && !$("crownGate").classList.contains("open")) {
          // só volta pro viewer se caiu em estado estranho sem unlock
        }
        if (view) {
          view.style.visibility = "visible";
          view.style.opacity = "1";
        }
      }
      try { closeDrawer(); } catch (e) {}
      try { loadTheme(); } catch (e) {}
      try { if ($("viewApp") && $("viewApp").classList.contains("on")) renderGrid(); } catch (e) {}
    } catch (e) {}
  }
  window.addEventListener("pageshow", function () {
    try {
      recoverAppUi();
      if (sessionStorage.getItem("__cine_expect_return") === "1") {
        sessionStorage.removeItem("__cine_expect_return");
        recoverAppUi();
      }
    } catch (e) {}
  });
  document.addEventListener("visibilitychange", function () {
    if (!document.hidden) {
      try { recoverAppUi(); } catch (e) {}
    }
  });
  window.addEventListener("focus", function () {
    try { recoverAppUi(); } catch (e) {}
  });
  try { runSplash(); } catch (e) {
    try { endSplash(); } catch (e2) {}
  }
  // segurança: tira splash no máximo em 4s
  setTimeout(function () { try { endSplash(); } catch (e) {} }, 5500);
  /* splash controlada por runSplash */
  // Sync automático com GitHub (HTML completo ou catalog.json)
  try {
    setTimeout(function () {
      pullCatalogFromGitHub(true).then(function (changed) {
        if (changed) { try { renderGrid(); } catch (e) {} }
      }).catch(function () {});
    }, 1200);
  } catch (e) {}
})();
</script>

</body></html>
