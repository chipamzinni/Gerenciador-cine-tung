<!DOCTYPE html>
<html lang="pt-BR" style="--bg: #000000;"><head>
<script>window.__CINE_TUNG_APP__=1;</script>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no"/>
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
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=Instrument+Serif:ital@0;1&display=swap');

:root {
  --bg: #0a0a0b;
  --bg2: #111113;
  --panel: #161618;
  --panel2: #1c1c1f;
  --line: rgba(255,255,255,.08);
  --line2: rgba(255,255,255,.12);
  --text: #f5f5f4;
  --muted: #a1a1aa;
  --accent: #e8a87c;
  --accent-dim: rgba(232,168,124,.15);
  --accent2: #c4a574;
  --danger: #f07178;
  --ok: #7fd99a;
  --shadow: 0 12px 40px rgba(0,0,0,.45);
  --radius: 16px;
  --radius-sm: 10px;
  --ease: cubic-bezier(.22,1,.36,1);
  --font: "DM Sans", system-ui, sans-serif;
  --serif: "Instrument Serif", Georgia, serif;
}

* { box-sizing: border-box; }
html { -webkit-tap-highlight-color: transparent; }
body {
  margin: 0;
  font-family: var(--font);
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  line-height: 1.45;
  background-image:
    radial-gradient(ellipse 80% 50% at 50% -20%, rgba(232,168,124,.07), transparent),
    radial-gradient(ellipse 60% 40% at 100% 0%, rgba(120,100,80,.04), transparent);
  background-attachment: fixed;
}
button, input, textarea, select { font-family: inherit; }
.hidden { display: none !important; }

/* ——— Splash ——— */
#splash {
  position: fixed; inset: 0; z-index: 99999;
  background: #050505;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  transition: opacity .6s var(--ease), visibility .6s;
}
#splash.hide { opacity: 0; visibility: hidden; pointer-events: none; }
#splashLogo {
  font-family: var(--serif);
  font-size: clamp(42px, 11vw, 72px);
  letter-spacing: .28em;
  color: #fff;
  min-height: 1.2em;
  font-weight: 400;
  text-indent: .28em;
}
#splashCredit {
  margin-top: 18px; font-size: 12px; letter-spacing: .2em;
  text-transform: uppercase; color: var(--muted);
  opacity: 0; transition: opacity 1.1s var(--ease);
}
#splashCredit.show { opacity: 1; }
#splashHint { margin-top: 28px; color: #555; font-size: 12px; letter-spacing: .06em; }

/* ——— Viewer ——— */
#viewApp { padding-bottom: 88px; display: none; }
#viewApp.on { display: block !important; visibility: visible !important; animation: fadeUp .45s var(--ease); min-height: 100vh; }
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: none; }
}
@keyframes fadeIn {
  from { opacity: 0; } to { opacity: 1; }
}
@keyframes scaleIn {
  from { opacity: 0; transform: scale(.96); }
  to { opacity: 1; transform: none; }
}

