<!DOCTYPE html>
<html lang="pt-BR" style="--bg: #000000;"><head>
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
#viewApp.on { display: block; animation: fadeUp .45s var(--ease); }
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
</style>
</head>
<body style="background-color: rgb(0, 0, 0);">


<!-- ========== VIEWER ========== -->
<div id="viewApp" class="">
  <div class="top">
    <div class="top-row">
      <div class="brand">Cine <span>Tung</span></div>
      <div class="top-right">
        <button type="button" class="genre-fab" id="btnGenreDrawer" title="Gêneros">☰</button>
        <button type="button" class="crown-btn" id="crownBtn" title="Área restrita">👑</button>
        <img class="top-avatar" id="topAvatar" alt="" title="Perfil" src="">
      </div>
    </div>
    <div class="search" id="searchBar" style="display: flex;">
      <span>🔍</span>
      <input id="q" placeholder="Pesquisar por nome...">
    </div>
  </div>

  <main class="view-main">
    <section class="page active" id="page-lib">
      <h1 class="page-title">📚 Biblioteca</h1>
      <p class="muted" id="syncInfo">7 título(s) · 21/11/2026, 23:01:31</p>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelLib">Gênero: Todos</span></div>
      <div class="grid" id="grid-lib"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-lib" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-movies">
      <h1 class="page-title">🎬 Filmes</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelMovies">Gênero: Todos</span></div>
      <div class="grid" id="grid-movies"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Sonic : O filme</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Cruella</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Coraline e a Porta secreta</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Homem-aranha: Um novo dia</b><small>Filme</small><div class="badge">Completo</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>JURASSIC WORLD</b><small>Filme</small><div class="badge">Completo</div></div></div></div>
      <div class="empty" id="empty-movies" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-series">
      <h1 class="page-title">📺 Séries</h1>
      <div class="genre-filter-bar"><span class="chip-active" id="genreLabelSeries">Gênero: Todos</span></div>
      <div class="grid" id="grid-series"><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>Dexter</b><small>Série</small><div class="badge">Série</div></div></div><div class="card"><div class="poster" style="background-image:url('')"></div><div class="info"><b>One Piece</b><small>Série</small><div class="badge">Série</div></div></div></div>
      <div class="empty" id="empty-series" style="display: none;">Nenhum título nesta aba.</div>
    </section>
    <section class="page" id="page-profile">
      <h1 class="page-title">👤 Perfil</h1>
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
    <button type="button" class="active" data-tab="lib"><span class="sym">📚</span>Biblioteca</button>
    <button type="button" data-tab="movies" class=""><span class="sym">🎬</span>Filmes</button>
    <button type="button" data-tab="series" class=""><span class="sym">📺</span>Séries</button>
    <button type="button" data-tab="profile" class=""><span class="sym">👤</span>Perfil</button>
  </nav>
</div>


<!-- genre drawer -->
<div class="drawer-backdrop" id="drawerBackdrop"></div>
<aside class="genre-drawer" id="genreDrawer" aria-hidden="true">
  <div class="drawer-head">
    <b>Gêneros</b>
    <button type="button" class="icon-btn" id="drawerClose" style="width:36px;height:36px;border-radius:50%;border:1px solid var(--line);background:var(--panel2);color:var(--text);cursor:pointer">✕</button>
  </div>
  <div class="drawer-body" id="drawerGenreList"><button type="button" class="drawer-item active"><span>✨</span> Todos</button><button type="button" class="drawer-item"><span>🏷</span> Ação</button><button type="button" class="drawer-item"><span>🏷</span> Aventura</button><button type="button" class="drawer-item"><span>🏷</span> Comédia</button><button type="button" class="drawer-item"><span>🏷</span> Drama</button><button type="button" class="drawer-item"><span>🏷</span> Terror</button><button type="button" class="drawer-item"><span>🏷</span> Romance</button><button type="button" class="drawer-item"><span>🏷</span> Ficção</button><button type="button" class="drawer-item"><span>🏷</span> Animação</button><button type="button" class="drawer-item"><span>🏷</span> Documentário</button><button type="button" class="drawer-item"><span>🏷</span> Suspense</button><button type="button" class="drawer-item"><span>🏷</span> Fantasia</button></div>
  <div class="drawer-foot">
    <button type="button" class="drawer-item" id="btnDrawerHome"><span>🏠</span> Início (Biblioteca)</button>
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
    <h3>👑 Área do criador</h3>
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
<div id="adminApp" class="on">
  <div class="admin-header">
    <div class="brand">Cine <span>Tung</span> · Criador</div>
    <div class="btns" style="margin:0">
      <button class="btn small secondary" type="button" id="btnBackViewer">← Voltar ao Cine Tung</button>
      <button class="btn small secondary" type="button" id="btnAdminMsg">Mensagens</button>
      <button class="btn small ok" type="button" id="btnCopyFullHtml">📋 Copiar HTML completo</button>
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
      <button type="button" id="btnPlay">▶️ Play/Pause</button>
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
<div class="toast show" id="toast">Montando HTML completo do Cine Tung…</div>