.top {
  position: sticky; top: 0; z-index: 20;
  background: rgba(10,10,11,.82);
  backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px);
  border-bottom: 1px solid var(--line);
  padding: 14px 18px 12px;
}
.top-row {
  display: flex; align-items: center; justify-content: space-between;
  gap: 10px; margin-bottom: 12px;
}
.brand {
  font-family: var(--serif);
  font-weight: 400;
  font-size: 22px;
  letter-spacing: .04em;
}
.brand span { color: var(--accent); font-style: italic; }
.top-right { display: flex; align-items: center; gap: 10px; }
.crown-btn {
  width: 38px; height: 38px; border-radius: 50%;
  border: 1px solid rgba(232,168,124,.35);
  background: linear-gradient(145deg, #2a2218, #14110c);
  color: var(--accent);
  font-size: 16px; cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: transform .25s var(--ease), box-shadow .25s;
}
.crown-btn:active { transform: scale(.92); }

/* icons (no emoji) */
.ico-crown, .ico-lib, .ico-movie, .ico-series, .ico-user, .ico-search {
  display: inline-block; width: 18px; height: 18px; vertical-align: -3px;
  background: currentColor; opacity: .9;
}
.ico-crown {
  width: 16px; height: 14px;
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M3 19h18v2H3v-2zm2-6l3-7 4 5 4-5 3 7H5z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M3 19h18v2H3v-2zm2-6l3-7 4 5 4-5 3 7H5z'/%3E%3C/svg%3E") center/contain no-repeat;
  background: var(--accent, #e8b86d);
}
.ico-lib {
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M4 4h5v16H4V4zm6 0h5v16h-5V4zm6 0h4v16h-4V4z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M4 4h5v16H4V4zm6 0h5v16h-5V4zm6 0h4v16h-4V4z'/%3E%3C/svg%3E") center/contain no-repeat;
}
.ico-movie {
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M18 4l2 4h-3l-2-4h-2l2 4h-3l-2-4H8l2 4H7L5 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V4h-4z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M18 4l2 4h-3l-2-4h-2l2 4h-3l-2-4H8l2 4H7L5 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V4h-4z'/%3E%3C/svg%3E") center/contain no-repeat;
}
.ico-series {
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M21 3H3c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h5v2h8v-2h5c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 14H3V5h18v12z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M21 3H3c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h5v2h8v-2h5c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 14H3V5h18v12z'/%3E%3C/svg%3E") center/contain no-repeat;
}
.ico-user {
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M12 12c2.7 0 4.8-2.1 4.8-4.8S14.7 2.4 12 2.4 7.2 4.5 7.2 7.2 9.3 12 12 12zm0 2.4c-3.2 0-9.6 1.6-9.6 4.8v2.4h19.2v-2.4c0-3.2-6.4-4.8-9.6-4.8z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M12 12c2.7 0 4.8-2.1 4.8-4.8S14.7 2.4 12 2.4 7.2 4.5 7.2 7.2 9.3 12 12 12zm0 2.4c-3.2 0-9.6 1.6-9.6 4.8v2.4h19.2v-2.4c0-3.2-6.4-4.8-9.6-4.8z'/%3E%3C/svg%3E") center/contain no-repeat;
}
.ico-search {
  width: 16px; height: 16px; opacity: .55;
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M15.5 14h-.79l-.28-.27A6.47 6.47 0 0016 9.5 6.5 6.5 0 109.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zM9.5 14C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z'/%3E%3C/svg%3E") center/contain no-repeat;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='black' d='M15.5 14h-.79l-.28-.27A6.47 6.47 0 0016 9.5 6.5 6.5 0 109.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zM9.5 14C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z'/%3E%3C/svg%3E") center/contain no-repeat;
}
.bottom-nav button .sym {
  display: block; margin: 0 auto 2px; width: 22px; height: 22px;
}
.card .poster {
  background-color: #141416 !important;
  background-size: cover !important;
  background-position: center !important;
  position: relative;
}
.card .poster.no-art::after {
  content: attr(data-initials);
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  font-family: var(--serif, Georgia, serif);
  font-size: 28px; letter-spacing: .04em;
  color: rgba(232,184,109,.55);
  background: linear-gradient(160deg, #1a1a1e 0%, #0e0e10 100%);
}
.card .info b { font-weight: 600; }
.dot {
  display: inline-block; width: 6px; height: 6px; border-radius: 50%;
  background: var(--accent, #e8b86d); margin-right: 8px; vertical-align: middle;
}

.top-avatar {
  width: 38px; height: 38px; border-radius: 50%; object-fit: cover;
  border: 2px solid var(--line2); background: var(--panel2); cursor: pointer;
  transition: border-color .25s;
}
.top-avatar:hover { border-color: var(--accent); }
.search {
  display: flex; align-items: center; gap: 10px;
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 14px;
  padding: 0 14px;
  transition: border-color .25s, box-shadow .25s;
}
.search:focus-within {
  border-color: rgba(232,168,124,.4);
  box-shadow: 0 0 0 3px var(--accent-dim);
}
.search input {
  flex: 1; border: 0; background: transparent; color: var(--text);
  padding: 12px 0; outline: none; font-size: 14px;
}
.search input::placeholder { color: #71717a; }

main.view-main { max-width: 1100px; margin: 0 auto; padding: 20px 16px; }
.page { display: none; }
.page.active { display: block; animation: fadeUp .4s var(--ease); }
.page-title {
  margin: 0 0 6px;
  font-family: var(--serif);
  font-size: 28px;
  font-weight: 400;
  letter-spacing: .02em;
}
.muted { color: var(--muted); font-size: 12px; }

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(148px, 1fr));
  gap: 16px;
  margin-top: 16px;
}
.card {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: var(--radius);
  overflow: hidden;
  cursor: pointer;
  transition: transform .35s var(--ease), border-color .3s, box-shadow .35s;
  animation: scaleIn .45s var(--ease) both;
}
.card:nth-child(1) { animation-delay: .02s; }
.card:nth-child(2) { animation-delay: .05s; }
.card:nth-child(3) { animation-delay: .08s; }
.card:nth-child(4) { animation-delay: .11s; }
.card:nth-child(5) { animation-delay: .14s; }
.card:nth-child(6) { animation-delay: .17s; }
.card:hover {
  transform: translateY(-4px);
  border-color: var(--line2);
  box-shadow: var(--shadow);
}
.card:active { transform: scale(.98); }
.poster {
  aspect-ratio: 2/3;
  background: var(--panel2) center/cover no-repeat;
  transition: transform .5s var(--ease);
}
.card:hover .poster { transform: scale(1.03); }
.info { padding: 12px; }
.info b {
  display: block; font-size: 13px; font-weight: 600;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.info small { color: var(--muted); font-size: 11px; }
.badge {
  display: inline-block; font-size: 10px; padding: 3px 8px;
  border-radius: 999px; margin-top: 8px;
  background: var(--accent-dim); color: var(--accent);
  font-weight: 600; letter-spacing: .03em;
}
.empty {
  color: var(--muted); padding: 48px 16px; text-align: center;
  font-size: 14px;
}

.bottom-nav {
  position: fixed; left: 0; right: 0; bottom: 0; z-index: 100; pointer-events: auto;
  background: rgba(10,10,11,.92);
  backdrop-filter: blur(18px); -webkit-backdrop-filter: blur(18px);
  border-top: 1px solid var(--line);
  display: grid; grid-template-columns: repeat(4, 1fr);
  padding: 8px 6px calc(10px + env(safe-area-inset-bottom));
}
.bottom-nav button {
  border: 0; background: transparent; color: var(--muted); cursor: pointer;
  padding: 8px 4px; border-radius: 12px; font-size: 10px; font-weight: 600;
  letter-spacing: .04em; text-transform: uppercase;
  display: flex; flex-direction: column; align-items: center; gap: 4px;
  transition: color .25s, background .25s, transform .2s;
}
.bottom-nav button .sym { font-size: 20px; transition: transform .25s var(--ease); }
.bottom-nav button.active {
  color: var(--text);
  background: rgba(255,255,255,.06);
}
.bottom-nav button.active .sym { transform: scale(1.08); color: var(--accent); }

/* profile */
.profile-box {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 20px;
  padding: 24px 20px;
  max-width: 420px;
  animation: scaleIn .4s var(--ease);
}
.avatar-wrap {
  display: flex; gap: 18px; align-items: center; margin-bottom: 20px;
  padding-bottom: 18px; border-bottom: 1px solid var(--line);
}
.avatar {
  width: 96px; height: 96px; min-width: 96px; border-radius: 50%;
  object-fit: cover; background: var(--panel2);
  border: 3px solid var(--line2);
}
.file-btn {
  display: inline-block; position: relative; overflow: hidden;
  border-radius: 12px; background: var(--panel2); color: #fff;
  font-weight: 600; font-size: 13px; padding: 10px 14px; cursor: pointer;
  border: 1px solid var(--line); transition: border-color .25s;
}
.file-btn:hover { border-color: var(--accent); }
.file-btn input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }
.field { margin: 0 0 16px; }
.field label {
  display: block; color: var(--muted); font-size: 12px;
  margin-bottom: 8px; font-weight: 600; letter-spacing: .03em;
}
.btn {
  border: 0; border-radius: 14px; padding: 14px 16px; cursor: pointer;
  background: var(--accent); color: #1a120a; font-weight: 700;
  width: 100%; margin-top: 10px; font-size: 15px;
  transition: transform .2s var(--ease), filter .2s, box-shadow .25s;
}
.btn:hover { filter: brightness(1.05); box-shadow: 0 8px 24px rgba(232,168,124,.25); }
.btn:active { transform: scale(.98); }
.btn.secondary { background: var(--panel2); color: var(--text); }
.btn.secondary:hover { box-shadow: none; filter: brightness(1.1); }
.btn.block { width: 100%; }
.btn.small { width: auto; padding: 10px 14px; margin: 0; font-size: 13px; }
.btn.ok { background: var(--ok); color: #0a1a10; }
.btn.danger { background: var(--danger); color: #fff; }
input[type="text"], input[type="password"], input[type="datetime-local"], textarea, select {
  width: 100%; background: var(--bg2); color: var(--text);
  border: 1px solid var(--line); border-radius: 12px;
  padding: 12px 14px; outline: none;
  transition: border-color .25s, box-shadow .25s;
}
input:focus, textarea:focus, select:focus {
  border-color: rgba(232,168,124,.45);
  box-shadow: 0 0 0 3px var(--accent-dim);
}
textarea {
  min-height: 90px; resize: vertical;
  -webkit-user-select: text; user-select: text;
}
textarea#msgText, #msgText {
  -webkit-user-select: text !important;
  user-select: text !important;
  pointer-events: auto !important;
}

/* crown gate */
#crownGate {
  display: none; position: fixed; inset: 0; z-index: 80;
  background: rgba(0,0,0,.65);
  align-items: center; justify-content: center; padding: 16px;
  backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);
  animation: fadeIn .25s ease;
}
#crownGate.open { display: flex; }
.crown-box {
  width: min(360px, 100%);
  background: var(--panel);
  border: 1px solid rgba(232,168,124,.25);
  border-radius: 20px; padding: 22px;
  box-shadow: var(--shadow);
  animation: scaleIn .35s var(--ease);
}
.crown-box h3 {
  margin: 0 0 8px; font-size: 18px; font-family: var(--serif);
  font-weight: 400; color: var(--accent);
}
.crown-box .err { color: var(--danger); font-size: 12px; min-height: 16px; margin-top: 8px; }

/* panels */
.panel {
  display: none; position: fixed; inset: 0; z-index: 60;
  background: var(--bg); flex-direction: column;
}
.panel.open { display: flex; animation: fadeUp .35s var(--ease); }
.panel-head {
  display: flex; align-items: center; gap: 12px;
  padding: 14px 16px; border-bottom: 1px solid var(--line);
  background: rgba(10,10,11,.9);
  backdrop-filter: blur(12px);
}
.panel-body { flex: 1; overflow: auto; padding: 16px; }
.icon-btn {
  border: 0; border-radius: 10px; padding: 8px 12px; cursor: pointer;
  background: var(--panel2); color: var(--text); font-weight: 600;
  transition: background .2s;
}
.icon-btn:hover { background: #252528; }
.hist-card {
  display: flex; gap: 12px; align-items: center;
  background: var(--panel); border: 1px solid var(--line);
  border-radius: 14px; padding: 12px; margin-bottom: 10px;
  transition: border-color .25s, transform .25s var(--ease);
  animation: fadeUp .35s var(--ease) both;
}
.hist-card:hover { border-color: var(--line2); }
.hist-card img {
  width: 56px; height: 80px; object-fit: cover;
  border-radius: 8px; background: #222;
}
.hist-card .meta { flex: 1; min-width: 0; }
.hist-card .meta b { display: block; font-size: 14px; }
.hist-card .meta small { color: var(--muted); display: block; margin-top: 4px; }

/* player */
.overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,.9);
  display: none; z-index: 70; padding: 12px;
  align-items: center; justify-content: center;
  animation: fadeIn .25s ease;
}
.overlay.open { display: flex; }
.player-box {
  width: min(1100px, 100%);
  background: var(--bg2);
  border: 1px solid var(--line);
  border-radius: 18px; overflow: hidden;
  box-shadow: var(--shadow);
  animation: scaleIn .35s var(--ease);
}
.player-top {
  display: flex; justify-content: space-between; gap: 10px; align-items: center;
  padding: 14px 16px; border-bottom: 1px solid var(--line);
}
.player-top h3 { margin: 0; font-size: 16px; font-weight: 600; }
.stage { position: relative; background: #000; aspect-ratio: 16/9; }
.stage iframe, .stage video {
  position: absolute; inset: 0; width: 100%; height: 100%; border: 0;
}
.ctrl {
  display: flex; flex-wrap: wrap; gap: 8px; padding: 12px;
  border-top: 1px solid var(--line); align-items: center;
}
.ctrl button {
  border: 0; border-radius: 10px; padding: 8px 12px; cursor: pointer;
  background: var(--panel2); color: var(--text); font-weight: 600;
  transition: background .2s;
}
.ctrl button:hover { background: #2a2a2e; }
.ep-list {
  max-height: 160px; overflow: auto; border-top: 1px solid var(--line);
  padding: 10px; display: none;
}
.ep-list.show { display: block; }
.ep {
  padding: 10px 12px; border-radius: 8px; cursor: pointer;
  border: 1px solid transparent; transition: background .2s, border-color .2s;
}
.ep:hover { background: rgba(255,255,255,.04); }
.ep.active {
  border-color: rgba(232,168,124,.4);
  background: var(--accent-dim);
}

/* ADMIN */
#adminApp {
  display: none; min-height: 100vh;
  background:
    radial-gradient(900px 500px at 0% -10%, rgba(232,168,124,.06), transparent),
    var(--bg);
}
#adminApp.on { display: block; animation: fadeUp .4s var(--ease); }
.admin-header {
  display: flex; align-items: center; justify-content: space-between; gap: 12px;
  padding: 16px 22px; border-bottom: 1px solid var(--line);
  background: rgba(10,10,11,.9);
  backdrop-filter: blur(14px);
  position: sticky; top: 0; z-index: 10;
}
.wrap {
  max-width: 1200px; margin: 0 auto; padding: 20px;
  display: grid; grid-template-columns: 320px 1fr; gap: 18px;
}
@media (max-width: 900px) { .wrap { grid-template-columns: 1fr; } }
.card2 {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 18px; padding: 18px;
}
.list {
  display: flex; flex-direction: column; gap: 8px;
  max-height: 70vh; overflow: auto;
}
.item {
  display: flex; gap: 10px; align-items: center; padding: 10px;
  border: 1px solid var(--line); border-radius: 12px;
  background: var(--bg2); cursor: pointer;
  transition: border-color .25s, transform .2s;
}
.item:hover { border-color: var(--line2); }
.item.active { border-color: rgba(232,168,124,.5); box-shadow: 0 0 0 1px var(--accent-dim); }
.item img {
  width: 48px; height: 68px; object-fit: cover;
  border-radius: 6px; background: #222;
}
.item .meta { flex: 1; min-width: 0; }
.item .meta b {
  display: block; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.badge2 {
  font-size: 10px; padding: 3px 7px; border-radius: 999px; font-weight: 700;
  background: var(--panel2); color: var(--muted);
}
.badge2.on { background: rgba(127,217,154,.15); color: var(--ok); }
.banner-preview {
  width: 100%; max-height: 200px; object-fit: cover; border-radius: 12px;
  background: var(--bg2); border: 1px solid var(--line); margin-top: 8px; min-height: 100px;
}
.row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.btns { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 12px; }
.ep-box {
  border: 1px dashed var(--line); border-radius: 12px;
  padding: 12px; margin-top: 10px;
}
.toast {
  position: fixed; right: 16px; bottom: 16px;
  background: var(--panel); border: 1px solid var(--line);
  padding: 12px 16px; border-radius: 12px;
  opacity: 0; transform: translateY(10px);
  transition: .3s var(--ease); z-index: 100;
  box-shadow: var(--shadow); font-size: 13px; font-weight: 500;
}
.toast.show { opacity: 1; transform: none; }
#updateBanner {
  display: none; position: fixed; left: 12px; right: 12px; top: 12px; z-index: 95;
  background: var(--panel); border: 1px solid rgba(232,168,124,.35);
  border-radius: 14px; padding: 12px 14px;
  box-shadow: var(--shadow);
  align-items: center; justify-content: space-between; gap: 10px;
  animation: fadeUp .4s var(--ease);
}
#updateBanner.show { display: flex; }
#updateBanner b { font-size: 13px; }
#updateBanner .muted { margin: 2px 0 0; }
label.lbl {
  display: block; font-size: 12px; color: var(--muted);
  margin: 10px 0 6px; font-weight: 600;
}

/* ——— Genre drawer ——— */
.genre-fab {
  width: 38px; height: 38px; border-radius: 50%;
  border: 1px solid var(--line2);
  background: var(--panel2);
  color: var(--text);
  font-size: 16px; cursor: pointer;
  display: flex; align-items: center; justify-content: center;
}
.drawer-backdrop {
  position: fixed; inset: 0; z-index: 80;
  background: rgba(0,0,0,.55);
  opacity: 0; visibility: hidden; pointer-events: none;
  transition: opacity .3s var(--ease), visibility .3s;
}
.drawer-backdrop.open { opacity: 1; visibility: visible; pointer-events: auto; }
.genre-drawer {
  position: fixed; top: 0; left: 0; bottom: 0; z-index: 90;
  width: min(300px, 86vw);
  background: var(--panel);
  border-right: 1px solid var(--line);
  transform: translateX(-105%);
  transition: transform .35s var(--ease);
  display: flex; flex-direction: column;
  box-shadow: 8px 0 40px rgba(0,0,0,.4);
  pointer-events: none;
}
.genre-drawer.open { transform: translateX(0); pointer-events: auto; }
.drawer-head {
  display: flex; align-items: center; justify-content: space-between;
  padding: 16px 14px; border-bottom: 1px solid var(--line);
}
.drawer-head b { font-size: 15px; }
.drawer-body { flex: 1; overflow-y: auto; padding: 10px 0; }
.drawer-item {
  display: flex; align-items: center; gap: 10px;
  width: 100%; border: 0; background: transparent;
  color: var(--text); text-align: left;
  padding: 12px 18px; font-size: 14px; cursor: pointer;
  transition: background .2s;
}
.drawer-item:hover, .drawer-item.active {
  background: var(--accent-dim); color: var(--accent);
}
.drawer-item.active { font-weight: 700; }
.drawer-foot {
  border-top: 1px solid var(--line);
  padding: 10px 0 calc(12px + env(safe-area-inset-bottom));
}
.drawer-settings {
  padding: 12px 16px 4px;
}
.drawer-settings .lbl { margin-top: 10px; }
.drawer-credits {
  margin-top: 14px; padding-top: 12px; border-top: 1px solid var(--line);
  font-size: 12px; color: var(--muted); line-height: 1.5;
}
.drawer-credits b { color: var(--text); }
.genre-chip-row {
  display: flex; flex-wrap: wrap; gap: 8px; margin: 8px 0 4px;
}
.genre-chip {
  display: inline-flex; align-items: center; gap: 4px;
  padding: 6px 10px; border-radius: 999px;
  border: 1px solid var(--line2); background: var(--panel2);
  font-size: 12px; cursor: pointer; color: var(--muted);
  user-select: none;
}
.genre-chip.on {
  background: var(--accent-dim); color: var(--accent);
  border-color: rgba(232,168,124,.45); font-weight: 600;
}
.genre-filter-bar {
  display: flex; align-items: center; gap: 8px;
  margin: 8px 0 0; flex-wrap: wrap;
}
.genre-filter-bar .chip-active {
  font-size: 12px; color: var(--accent); font-weight: 600;
}


/* copy result modal */
#copyModal {
  display: none; position: fixed; inset: 0; z-index: 200;
  background: rgba(0,0,0,.72); align-items: center; justify-content: center;
  padding: 16px; pointer-events: auto;
}
#copyModal.open { display: flex; }
#copyModal .box {
  width: min(560px, 100%); max-height: 88vh;
  background: var(--panel); border: 1px solid var(--line2);
  border-radius: 16px; padding: 16px; display: flex; flex-direction: column; gap: 10px;
  box-shadow: var(--shadow);
}
#copyModal textarea {
  width: 100%; min-height: 220px; max-height: 50vh; resize: vertical;
  background: var(--bg2); color: var(--text); border: 1px solid var(--line);
  border-radius: 12px; padding: 12px; font-size: 11px; font-family: ui-monospace, monospace;
  -webkit-user-select: text; user-select: text;
}

/* OBLIRY intro — câmera entrando pelo O */
.obliry-splash{
  position:fixed; inset:0; z-index:2147483000;
  background:#000;
  display:flex; align-items:center; justify-content:center;
  overflow:hidden;
  transition: opacity .95s ease, visibility .95s ease;
}
.obliry-splash.hide{ opacity:0; visibility:hidden; pointer-events:none; }
.obliry-splash.gone{ display:none !important; }
.ob-stage{
  position:relative; width:100%; height:100%;
  display:flex; flex-direction:column; align-items:center; justify-content:center;
  transform-origin: 18% 48%; /* aponta para o O */
  will-change: transform;
}
.ob-word{
  position:relative;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  font-weight: 800;
  font-size: clamp(44px, 15vw, 96px);
  letter-spacing: 0.22em;
  color: #f3f3f3;
  line-height: 1;
  user-select: none;
  transform: scale(0.78);
  opacity: 0;
  filter: blur(10px);
}
.ob-letter{ display:inline-block; opacity:0; transform: translateY(22px); }
.ob-hole{
  position:absolute;
  left: 0;
  top: 50%;
  width: 0.78em; height: 0.78em;
  margin-left: -0.02em;
  transform: translateY(-50%) scale(0.12);
  border-radius: 50%;
  pointer-events: none;
  opacity: 0;
  background:
    radial-gradient(circle at 50% 42%,
      rgba(30,30,32,0.2) 0%,
      rgba(0,0,0,0.65) 42%,
      #000 100%);
  box-shadow:
    0 0 0 1px rgba(232,184,109,0.4),
    0 0 28px 6px rgba(232,184,109,0.22),
    inset 0 0 24px rgba(0,0,0,0.9);
  will-change: transform, opacity;
}
.ob-credit{
  margin-top: 32px;
  font-size: 12.5px;
  letter-spacing: 0.14em;
  color: #9a9a9a;
  opacity: 0;
  transform: translateY(12px);
  font-weight: 500;
  text-transform: none;
}
.ob-vignette{
  position:absolute; inset:0; pointer-events:none;
  background: radial-gradient(ellipse at 18% 48%, transparent 20%, rgba(0,0,0,0.55) 55%, rgba(0,0,0,0.92) 100%);
  opacity: 1;
  transition: opacity 1.4s ease;
}
.obliry-splash.phase-in .ob-word{
  opacity: 1; filter: blur(0); transform: scale(1);
  transition: opacity 1.25s ease, filter 1.35s ease, transform 1.55s cubic-bezier(.2,.75,.2,1);
}
.obliry-splash.phase-in .ob-letter{
  opacity: 1; transform: translateY(0);
  transition: opacity .75s ease, transform .9s cubic-bezier(.2,.75,.2,1);
}
.obliry-splash.phase-in .ob-letter:nth-child(1){ transition-delay: .18s; }
.obliry-splash.phase-in .ob-letter:nth-child(2){ transition-delay: .32s; }
.obliry-splash.phase-in .ob-letter:nth-child(3){ transition-delay: .46s; }
.obliry-splash.phase-in .ob-letter:nth-child(4){ transition-delay: .60s; }
.obliry-splash.phase-in .ob-letter:nth-child(5){ transition-delay: .74s; }
.obliry-splash.phase-in .ob-letter:nth-child(6){ transition-delay: .88s; }
.obliry-splash.phase-in .ob-credit{
  opacity: 1; transform: translateY(0);
  transition: opacity 1.1s ease 1.05s, transform 1.1s ease 1.05s;
}
.obliry-splash.phase-hole .ob-hole{
  opacity: 1;
  transform: translateY(-50%) scale(0.95);
  transition: opacity .9s ease, transform 1.6s cubic-bezier(.15,.7,.1,1);
}
.obliry-splash.phase-dive .ob-stage{
  animation: obDive 2.85s cubic-bezier(.1,.55,.1,1) forwards;
}
.obliry-splash.phase-dive .ob-hole{
  transform: translateY(-50%) scale(22);
  opacity: 1;
  transition: transform 2.85s cubic-bezier(.1,.55,.1,1), opacity 1.4s ease 1.3s;
}
.obliry-splash.phase-dive .ob-letter{
  opacity: 0;
  transition: opacity 1s ease .25s;
}
.obliry-splash.phase-dive .ob-credit{
  opacity: 0;
  transition: opacity .8s ease;
}
.obliry-splash.phase-dive .ob-vignette{
  opacity: 0;
  transition: opacity 1.6s ease .35s;
}
@keyframes obDive{
  0%   { transform: scale(1) translate(0,0); }
  35%  { transform: scale(1.45) translate(-6%, 1%); }
  100% { transform: scale(3.2) translate(-14%, 2%); }
}

</style>
</head>
<body style="background-color: rgb(0, 0, 0);">

<!-- OBLIRY INTRO -->
<style id="obliryIntroCss">
#splash.obliry-splash{
  position:fixed!important; inset:0!important; z-index:2147483646!important;
  background:#000!important; margin:0!important; padding:0!important;
  display:flex!important; align-items:center!important; justify-content:center!important;
  overflow:hidden!important;
  transition:opacity .55s ease, visibility .55s ease;
}
#splash.obliry-splash.hide,
#splash.obliry-splash.gone{
  opacity:0!important; visibility:hidden!important; pointer-events:none!important;
  display:none!important; z-index:-1!important;
}
#splash.obliry-splash.hide{ display:none!important; }
#splash.obliry-splash.gone{ display:none!important; }
#splash .ob-stage{
  position:relative; width:100%; height:100%;
  display:flex; flex-direction:column; align-items:center; justify-content:center;
  transform-origin: 16% 50%;
  will-change: transform;
}
#splash .ob-word{
  position:relative;
  font-family: system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;
  font-weight:800;
  font-size: clamp(48px, 16vw, 100px);
  letter-spacing: 0.2em;
  color:#f5f5f5;
  line-height:1;
  opacity:0;
  transform: scale(0.85);
  filter: blur(12px);
}
#splash .ob-letter{
  display:inline-block;
  opacity:0;
  transform: translateY(24px);
}
#splash .ob-hole{
  position:absolute;
  left:0; top:50%;
  width:0.82em; height:0.82em;
  margin-left:-0.04em;
  transform: translateY(-50%) scale(0.08);
  border-radius:50%;
  opacity:0;
  pointer-events:none;
  background: radial-gradient(circle at 50% 40%, rgba(40,40,42,.25) 0%, rgba(0,0,0,.7) 40%, #000 100%);
  box-shadow: 0 0 0 2px rgba(232,184,109,.45), 0 0 30px 8px rgba(232,184,109,.2), inset 0 0 30px #000;
}
#splash .ob-credit{
  margin-top:36px;
  font-size:13px;
  letter-spacing:0.16em;
  color:#a0a0a0;
  opacity:0;
  transform: translateY(14px);
  font-weight:500;
}
#splash .ob-vignette{
  position:absolute; inset:0; pointer-events:none;
  background: radial-gradient(ellipse at 16% 50%, transparent 18%, rgba(0,0,0,.5) 50%, rgba(0,0,0,.95) 100%);
}
/* phases */
#splash.phase-in .ob-word{
  opacity:1; filter:blur(0); transform:scale(1);
  transition: opacity 1.3s ease, filter 1.4s ease, transform 1.6s cubic-bezier(.2,.75,.2,1);
}
#splash.phase-in .ob-letter{
  opacity:1; transform:translateY(0);
  transition: opacity .8s ease, transform .95s cubic-bezier(.2,.75,.2,1);
}
#splash.phase-in .ob-letter:nth-child(1){ transition-delay:.2s; }
#splash.phase-in .ob-letter:nth-child(2){ transition-delay:.35s; }
#splash.phase-in .ob-letter:nth-child(3){ transition-delay:.5s; }
#splash.phase-in .ob-letter:nth-child(4){ transition-delay:.65s; }
#splash.phase-in .ob-letter:nth-child(5){ transition-delay:.8s; }
#splash.phase-in .ob-letter:nth-child(6){ transition-delay:.95s; }
#splash.phase-in .ob-credit{
  opacity:1; transform:translateY(0);
  transition: opacity 1.2s ease 1.15s, transform 1.2s ease 1.15s;
}
#splash.phase-hole .ob-hole{
  opacity:1;
  transform: translateY(-50%) scale(1);
  transition: opacity 1s ease, transform 1.7s cubic-bezier(.15,.7,.1,1);
}
#splash.phase-dive .ob-stage{
  animation: obCamDive 1.55s cubic-bezier(.12,.6,.12,1) forwards;
}
#splash.phase-dive .ob-hole{
  transform: translateY(-50%) scale(24);
  transition: transform 1.55s cubic-bezier(.12,.6,.12,1), opacity .7s ease .5s;
}
#splash.phase-dive .ob-letter{ opacity:0; transition: opacity .45s ease .05s; }
#splash.phase-dive .ob-credit{ opacity:0; transition: opacity .4s ease; }
#splash.phase-dive .ob-vignette{ opacity:0; transition: opacity .7s ease .15s; }
@keyframes obCamDive{
  0%{ transform: scale(1) translate(0,0); }
  30%{ transform: scale(1.5) translate(-5%,0); }
  100%{ transform: scale(3.4) translate(-12%,1%); }
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
  function finish() {
    if (done) return;
    done = true;
    try { splash.classList.add("hide"); splash.style.pointerEvents="none"; splash.style.display="none"; } catch (e) {}
    setTimeout(function () {
      try {
        splash.classList.add("gone");
        if (splash.parentNode) splash.parentNode.removeChild(splash);
      } catch (e) {}
    }, 560);
  }
  // timeline (intro ok; preto no O mais curto)
  setTimeout(function () { try { splash.classList.add("phase-in"); } catch (e) {} }, 180);
  setTimeout(function () { try { splash.classList.add("phase-hole"); } catch (e) {} }, 1750);
  setTimeout(function () { try { splash.classList.add("phase-dive"); } catch (e) {} }, 2350);
  setTimeout(finish, 4000);
  // toque / clique = pular intro
  function skip(ev) {
    try { if (ev && ev.preventDefault) ev.preventDefault(); } catch (e) {}
    try { splash.classList.add("phase-hole"); } catch (e) {}
    try { splash.classList.add("phase-dive"); } catch (e) {}
    setTimeout(finish, 650);
  }
  splash.addEventListener("click", skip);
  splash.addEventListener("touchend", skip, { passive: false });
  // expõe para o app principal se quiser
  window.__obliryEndSplash = finish;
})();
</script>