<script>
(function () {
  var KEY = "cinetung_catalog_v1";
  var HIST_KEY = "cinetung_history_v1";
  var PROFILE_KEY = "cinetung_profile_v1";
  var MSG_KEY = "cinetung_messages_v1";
  var SYNC_URL_KEY = "cinetung_sync_url_v1";
  var MSG_SYNC_URL_KEY = "cinetung_msg_sync_url_v1";
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
    "poster": "",
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
    "poster": "",
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
  var memoryCatalog = null;
  var memoryMsgs = null;
  function loadRaw() {
    if (memoryCatalog && Array.isArray(memoryCatalog) && memoryCatalog.length) return memoryCatalog;
    try {
      var list = JSON.parse(localStorage.getItem(KEY) || "[]");
      if (Array.isArray(list) && list.length) {
        memoryCatalog = list;
        return list;
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
          if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 8000) {
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
      try { localStorage.setItem(KEY + "_backup", JSON.stringify(catalog.map(function(it){ var o={}; for(var k in it)o[k]=it[k]; if(o.poster&&o.poster.length>8000&&o.poster.indexOf("data:")===0)o.poster=""; return o; }))); } catch (e2) {}
    }
    if (msg) toast(msg);
    else if (saved) { /* silent ok */ }
    try { renderAdminList(); } catch (e) {}
    try { renderGrid(); } catch (e) {}
  }
  function getSyncUrl() {
    return (localStorage.getItem(SYNC_URL_KEY) || "").trim();
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
  function fetchJsonFirstOk(urls) {
    var i = 0;
    function next() {
      if (i >= urls.length) return Promise.reject(new Error("Nenhuma URL respondeu"));
      var base = urls[i++];
      var u = base + (base.indexOf("?") >= 0 ? "&" : "?") + "t=" + Date.now();
      return fetch(u, { cache: "no-store", mode: "cors" })
        .then(function (r) {
          if (!r.ok) throw new Error("HTTP " + r.status);
          return r.json();
        })
        .catch(function () {
          // fallback XHR (alguns WebViews se comportam melhor)
          return new Promise(function (resolve, reject) {
            try {
              var xhr = new XMLHttpRequest();
              xhr.open("GET", u, true);
              xhr.setRequestHeader("Cache-Control", "no-cache");
              xhr.onload = function () {
                if (xhr.status >= 200 && xhr.status < 300) {
                  try { resolve(JSON.parse(xhr.responseText)); }
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
    return fetchJsonFirstOk(urls)
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

  /* splash */
  function endSplash() {
    var splash = $("splash");
    if (!splash) return;
    splash.classList.add("hide");
    setTimeout(function () { if (splash.parentNode) splash.parentNode.removeChild(splash); }, 500);
  }
  function runSplash() {
    var splash = $("splash");
    var logo = $("splashLogo");
    var credit = $("splashCredit");
    if (!splash) return;
    splash.addEventListener("click", endSplash);
    var word = "OBLIRY", i = 0;
    setTimeout(function type() {
      if (!$("splash")) return;
      if (i <= word.length) {
        if (logo) logo.textContent = word.slice(0, i);
        i++;
        setTimeout(type, 130);
      } else {
        if (credit) credit.classList.add("show");
        setTimeout(endSplash, 1600);
      }
    }, 700);
    setTimeout(endSplash, 3200);
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
      b.innerHTML = "<span>" + (g === "Todos" ? "✨" : "🏷") + "</span> " + escapeHtml(g);
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
      el.innerHTML =
        '<div class="poster" style="background-image:url(\'' + String(item.poster || "").replace(/'/g, "%27") + '\')"></div>' +
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
        "entrepeliculas", "grantorrent", "divxtotal"
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
    if (!url) return;
    url = String(url).trim();
    var opened = false;

    // 1) Cordova / InAppBrowser (alguns HTML→APK)
    try {
      if (window.cordova && window.cordova.InAppBrowser) {
        window.cordova.InAppBrowser.open(url, "_system");
        opened = true;
      } else if (window.SafariViewController) {
        // raramente presente
      }
    } catch (e) {}

    // 2) ponte Android comum em builders
    try {
      if (!opened && window.Android && typeof window.Android.openUrl === "function") {
        window.Android.openUrl(url); opened = true;
      }
      if (!opened && window.Android && typeof window.Android.openExternal === "function") {
        window.Android.openExternal(url); opened = true;
      }
      if (!opened && window.webkit && window.webkit.messageHandlers && window.webkit.messageHandlers.openUrl) {
        window.webkit.messageHandlers.openUrl.postMessage(url); opened = true;
      }
    } catch (e) {}

    // 3) window.open com alvos que alguns WebViews respeitam
    if (!opened) {
      try {
        var w = window.open(url, "_system");
        if (w) opened = true;
      } catch (e) {}
    }
    if (!opened) {
      try {
        var w2 = window.open(url, "_blank");
        if (w2) opened = true;
      } catch (e) {}
    }

    // 4) <a target=_blank> sintético
    if (!opened) {
      try {
        var a = document.createElement("a");
        a.href = url;
        a.target = "_blank";
        a.rel = "noopener noreferrer";
        a.style.display = "none";
        document.body.appendChild(a);
        a.click();
        setTimeout(function () { try { a.remove(); } catch (e) {} }, 500);
        opened = true; // assume tentativa
      } catch (e) {}
    }

    // 5) fallback forte em WebView: navega a própria janela (usuário volta com botão voltar do Android)
    // fazemos com confirmação visual via toast e delay curto
    try {
      toast("Abrindo link…");
    } catch (e) {}
    setTimeout(function () {
      try {
        // tenta top (sai do iframe/contexto)
        if (window.top && window.top !== window) {
          window.top.location.href = url;
        } else {
          window.location.href = url;
        }
      } catch (e) {
        try { window.location.assign(url); } catch (e2) {}
      }
    }, 120);
  }
  function renderExternalCard(stage, url, reason) {
    setNativeControlsVisible(false);
    stage.innerHTML = "";
    var box = document.createElement("div");
    box.style.cssText = "position:absolute;inset:0;display:flex;align-items:center;justify-content:center;padding:20px;background:#0c101a";
    box.innerHTML =
      '<div style="max-width:360px;width:100%;background:#121722;border:1px solid #243044;border-radius:16px;padding:20px;text-align:center">' +
      '<div style="font-size:28px;margin-bottom:10px">🌐</div>' +
      '<b style="display:block;font-size:16px;margin-bottom:8px">Assistir fora do player</b>' +
      '<p style="color:#97a3bb;font-size:13px;line-height:1.45;margin:0 0 14px">' + (reason || "Este site bloqueia o player embutido. Abra o link para assistir.") + '</p>' +
      '<p style="color:#6b7a99;font-size:11px;word-break:break-all;margin:0 0 16px">' + String(url).replace(/</g,"&lt;") + '</p>' +
      '<button type="button" id="btnExtOpen" style="width:100%;border:0;border-radius:12px;padding:14px;background:#6c5ce7;color:#fff;font-weight:800;font-size:15px;cursor:pointer">Assistir no navegador</button>' +
      '<button type="button" id="btnExtHere" style="width:100%;border:0;border-radius:12px;padding:12px;margin-top:8px;background:#2ed573;color:#06210f;font-weight:800;font-size:13px;cursor:pointer">Abrir nesta tela</button>' +
      '<button type="button" id="btnExtCopy" style="width:100%;border:0;border-radius:12px;padding:12px;margin-top:8px;background:#243049;color:#fff;font-weight:700;font-size:13px;cursor:pointer">Copiar link</button>' +
      '</div>';
    stage.appendChild(box);
    var b1 = document.getElementById("btnExtOpen");
    var bHere = document.getElementById("btnExtHere");
    var b2 = document.getElementById("btnExtCopy");
    if (b1) b1.onclick = function () { openExternalUrl(url); };
    if (bHere) bHere.onclick = function () {
      // navega o WebView inteiro para o site (funciona em quase todo HTML→APK)
      try { toast("Carregando site…"); } catch (e) {}
      setTimeout(function () {
        try { window.location.href = url; } catch (e) {
          try { window.location.assign(url); } catch (e2) {}
        }
      }, 80);
    };
    if (b2) b2.onclick = function () {
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(url).then(function () { toast("Link copiado"); }).catch(function () {
          prompt("Copie o link:", url);
        });
      } else {
        prompt("Copie o link:", url);
      }
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
      try { toast("Site bloqueou o embed · abrindo fora…"); } catch (e) {}
      // mesma lógica do "Abrir fora" / nesta tela
      try { $("overlay").classList.remove("open"); } catch (e) {}
      setTimeout(function () {
        try { window.location.href = url; } catch (e) {
          try { openExternalUrl(url); } catch (e2) {}
        }
      }, 150);
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
      try { toast("Abrindo site…"); } catch (e) {}
      // fecha overlay e navega o WebView inteiro
      try { $("overlay").classList.remove("open"); } catch (e) {}
      setTimeout(function () {
        try { window.location.href = url; } catch (e) {
          try { window.location.assign(url); } catch (e2) {
            openExternalUrl(url);
          }
        }
      }, 80);
      return;
    }

    // 5) Páginas de player web (Seekee, drives, embeds) e demais sites
    setNativeControlsVisible(false);
    var webPlayer = isWebPlayerPage(url);
    var isSeekee = false;
    try {
      var h = (new URL(url).hostname || "").toLowerCase();
      isSeekee = h.indexOf("seekee") >= 0 || h.indexOf("aiseekee") >= 0;
    } catch (e) {}

    var wrap = document.createElement("div");
    wrap.style.cssText = "position:absolute;inset:0;display:flex;flex-direction:column;background:#000";
    var bar = document.createElement("div");
    bar.style.cssText = "flex:0 0 auto;display:flex;align-items:center;gap:8px;padding:8px 10px;background:#0c101a;border-bottom:1px solid #243044;font-size:12px;color:#97a3bb";
    var titleSpan = document.createElement("span");
    titleSpan.style.flex = "1";
    titleSpan.textContent = isSeekee ? "Player Seekee" : (webPlayer ? "Player web" : "Site externo");
    bar.appendChild(titleSpan);

    var btnFs = document.createElement("button");
    btnFs.type = "button";
    btnFs.textContent = "⛶ Tela cheia";
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

    // se o iframe mostrar página de erro (ERR_BLOCKED...), abre fora sozinho
    watchIframeForBlockError(iframe, url);

    // tela cheia automática para sites que NÃO são Seekee
    if (!isSeekee) {
      setTimeout(requestStageFullscreen, 250);
    }
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
        var r = new FileReader();
        r.onload = function () {
          $("f_poster").value = r.result;
          $("bannerPreview").src = r.result;
        };
        r.readAsDataURL(f);
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
      saveRaw(list, "Publicado no app");
      renderEditor();
    };
    $("btnDraft").onclick = function () {
      var list = loadRaw();
      var it = null;
      for (var i = 0; i < list.length; i++) if (list[i].id === currentId) it = list[i];
      if (!it) return;
      readInto(it);
      it.published = false;
      saveRaw(list, "Rascunho salvo");
      renderEditor();
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
            if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 4000) {
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
      if (o.poster && typeof o.poster === "string" && o.poster.indexOf("data:") === 0 && o.poster.length > 4000) {
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
    try { toast("Montando HTML completo do Cine Tung…"); } catch (e) {}

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

        // remove data:image enormes que sobraram em qualquer lugar (senão clipboard zera)
        try {
          html = html.replace(/data:image\/[a-zA-Z0-9+.-]+;base64,[A-Za-z0-9+\/=]{500,}/g, "");
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

  // ao voltar pro app / foco → atualiza catálogo
  function onAppActive() {
    if ($("viewApp").classList.contains("on")) {
      try { renderGrid(); } catch (e) {}
    }
  }
  document.addEventListener("visibilitychange", function () {
    if (!document.hidden) onAppActive();
  });
  window.addEventListener("focus", onAppActive);
  window.addEventListener("online", onAppActive);

  // init (protegido — não trava na splash se algo falhar)
  try { ensureDemo(); } catch (e) { catalog = (EMBEDDED_CATALOG || []).slice(); memoryCatalog = catalog; }
  try { loadProfileUI(); } catch (e) {}
  try { setTab("lib"); } catch (e) {}
  try { renderGrid(); } catch (e) {}
  try { showViewer(); } catch (e) {
    try { $("viewApp").classList.add("on"); $("adminApp").classList.remove("on"); } catch (e2) {}
  }
  try { runSplash(); } catch (e) {
    try { endSplash(); } catch (e2) {}
  }
  // segurança: tira splash no máximo em 4s
  setTimeout(function () { try { endSplash(); } catch (e) {} }, 4000);
  /* pullCatalog desativado (sem GitHub) */
  /* auto-sync GitHub desativado */
})();
</script>



</body></html>