<!-- ========== VIEWER ========== -->
<div id="viewApp" class="on">
  <div class="top">
    <div class="top-row">
      <div class="brand">Cine <span>Tung</span></div>
      <div class="top-right">
        <button type="button" class="genre-fab" id="btnGenreDrawer" title="Gêneros">☰</button>
        <button type="button" class="crown-btn" id="crownBtn" title="Área restrita"><span class="ico-crown"></span></button>
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
      <p class="muted" id="syncInfo">7 título(s) · 21/11/2026, 23:01:31</p>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelLib">Gênero: Todos</span></div>
      <div class="grid" id="grid-lib"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-lib" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-movies">
      <h1 class="page-title">Filmes</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelMovies">Gênero: Todos</span></div>
      <div class="grid" id="grid-movies"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-movies" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-series">
      <h1 class="page-title">Séries</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelSeries">Gênero: Todos</span></div>
      <div class="grid" id="grid-series"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div></div>
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
  </main>

  <nav class="bottom-nav" id="bottomNav">
    <button type="button" class="active" data-tab="lib"><span class="sym ico-lib"></span>Biblioteca</button>
    <button type="button" data-tab="movies"><span class="sym ico-movie"></span>Filmes</button>
    <button type="button" data-tab="series"><span class="sym ico-series"></span>Séries</button>
    <button type="button" data-tab="profile"><span class="sym ico-user"></span>Perfil</button>
  </nav>
</div>


<!-- genre drawer -->
<div class="drawer-backdrop" id="drawerBackdrop"></div>
<aside class="genre-drawer" id="genreDrawer" aria-hidden="true">
  <div class="drawer-head">
    <b>Gêneros</b>
    <button type="button" class="icon-btn" id="drawerClose" style="width:36px;height:36px;border-radius:50%;border:1px solid var(--line);background:var(--panel2);color:var(--text);cursor:pointer">✕</button>
  </div>
  <div class="drawer-body" id="drawerGenreList"><button type="button" class="drawer-item active"><span class="dot"></span> Todos</button><button type="button" class="drawer-item"><span>·</span> Ação</button><button type="button" class="drawer-item"><span>·</span> Aventura</button><button type="button" class="drawer-item"><span>·</span> Comédia</button><button type="button" class="drawer-item"><span>·</span> Drama</button><button type="button" class="drawer-item"><span>·</span> Terror</button><button type="button" class="drawer-item"><span>·</span> Romance</button><button type="button" class="drawer-item"><span>·</span> Ficção</button><button type="button" class="drawer-item"><span>·</span> Animação</button><button type="button" class="drawer-item"><span>·</span> Documentário</button><button type="button" class="drawer-item"><span>·</span> Suspense</button><button type="button" class="drawer-item"><span>·</span> Fantasia</button></div>
  <div class="drawer-foot">
    <button type="button" class="drawer-item" id="btnDrawerHome"><span class="dot"></span> Início (Biblioteca)</button>
    <button type="button" class="drawer-item" id="btnDrawerSettings"><span>⚙️</span> Configurações</button>
    <div class="drawer-settings hidden" id="drawerSettingsPanel">
      <label class="lbl">Cor de fundo do app</label>
      <input type="color" id="bgColorPicker" value="#0a0a0b" style="width:100%;height:42px;border:0;border-radius:10px;background:transparent;cursor:pointer">
      <button class="btn small secondary block" type="button" id="btnResetBg" style="margin-top:8px">Cor padrão</button>
      <div class="drawer-credits">
        <b>Créditos</b><br>
        Raphael Rocha<br>
        Empresa: Obliry
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
      <div class="list" id="adminList"><div class="item"><img src="" alt=""><div class="meta"><b>Dexter</b><small class="muted">AO AR · Série</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Sonic : O filme</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Cruella</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Coraline e a Porta secreta</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>One Piece</b><small class="muted">AO AR · Série</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>Homem-aranha: Um novo dia</b><small class="muted">AO AR · Filme</small></div></div><div class="item"><img src="" alt=""><div class="meta"><b>JURASSIC WORLD</b><small class="muted">AO AR · Filme</small></div></div></div>
    </aside>
    <main class="card2" id="editor"><h2 style="margin:0 0 8px;font-size:16px">Editor</h2><p class="muted">Selecione ou crie um item.</p></main>
  </div>
</div>

<!-- history -->
<div class="panel" id="histPanel">
  <div class="panel-head">
    <button class="icon-btn" type="button" id="histBack">← Voltar</button>
    <div><b>Histórico</b><div class="muted">Continue de onde parou</div></div>
  </div>
  <div class="panel-body" id="histBody"></div>
</div>

<!-- messages user -->
<div class="panel" id="msgPanel">
  <div class="panel-head">
    <button class="icon-btn" type="button" id="msgBack">← Voltar</button>
    <div><b>Mensagem para o dono</b><div class="muted">Escreva e copie para enviar ao dono</div></div>
  </div>
  <div class="panel-body">
    <textarea id="msgText" rows="4" autocomplete="off" autocorrect="on" autocapitalize="sentences" spellcheck="true" placeholder="Ex: Podem adicionar o filme X completo?" style="width:100%;min-height:110px;resize:vertical;-webkit-user-select:text;user-select:text;pointer-events:auto;"></textarea>
    <div class="btns" style="margin-top:10px">
      <button class="btn" type="button" id="msgSend" style="flex:1;width:auto;margin:0">Salvar local</button>
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
<div id="copyModal">
  <div class="box">
    <b id="copyModalTitle">Código pronto</b>
    <p class="muted" id="copyModalHint" style="margin:0">Toque em <b>Copiar agora</b>. Se a área de transferência ficar vazia, use <b>Selecionar tudo</b> → segurar → <b>Copiar</b>.</p>
    <textarea id="copyModalText" readonly=""></textarea>
    <div class="btns" style="margin:0;flex-wrap:wrap">
      <button class="btn small" type="button" id="copyModalSelect">Selecionar tudo</button>
      <button class="btn small ok" type="button" id="copyModalCopy">📋 Copiar agora</button>
      <button class="btn small secondary" type="button" id="copyModalClose">Fechar</button>
    </div>
  </div>
</div>
<div class="toast" id="toast"></div>

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
  var EMBEDDED_CATALOG = [
  {
    "id": "id_su7t8dsmwr6ugs9",
    "type": "series",
    "title": "Dexter",
    "description": "",
    "poster": "",
    "url": "",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "genres": [
      "Ação",
      "Terror",
      "Documentário",
      "Suspense",
      "Ficção"
    ],
    "seasons": [
      {
        "name": "Temporada 1",
        "episodes": [
          {
            "title": "Ep 1",
            "url": "https://latino.solo-latino.com/pt/detail/drama/9HS3fgJL64irdVnmhcFEf-Dexter-Season-1/1",
            "subtitleUrl": ""
          }
        ]
      }
    ],
    "updatedAt": 1795312868808,
    "createdAt": 1795312730745
  },
  {
    "id": "id_40f1o1wmwr6s93n",
    "type": "movie",
    "title": "Sonic : O filme",
    "description": "",
    "poster": "",
    "url": "https://es.cuevana4br.com/pt/detail/movie/hch3oQtwgSCZQ67Kitib6-Sonic-O-Filme",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "genres": [
      "Ação",
      "Aventura",
      "Ficção",
      "Animação",
      "Fantasia",
      "Suspense"
    ],
    "seasons": [],
    "updatedAt": 1795312887110,
    "createdAt": 1795312627475
  },
  {
    "id": "id_v4ero27mwr6pmza",
    "type": "movie",
    "title": "Cruella",
    "description": "",
    "poster": "",
    "url": "https://ver.123pelicula.com/pt/detail/movie/Qs1nBk4HkERH83EMoPTeo-Cruella",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "genres": [
      "Ação",
      "Suspense",
      "Drama",
      "Fantasia",
      "Ficção"
    ],
    "seasons": [],
    "updatedAt": 1795312891455,
    "createdAt": 1795312505494
  },
  {
    "id": "id_cp4b9k1mwqyf6ke",
    "type": "movie",
    "title": "Coraline e a Porta secreta",
    "description": "",
    "poster": "",
    "url": "https://ver.123pelicula.com/pt/detail/movie/dJOaDemkfHpWOgAihdiDM-Coraline",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "seasons": [],
    "updatedAt": 1795302415424,
    "createdAt": 1795298580734,
    "genres": [
      "Suspense",
      "Terror",
      "Aventura",
      "Ficção",
      "Fantasia",
      "Animação"
    ]
  },
  {
    "id": "id_tkr31g0mwql19g3",
    "type": "series",
    "title": "One Piece",
    "description": "",
    "poster": "",
    "url": "",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "seasons": [
      {
        "name": "Temporada 1",
        "episodes": [
          {
            "title": "Ep 1",
            "url": "https://ver.123pelicula.com/pt/detail/drama/hKQr7WDgMEBe4uEbU3CNU-One-Piece/1",
            "subtitleUrl": ""
          }
        ]
      },
      {
        "name": "Temporada 2",
        "episodes": [
          {
            "title": "Ep 1",
            "url": "https://ver.123pelicula.com/pt/detail/drama/hKQr7WDgMEBe4uEbU3CNU-One-Piece/2",
            "subtitleUrl": ""
          }
        ]
      }
    ],
    "updatedAt": 1795302397799,
    "createdAt": 1795276096275,
    "genres": [
      "Ação",
      "Aventura",
      "Ficção",
      "Animação",
      "Fantasia"
    ]
  },
  {
    "id": "id_mp0fas2mwqk3z5i",
    "type": "movie",
    "title": "Homem-aranha: Um novo dia",
    "description": "",
    "poster": "data:image/jpeg;base64,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",
    "url": "https://es.cuevana4br.com/pt/detail/movie/iswnnqdfuZdhcRaFZNeuO-Spider-Man-Brand-New-Day",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "seasons": [],
    "updatedAt": 1795302369847,
    "createdAt": 1795274543286,
    "genres": [
      "Ficção",
      "Ação",
      "Aventura"
    ]
  },
  {
    "id": "id_ayx45czmwqgp7lq",
    "type": "movie",
    "title": "JURASSIC WORLD",
    "description": "",
    "poster": "data:image/jpeg;base64,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",
    "url": "https://es.cuevana4br.com/pt/detail/movie/1h7LV1iNaWJ8J1Py6E2h1-Jurassic-World",
    "subtitleUrl": "",
    "published": true,
    "releaseAt": "",
    "seasons": [],
    "updatedAt": 1795302346121,
    "createdAt": 1795268815550,
    "genres": [
      "Ação",
      "Ficção",
      "Aventura",
      "Fantasia"
    ]
  }
];
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
  function resolvePoster(item) {
    if (!item) return "";
    var p = (item.poster || "").trim();
    if (p) return p;
    try {
      var map = loadPosterMap();
      if (item.id && map[item.id]) return map[item.id];
    } catch (e) {}
    return "";
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
  function hydratePostersFromEmbedded() {
    try {
      var emb = (typeof EMBEDDED_CATALOG !== "undefined" && EMBEDDED_CATALOG) ? EMBEDDED_CATALOG : [];
      if (!emb || !emb.length) return;
      var map = loadPosterMap();
      var changed = false;
      emb.forEach(function (it) {
        if (it && it.id && it.poster && String(it.poster).trim()) {
          if (!map[it.id] || map[it.id].length < String(it.poster).length) {
            map[it.id] = it.poster;
            changed = true;
          }
        }
      });
      if (changed) savePosterMap(map);
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
    $("adminApp").classList.remove("on");
    $("viewApp").classList.add("on");
    localStorage.setItem("cinetung_ui", "viewer");
    renderGrid();
    loadProfileUI();
  }
  function showAdmin() {
    $("viewApp").classList.remove("on");
    $("adminApp").classList.add("on");
    localStorage.setItem("cinetung_ui", "admin");
    catalog = loadRaw();
    renderAdminList();
    renderEditor();
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
  $("crownBtn").onclick = openCrown;
  $("crownEnter").onclick = tryCrown;
  $("crownCancel").onclick = closeCrown;
  $("crownCode").addEventListener("keydown", function (e) {
    if (e.key === "Enter") tryCrown();
  });
  // clicar fora da caixa fecha
  $("crownGate").addEventListener("click", function (e) {
    if (e.target === $("crownGate")) closeCrown();
  });
  $("btnBackViewer").onclick = showViewer;

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
  $("btnSaveProfile").onclick = function () {
    var name = ($("profileName").value || "Espectador").trim();
    var avatar = $("avatarImg").getAttribute("src") || "";
    localStorage.setItem(PROFILE_KEY, JSON.stringify({ name: name, avatar: avatar }));
    applyAvatar(avatar);
    toast("Perfil salvo");
  };
  $("avatarFile").onchange = function (ev) {
    var f = ev.target.files && ev.target.files[0];
    if (!f) return;
    var r = new FileReader();
    r.onload = function () { $("avatarImg").src = r.result; applyAvatar(r.result); };
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
    try {
      // fecha overlays que possam bloquear cliques
      try { closeDrawer(); } catch (e) {}
      try {
        var cm = $("copyModal");
        if (cm) cm.classList.remove("open");
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
  $("q").oninput = function () { renderGrid(); };

  /* ——— gaveta de gêneros + configurações ——— */
  function openDrawer() {
    if ($("genreDrawer")) $("genreDrawer").classList.add("open");
    if ($("drawerBackdrop")) $("drawerBackdrop").classList.add("open");
    if ($("genreDrawer")) $("genreDrawer").setAttribute("aria-hidden", "false");
    renderDrawerGenres();
  }
  function closeDrawer() {
    if ($("genreDrawer")) $("genreDrawer").classList.remove("open");
    if ($("drawerBackdrop")) $("drawerBackdrop").classList.remove("open");
    if ($("genreDrawer")) $("genreDrawer").setAttribute("aria-hidden", "true");
    if ($("drawerSettingsPanel")) $("drawerSettingsPanel").classList.add("hidden");
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
  function applyBgColor(hex) {
    if (!hex) hex = "#0a0a0b";
    try {
      document.documentElement.style.setProperty("--bg", hex);
      document.body.style.backgroundColor = hex;
      localStorage.setItem(THEME_KEY, hex);
      if ($("bgColorPicker")) $("bgColorPicker").value = hex;
    } catch (e) {}
  }
  function loadTheme() {
    try {
      var c = localStorage.getItem(THEME_KEY);
      if (c) applyBgColor(c);
    } catch (e) {}
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
  if ($("bgColorPicker")) {
    $("bgColorPicker").oninput = function () { applyBgColor($("bgColorPicker").value); };
  }
  if ($("btnResetBg")) {
    $("btnResetBg").onclick = function () {
      applyBgColor("#0a0a0b");
      toast("Cor padrão");
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
      el.onclick = function () { openPlayer(item); };
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
    box.style.cssText = "position:absolute;inset:0;display:flex;align-items:center;justify-content:center;padding:20px;background:#0a0a0b";
    var safe = String(url || "").replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/"/g, "&quot;");
    box.innerHTML =
      '<div style="max-width:360px;width:100%;background:#121214;border:1px solid #2a2a2e;border-radius:16px;padding:22px;text-align:center">' +
      '<div style="font-size:15px;font-weight:600;margin-bottom:8px;color:#f4f4f5">Abrir no navegador</div>' +
      '<div style="font-size:13px;color:#a1a1aa;line-height:1.45;margin-bottom:16px">' +
      (reason ? String(reason) : "Este link precisa abrir fora do app.") +
      '</div>' +
      '<button type="button" id="btnExtOpen" style="width:100%;border:0;border-radius:12px;padding:14px 16px;background:#e8b86d;color:#111;font-weight:700;font-size:15px;cursor:pointer">Abrir agora</button>' +
      '<div style="margin-top:12px;font-size:11px;color:#52525b;word-break:break-all">' + safe + '</div></div>';
    stage.appendChild(box);
    var b = box.querySelector("#btnExtOpen");
    if (b) b.onclick = function () { openExternalUrl(url); };
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
    if (play) play.style.display = show ? "" : "none";
    if (speedWrap) speedWrap.style.display = show ? "" : "none";
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
    current = item; currentEp = null;
    var __resumeAt = Number(startSeconds) || 0;
    var stage = $("stage");
    $("epList").classList.remove("show");
    $("epList").innerHTML = "";
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
    var list = loadMsgs();
    list.unshift({
      id: "m_" + Date.now().toString(36) + Math.random().toString(36).slice(2, 6),
      fromName: p.name || "Espectador",
      fromAvatar: p.avatar || "",
      text: text,
      at: Date.now()
    });
    saveMsgs(list);
    if (ta) ta.value = "";
    renderMyMsgs();
    toast("Mensagem salva localmente");
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
      '<label class="lbl">Banner (URL)</label><input id="f_poster" value="' + escapeHtml(item.poster || "") + '" />' +
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
        try { toast("Comprimindo capa…"); } catch (e) {}
        compressImageFile(f, 300, 0.55).then(function (dataUrl) {
          $("f_poster").value = dataUrl;
          $("bannerPreview").src = dataUrl;
          try { if (item && item.id) rememberPoster(item.id, dataUrl); } catch (e) {}
          try { toast("Capa pronta (" + Math.round(dataUrl.length / 1024) + " KB) · use Copiar HTML p/ outros aparelhos"); } catch (e) {}
        }).catch(function () {
          // fallback sem compressão
          var r = new FileReader();
          r.onload = function () {
            $("f_poster").value = r.result;
            $("bannerPreview").src = r.result;
          };
          r.readAsDataURL(f);
        });
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
      item.poster = $("f_poster").value.trim();
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

    $("btnPublish").onclick = function () {
      var list = loadRaw();
      var it = null;
      for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
      if (!it) return;
      readInto(it);
      it.published = true;
      if (it.id) clearDeletedId(it.id);
      function finish() {
        saveRaw(list, "Publicado no app");
        renderEditor();
      }
      // garante capa leve para ir pro GitHub/outros aparelhos
      if (it.poster && it.poster.indexOf("data:image") === 0 && it.poster.length > 60000) {
        try { toast("Otimizando capa…"); } catch (e) {}
        compressDataUrl(it.poster, 360, 0.65).then(function (p) {
          it.poster = p;
          finish();
        }).catch(function () { finish(); });
      } else {
        finish();
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
    var ok = false;
    var ta = document.createElement("textarea");
    ta.value = str;
    ta.setAttribute("readonly", "");
    ta.style.cssText = "position:fixed;top:10px;left:10px;width:2px;height:2px;opacity:0.05;z-index:999999;";
    document.body.appendChild(ta);
    try {
      ta.focus();
      ta.select();
      ta.setSelectionRange(0, str.length);
      ok = document.execCommand("copy") === true;
    } catch (e) { ok = false; }
    try { document.body.removeChild(ta); } catch (e) {}
    if (ok) return true;
    try {
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(str);
        return true;
      }
    } catch (e) {}
    try {
      if (typeof AndroidAPI !== "undefined" && AndroidAPI && typeof AndroidAPI.shareText === "function") {
        AndroidAPI.shareText(str);
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
        // export sempre no modo espectador (evita outros aparelhos abrirem no Criador)
        try {
          html = html.replace(/id="adminApp" class="on"/g, 'id="adminApp" class=""');
          html = html.replace(/id="viewApp" class=""/g, 'id="viewApp" class="on"');
          html = html.replace(/id="viewApp" class=''/g, 'id="viewApp" class="on"');
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
      var str = (window.__cineCopyPayload || (ta && ta.value) || "").trim();
      if (!str) { toast("Nada para copiar"); return; }

      // 1) tenta copiar do próprio textarea do modal (visível = melhor no WebView)
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

      // 2) forceCopyText (textarea auxiliar)
      if (!ok) {
        var r = forceCopyText(str);
        if (r === true) ok = true;
        if (r === "share") {
          toast("No menu do Android toque em Copiar");
          return;
        }
      }

      // 3) clipboard API
      if (!ok && navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(str).then(function () {
          toast("Copiado!");
        }).catch(function () {
          toast("Falhou · use Selecionar tudo e copie na mão");
        });
        return;
      }

      if (ok) toast("Copiado · confira colando em outro app");
      else {
        // último recurso: abre share do Android
        try {
          if (typeof AndroidAPI !== "undefined" && AndroidAPI && AndroidAPI.shareText) {
            AndroidAPI.shareText(str);
            toast("Escolha Copiar no menu");
            return;
          }
        } catch (e) {}
        toast("Selecione o texto e copie manualmente");
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

  // init (protegido — não trava na splash se algo falhar)
  try { hydratePostersFromEmbedded(); } catch (e) {}
  try { ensureDemo(); } catch (e) { catalog = attachPosters((EMBEDDED_CATALOG || []).slice()); memoryCatalog = catalog; }
  try { loadProfileUI(); } catch (e) {}
  try { setTab("lib"); } catch (e) {}
  try { renderGrid(); } catch (e) {}
  // sempre mostra o viewer (evita tela preta)
  try {
    if ($("adminApp")) $("adminApp").classList.remove("on");
    if ($("viewApp")) $("viewApp").classList.add("on");
    localStorage.setItem("cinetung_ui", "viewer");
  } catch (e) {}
  try { showViewer(); } catch (e) {
    try { $("viewApp").classList.add("on"); $("adminApp").classList.remove("on"); } catch (e2) {}
  }

  function recoverAppUi() {
    try {
      // tira splash residual
      var sp = document.getElementById("splash");
      if (sp) {
        try { sp.classList.add("hide"); sp.classList.add("gone"); sp.style.display = "none"; sp.style.pointerEvents = "none"; } catch (e) {}
        try { if (sp.parentNode) sp.parentNode.removeChild(sp); } catch (e) {}
      }
      // garante viewer
      var admin = document.getElementById("adminApp");
      var view = document.getElementById("viewApp");
      if (admin) admin.classList.remove("on");
      if (view) {
        view.classList.add("on");
        view.style.display = "block";
        view.style.visibility = "visible";
        view.style.opacity = "1";
      }
      document.body.style.background = "#000";
      try { showViewer(); } catch (e) {}
      try { renderGrid(); } catch (e) {}
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
