<html>
<head>
<script>
    var generateUniqueID = function () {
    return "v2-" + Date.now() + "-" + (Math.floor(Math.random() * 8999999999999) + 1e12)
},
    firstHiddenTime = -1,
    initHiddenTime = function () {
        return document.visibilityState === "hidden" ? 0 : Infinity
    },
    trackChanges = function () {
        onHidden(function (n) {
            var t = n.timeStamp;
            firstHiddenTime = t
        }, !0)
    },
    getVisibilityWatcher = function () {
        return firstHiddenTime < 0 && (window.__WEB_VITALS_POLYFILL__ ? (firstHiddenTime = window.webVitals.firstHiddenTime, firstHiddenTime === Infinity && trackChanges()) : (firstHiddenTime = initHiddenTime(), trackChanges()), onBFCacheRestore(function () {
            setTimeout(function () {
                firstHiddenTime = initHiddenTime();
                trackChanges()
            }, 0)
        })), {
            get firstHiddenTime() {
                return firstHiddenTime
            }
        }
    },
    getRating = function (n, t) {
        return n > t[1] ? "poor" : n > t[0] ? "needs-improvement" : "good"
    },
    bindReporter = function (n, t, i, r) {
        var u, f;
        return function (e) {
            t.value >= 0 && (e || r) && (f = t.value - (u || 0), (f || u === undefined) && (u = t.value, t.delta = f, t.rating = getRating(t.value, i), n(t)))
        }
    },
    onHidden = function (n, t) {
        var i = function (r) {
            (r.type === "pagehide" || document.visibilityState === "hidden") && (n(r), t && (removeEventListener("visibilitychange", i, !0), removeEventListener("pagehide", i, !0)))
        };
        addEventListener("visibilitychange", i, !0);
        addEventListener("pagehide", i, !0)
    },
    observe = function (n, t, i) {
        try {
            if (PerformanceObserver.supportedEntryTypes.includes(n)) {
                var r = new PerformanceObserver(function (n) {
                    Promise.resolve().then(function () {
                        t(n.getEntries())
                    })
                });
                return r.observe(Object.assign({
                    type: n,
                    buffered: !0
                }, i || {})), r
            }
        } catch (u) { }
        return
    },
    doubleRAF = function (n) {
        requestAnimationFrame(function () {
            return requestAnimationFrame(function () {
                return n()
            })
        })
    },
    FCPThresholds = [1800, 3e3],
    getFCP = function (n, t) {
        whenActivated(function () {
            var f = getVisibilityWatcher(),
                i = initMetric("FCP"),
                r, e = function (n) {
                    n.forEach(function (n) {
                        n.name === "first-contentful-paint" && (u.disconnect(), n.startTime < f.firstHiddenTime && (i.value = Math.max(n.startTime - getActivationStart(), 0), i.entries.push(n), r(!0)))
                    })
                },
                u = observe("paint", e);
            u && (r = bindReporter(n, i, FCPThresholds, t), onBFCacheRestore(function (u) {
                i = initMetric("FCP");
                r = bindReporter(n, i, FCPThresholds, t);
                doubleRAF(function () {
                    i.value = performance.now() - u.timeStamp;
                    r(!0)
                })
            }))
        })
    },
    getNavigationEntryFromPerformanceTiming = function () {
        var t = performance.timing,
            i = performance.navigation.type,
            r = {
                entryType: "navigation",
                startTime: 0,
                type: i == 2 ? "back_forward" : i === 1 ? "reload" : "navigate"
            };
        for (var n in t) n !== "navigationStart" && n !== "toJSON" && (r[n] = Math.max(t[n] - t.navigationStart, 0));
        return r
    },
    getNavigationEntry = function () {
        return window.__WEB_VITALS_POLYFILL__ ? window.performance && (performance.getEntriesByType && performance.getEntriesByType("navigation")[0] || getNavigationEntryFromPerformanceTiming()) : window.performance && performance.getEntriesByType && performance.getEntriesByType("navigation")[0]
    },
    bfcacheRestoreTime = -1,
    getBFCacheRestoreTime = function () {
        return bfcacheRestoreTime
    },
    onBFCacheRestore = function (n) {
        addEventListener("pageshow", function (t) {
            t.persisted && (bfcacheRestoreTime = t.timeStamp, n(t))
        }, !0)
    },
    getActivationStart = function () {
        var n = getNavigationEntry();
        return n && n.activationStart || 0
    },
    initMetric = function (n, t) {
        var r = getNavigationEntry(),
            i = "navigate";
        return getBFCacheRestoreTime() >= 0 ? i = "back-forward-cache" : r && (document.prerendering || getActivationStart() > 0 ? i = "prerender" : document.wasDiscarded ? i = "restore" : r.type && (i = r.type.replace(/_/g, "-"))), {
            name: n,
            value: typeof t == "undefined" ? -1 : t,
            rating: "good",
            delta: 0,
            entries: [],
            id: generateUniqueID(),
            navigationType: i
        }
    },
    reportedMetricIDs = {},
    LCPThresholds = [2500, 4e3],
    getLCP = function (n, t) {
        whenActivated(function () {
            var o = getVisibilityWatcher(),
                i = initMetric("LCP"),
                r, e = function (n) {
                    var t = n[n.length - 1];
                    t && t.startTime < o.firstHiddenTime && (i.value = Math.max(t.startTime - getActivationStart(), 0), i.entries = [t], r(!1))
                },
                u = observe("largest-contentful-paint", e),
                f;
            u && (r = bindReporter(n, i, LCPThresholds, t), f = runOnce(function () {
                reportedMetricIDs[i.id] || (e(u.takeRecords()), u.disconnect(), reportedMetricIDs[i.id] = !0, r(!0))
            }), ["keydown", "click"].forEach(function (n) {
                addEventListener(n, f, !0)
            }), onHidden(f), onBFCacheRestore(function (u) {
                i = initMetric("LCP");
                r = bindReporter(n, i, LCPThresholds, t);
                doubleRAF(function () {
                    i.value = performance.now() - u.timeStamp;
                    reportedMetricIDs[i.id] = !0;
                    r(!0)
                })
            }))
        })
    },
    runOnce = function (n) {
        var t = !1;
        return function (i) {
            t || (n(i), t = !0)
        }
    },
    CLSThresholds = [.1, .25],
    getCLS = function (n, t) {
        getFCP(runOnce(function () {
            var i = initMetric("CLS", 0),
                r, u = 0,
                f = [],
                e = function (n) {
                    n.forEach(function (n) {
                        if (!n.hadRecentInput) {
                            var t = f[0],
                                i = f[f.length - 1];
                            u && n.startTime - i.startTime < 1e3 && n.startTime - t.startTime < 5e3 ? (u += n.value, f.push(n)) : (u = n.value, f = [n])
                        }
                    });
                    u > i.value && (i.value = u, i.entries = f, r(!0))
                },
                o = observe("layout-shift", e);
            o && (r = bindReporter(n, i, CLSThresholds, t), onHidden(function () {
                e(o.takeRecords());
                r(!0)
            }), onBFCacheRestore(function () {
                u = 0;
                i = initMetric("CLS", 0);
                r = bindReporter(n, i, CLSThresholds, t);
                doubleRAF(function () {
                    return r()
                })
            }), setTimeout(r, 0))
        }))
    },
    whenActivated = function (n) {
        document.prerendering ? addEventListener("prerenderingchange", function () {
            return n()
        }, !0) : n()
    },
    interactionCountEstimate = 0,
    minKnownInteractionId = Infinity,
    maxKnownInteractionId = 0,
    updateEstimate = function (n) {
        n.forEach(function (n) {
            n.interactionId && (minKnownInteractionId = Math.min(minKnownInteractionId, n.interactionId), maxKnownInteractionId = Math.max(maxKnownInteractionId, n.interactionId), interactionCountEstimate = maxKnownInteractionId ? (maxKnownInteractionId - minKnownInteractionId) / 7 + 1 : 0)
        })
    },
    po, getInteractionCount = function () {
        return po ? interactionCountEstimate : performance.interactionCount || 0
    },
    initInteractionCountPolyfill = function () {
        "interactionCount" in performance || po || (po = observe("event", updateEstimate, {
            type: "event",
            buffered: !0,
            durationThreshold: 0
        }))
    },
    INPThresholds = [200, 500],
    prevInteractionCount = 0,
    getInteractionCountForNavigation = function () {
        return getInteractionCount() - prevInteractionCount
    },
    MAX_INTERACTIONS_TO_CONSIDER = 10,
    longestInteractionList = [],
    longestInteractionMap = {},
    processEntry = function (n) {
        var r = longestInteractionList[longestInteractionList.length - 1],
            t = longestInteractionMap[n.interactionId],
            i;
        (t || longestInteractionList.length < MAX_INTERACTIONS_TO_CONSIDER || n.duration > r.latency) && (t ? (t.entries.push(n), t.latency = Math.max(t.latency, n.duration)) : (i = {
            id: n.interactionId,
            latency: n.duration,
            entries: [n]
        }, longestInteractionMap[i.id] = i, longestInteractionList.push(i)), longestInteractionList.sort(function (n, t) {
            return t.latency - n.latency
        }), longestInteractionList.splice(MAX_INTERACTIONS_TO_CONSIDER).forEach(function (n) {
            delete longestInteractionMap[n.id]
        }))
    },
    estimateP98LongestInteraction = function () {
        var n = Math.min(longestInteractionList.length - 1, Math.floor(getInteractionCountForNavigation() / 50));
        return longestInteractionList[n]
    },
    getINP = function (n, t) {
        t = t || {};
        whenActivated(function () {
            initInteractionCountPolyfill();
            var i = initMetric("INP"),
                r, f = function (n) {
                    n.forEach(function (n) {
                        if (n.interactionId && processEntry(n), n.entryType === "first-input") {
                            var t = !longestInteractionList.some(function (t) {
                                return t.entries.some(function (t) {
                                    return n.duration === t.duration && n.startTime === t.startTime
                                })
                            });
                            t && processEntry(n)
                        }
                    });
                    var t = estimateP98LongestInteraction();
                    t && t.latency !== i.value && (i.value = t.latency, i.entries = t.entries, r(!0))
                },
                u = observe("event", f, {
                    durationThreshold: t.durationThreshold || 40
                });
            r = bindReporter(n, i, INPThresholds, t.reportAllChanges);
            u && ("interactionId" in PerformanceEventTiming.prototype && u.observe({
                type: "first-input",
                buffered: !0
            }), onHidden(function () {
                f(u.takeRecords());
                i.value < 0 && getInteractionCountForNavigation() > 0 && (i.value = 0, i.entries = []);
                r(!0)
            }), onBFCacheRestore(function () {
                longestInteractionList = [];
                prevInteractionCount = getInteractionCount();
                i = initMetric("INP");
                r = bindReporter(n, i, INPThresholds, t.reportAllChanges)
            }))
        })
    },
    windowCurrent = parent.window || window,
    WindowEvent, VisibilityType;
(function (n) {
    n.Load = "load";
    n.BeforeUnload = "beforeunload";
    n.Abort = "abort";
    n.Error = "error";
    n.Unload = "unload"
})(WindowEvent || (WindowEvent = {})),
    function (n) {
        n[n.Focus = 0] = "Focus";
        n[n.Blur = 1] = "Blur"
    }(VisibilityType || (VisibilityType = {}));
var AjaxTiming = function () {
    function n(n, t, i, r) {
        var u = this;
        this.getPerformanceTimings = function (n) {
            u.connect = n.connectEnd - n.connectStart;
            u.dns = n.domainLookupEnd - n.domainLookupStart;
            u.duration = n.duration;
            u.load = n.responseEnd - n.responseStart;
            u.wait = n.responseStart - n.requestStart;
            u.start = n.startTime;
            u.redirect = n.redirectEnd - n.redirectStart;
            n.secureConnectionStart && (u.ssl = n.connectEnd - n.secureConnectionStart)
        };
        this.url = n;
        this.method = t;
        this.isAsync = i;
        this.open = r
    }
    return n
}(),
    ProfilerJsError = function () {
        function n(n, t, i) {
            this.count = 0;
            this.message = n;
            this.url = t;
            this.lineNumber = i
        }
        return n.createText = function (n, t, i) {
            return [n, t, i].join(":")
        }, n.prototype.getText = function () {
            return n.createText(this.message, this.url, this.lineNumber)
        }, n
    }(),
    ProfilerEventManager = function () {
        function n() {
            this.events = [];
            this.hasAttachEvent = !!window.attachEvent
        }
        return n.prototype.add = function (n, t, i) {
            this.events.push({
                type: n,
                target: t,
                func: i
            });
            this.hasAttachEvent ? t.attachEvent("on" + n, i) : t.addEventListener(n, i, !1)
        }, n.prototype.remove = function (n, t, i) {
            this.hasAttachEvent ? t.detachEvent(n, i) : t.removeEventListener(n, i, !1);
            var r = this.events.indexOf({
                type: n,
                target: t,
                func: i
            });
            r !== 1 && this.events.splice(r, 1)
        }, n.prototype.clear = function () {
            for (var n, i = this.events, t = 0; t < i.length; t++) n = i[t], this.remove(n.type, n.target, n.func);
            this.events = []
        }, n
    }(),
    AjaxRequestsHandler = function () {
        function n() {
            var t = this;
            this.fetchRequests = [];
            this.fetchEntriesIndices = {};
            this.compareEntriesDelay = 100;
            this.hasPerformance = typeof performance == "object" && typeof window.performance.now == "function" && typeof window.performance.getEntriesByType == "function";
            this.captureFetchRequests = function () {
                var n = [],
                    i = t,
                    r = function (n) {
                        return n
                    },
                    u = function (n) {
                        return Promise.reject(n)
                    };
                window.fetch && (window.fetch = function (t) {
                    return function () {
                        for (var o, f, s = [], e = 0; e < arguments.length; e++) s[e] = arguments[e];
                        return o = 0, f = Promise.resolve(s), f = f.then(function (t) {
                            var r, u = {},
                                e, f;
                            if (t.length && t.length >= 1) r = t[0], t.length > 1 && (u = t[1]);
                            else return [];
                            return e = "GET", u.method && (e = u.method), o = n.length, f = "", f = typeof r != "object" || !r ? r : Array.isArray(r) && r.length > 0 ? r[0] : r.url, f && n.push(new AjaxTiming(f, e, !0, i.now())), [r, u]
                        }, r), f = f.then(function (n) {
                            return t.apply(void 0, n)
                        }), f.then(function (t) {
                            var r = n[o],
                                u = i.fetchRequests;
                            return i.processPerformanceEntries(r, u), t
                        }, u)
                    }
                }(window.fetch))
            };
            this.captureFetchRequests();
            n.startAjaxCapture(this)
        }
        return n.prototype.getAjaxRequests = function () {
            return this.fetchRequests
        }, n.prototype.clear = function () {
            this.fetchRequests = []
        }, n.prototype.now = function () {
            return this.hasPerformance ? window.performance.now() : (new Date).getTime()
        }, n.prototype.processPerformanceEntries = function (n, t) {
            var i = this;
            setTimeout(function () {
                var f, o, s, h, e;
                if (i.hasPerformance) {
                    var u = n.url,
                        r = [],
                        c = performance.getEntriesByType("resource");
                    for (f = 0, o = c; f < o.length; f++) s = o[f], s.name === u && r.push(s);
                    if (t.push(n), r.length !== 0) {
                        if (i.fetchEntriesIndices[u] || (i.fetchEntriesIndices[u] = []), r.length === 1) {
                            n.getPerformanceTimings(r[0]);
                            i.fetchEntriesIndices[u].push(0);
                            return
                        }
                        h = i.fetchEntriesIndices[u];
                        for (e in r)
                            if (h.indexOf(e) === -1) {
                                n.getPerformanceTimings(r[e]);
                                h.push(e);
                                return
                            } n.getPerformanceTimings(r[0])
                    }
                }
            }, i.compareEntriesDelay)
        }, n.startAjaxCapture = function (n) {
            var t = XMLHttpRequest.prototype,
                r = t.open,
                u = t.send,
                i = [];
            n.hasPerformance && typeof window.performance.setResourceTimingBufferSize == "function" && window.performance.setResourceTimingBufferSize(300);
            t.open = function (t, u, f, e, o) {
                this.rpIndex = i.length;
                i.push(new AjaxTiming(u, t, f, n.now()));
                r.call(this, t, u, f === !1 ? !1 : !0, e, o)
            };
            t.send = function (t) {
                var r = this,
                    e = this.onreadystatechange,
                    f;
                (this.onreadystatechange = function (t) {
                    var u = i[r.rpIndex],
                        o, f;
                    if (u) {
                        o = r.readyState;
                        f = !!(r.response && r.response !== null && r.response !== undefined);
                        switch (o) {
                            case 1:
                                u.connectionEstablished = n.now();
                                break;
                            case 2:
                                u.requestReceived = n.now();
                                break;
                            case 3:
                                u.processingTime = n.now();
                                break;
                            case 4:
                                u.complete = n.now();
                                switch (r.responseType) {
                                    case "text":
                                    case "":
                                        typeof r.responseText == "string" && (u.responseSize = r.responseText.length);
                                        break;
                                    case "json":
                                        f && typeof r.response.toString == "function" && (u.responseSize = r.response.toString().length);
                                        break;
                                    case "arraybuffer":
                                        f && typeof r.response.byteLength == "number" && (u.responseSize = r.response.byteLength);
                                        break;
                                    case "blob":
                                        f && typeof r.response.size == "number" && (u.responseSize = r.response.size)
                                }
                                n.processPerformanceEntries(u, n.fetchRequests)
                        }
                        typeof e == "function" && e.call(r, t)
                    }
                }, f = i[this.rpIndex], f) && (t && !isNaN(t.length) && (f.sendSize = t.length), f.send = n.now(), u.call(this, t))
            }
        }, n
    }(),
    RProfiler = function () {
        function n() {
            function r(n) {
                var i = n.target || n.srcElement;
                return i.nodeType == 3 && (i = i.parentNode), t("N/A", i.src || i.URL, -1), !1
            }
            var n = this,
                t, i;
            this.restUrl = "g.3gl.net/jp/906/v3.3.9/M";
            this.startTime = (new Date).getTime();
            this.eventsTimingHandler = new EventsTimingHandler;
            this.inputDelay = new InputDelayHandler;
            this.version = "v3.3.9";
            this.info = {};
            this.hasInsight = !1;
            this.data = {
                start: this.startTime,
                jsCount: 0,
                jsErrors: [],
                loadTime: -1,
                loadFired: window.document.readyState == "complete"
            };
            this.eventManager = new ProfilerEventManager;
            this.setCLS = function (t) {
                var i = t.name,
                    r = t.delta,
                    u = i === "CLS" ? r : undefined;
                n.cls = u
            };
            this.setLCP = function (t) {
                var i = t.name,
                    r = t.delta,
                    u = i === "LCP" ? r : undefined;
                n.lcp = u
            };
            this.setINP = function (t) {
                var i = t.name,
                    r = t.value,
                    u = i === "INP" ? r : undefined;
                    console.log(t.entries, t.entries[0].name,"Entries length : "+t.entries.length,u,i,r);
                    window.inpEventName = t.entries[0].name;
                n.inp = u
            };
            this.recordPageLoad = function () {
                n.data.loadTime = (new Date).getTime();
                n.data.loadFired = !0
            };
            this.addError = function (t, i, r) {
                var s, f, u, e, o;
                for (n.data.jsCount++, s = ProfilerJsError.createText(t, i, r), f = n.data.jsErrors, u = 0, e = f; u < e.length; u++)
                    if (o = e[u], o.getText() == s) {
                        o.count++;
                        return
                    } f.push(new ProfilerJsError(t, i, r))
            };
            this.getAjaxRequests = function () {
                return n.ajaxHandler.getAjaxRequests()
            };
            this.clearAjaxRequests = function () {
                n.ajaxHandler.clear()
            };
            this.addInfo = function (t, i, r) {
                if (!n.isNullOrEmpty(t)) {
                    if (n.isNullOrEmpty(r)) n.info[t] = i;
                    else {
                        if (n.isNullOrEmpty(i)) return;
                        n.isNullOrEmpty(n.info[t]) && (n.info[t] = {});
                        n.info[t][i] = r
                    }
                    n.hasInsight = !0
                }
            };
            this.clearInfo = function () {
                n.info = {};
                n.hasInsight = !1
            };
            this.clearErrors = function () {
                n.data.jsCount = 0;
                n.data.jsErrors = []
            };
            this.getInfo = function () {
                return n.hasInsight ? n.info : null
            };
            this.getEventTimingHandler = function () {
                return n.eventsTimingHandler
            };
            this.getInputDelay = function () {
                return n.inputDelay
            };
            this.getCPWebVitals = function () {
                return getCLS(n.setCLS, !1), getLCP(n.setLCP, !1), getINP(n.setINP, {
                    reportAllChanges: !1
                }), {
                    cls: n.cls,
                    lcp: n.lcp,
                    inp: n.inp
                }
            };
            this.attachIframe = function () {
                var r = window.location.protocol,
                    t = document.createElement("iframe"),
                    i;
                t.src = "about:blank";
                i = t.style;
                i.position = "absolute";
                i.top = "-10000px";
                i.left = "-1000px";
                t.addEventListener("load", function (t) {
                    var u = t.currentTarget,
                        f, i;
                    u && u.contentDocument && (f = u.contentDocument, i = f.createElement("script"), i.type = "text/javascript", i.src = r + "//" + n.restUrl, f.body.appendChild(i))
                });
                document.body && document.body.insertAdjacentElement("afterbegin", t)
            };
            this.eventManager.add(WindowEvent.Load, window, this.recordPageLoad);
            t = this.addError;
            this.ajaxHandler = new AjaxRequestsHandler;
            getCLS(this.setCLS, !1);
            getLCP(this.setLCP, !1);
            getINP(this.setINP, {
                reportAllChanges: !1
            });
            window.opera ? this.eventManager.add(WindowEvent.Error, document, r) : "onerror" in window && (i = window.onerror, window.onerror = function (n, r, u) {
                return (t(n, r, u), !!i) ? i(n, r, u) : !1
            });
            !window.__cpCdnPath || (this.restUrl = window.__cpCdnPath.trim())
        }
        return n.prototype.isNullOrEmpty = function (n) {
            if (n === undefined || n === null) return !0;
            if (typeof n == "string") {
                var t = n;
                return t.trim().length == 0
            }
            return !1
        }, n.prototype.dispatchCustomEvent = function (n) {
            (function (n) {
                function t(n, t) {
                    t = t || {
                        bubbles: !1,
                        cancelable: !1,
                        detail: undefined
                    };
                    var i = document.createEvent("CustomEvent");
                    return i.initCustomEvent(n, t.bubbles, t.cancelable, t.detail), i
                }
                if (typeof n.CustomEvent == "function") return !1;
                t.prototype = Event.prototype;
                n.CustomEvent = t
            })(window);
            var t = new CustomEvent(n);
            window.dispatchEvent(t)
        }, n
    }(),
    InputDelayHandler = function () {
        function n() {
            var n = this;
            this.firstInputDelay = 0;
            this.firstInputTimeStamp = 0;
            this.startTime = 0;
            this.delay = 0;
            this.profileManager = new ProfilerEventManager;
            this.eventTypes = ["click", "mousedown", "keydown", "touchstart", "pointerdown",];
            this.addEventListeners = function () {
                n.eventTypes.forEach(function (t) {
                    n.profileManager.add(t, document, n.onInput)
                })
            };
            this.now = function () {
                return (new Date).getTime()
            };
            this.removeEventListeners = function () {
                n.eventTypes.forEach(function (t) {
                    n.profileManager.remove(t, document, n.onInput)
                })
            };
            this.onInput = function (t) {
                var i, r, u;
                t.cancelable && (i = t.timeStamp > 1e12, n.firstInputTimeStamp = n.now(), r = i || !window.performance, u = r ? n.firstInputTimeStamp : window.performance.now(), n.delay = u - t.timeStamp, t.type == "pointerdown" ? n.onPointerDown() : (n.removeEventListeners(), n.updateFirstInputDelay()))
            };
            this.onPointerUp = function () {
                n.removeEventListeners();
                n.updateFirstInputDelay()
            };
            this.onPointerCancel = function () {
                n.removePointerEventListeners()
            };
            this.removePointerEventListeners = function () {
                n.profileManager.remove("pointerup", document, n.onPointerUp);
                n.profileManager.remove("pointercancel", document, n.onPointerCancel)
            };
            this.updateFirstInputDelay = function () {
                n.delay >= 0 && n.delay < n.firstInputTimeStamp - n.startTime && (n.firstInputDelay = Math.round(n.delay))
            };
            this.startSoftNavigationCapture = function () {
                n.resetSoftNavigationCapture()
            };
            this.resetSoftNavigationCapture = function () {
                n.resetFirstInputDelay();
                n.addEventListeners()
            };
            this.resetFirstInputDelay = function () {
                n.delay = 0;
                n.firstInputDelay = 0;
                n.startTime = 0;
                n.firstInputTimeStamp = 0
            };
            this.startTime = this.now();
            this.addEventListeners()
        }
        return n.prototype.onPointerDown = function () {
            this.profileManager.add("pointerup", document, this.onPointerUp);
            this.profileManager.add("pointercancel", document, this.onPointerCancel)
        }, n.prototype.getFirstInputDelay = function () {
            return this.firstInputDelay
        }, n
    }(),
    EventsTimingHandler = function () {
        function n() {
            var n = this;
            this.hiddenStrings = ["hidden", "msHidden", "webkitHidden", "mozHidden"];
            this.visibilityStrings = ["visibilitychange", "msvisibilitychange", "webkitvisibilitychange", "mozvisibilitychange"];
            this.captureSoftNavigation = !1;
            this.hidden = "hidden";
            this.visibilityChange = "visibilitychange";
            this.visibilityEvents = [];
            this.eventManager = new ProfilerEventManager;
            this.engagementTimeIntervalMs = 1e3;
            this.engagementTime = 0;
            this.firstEngagementTime = 0;
            this.lastEventTimeStamp = 0;
            this.timeoutId = undefined;
            this.startTime = (new Date).getTime();
            this.now = function () {
                return (new Date).getTime()
            };
            this.startVisibilityCapture = function () {
                n.initializeVisibilityProperties();
                document.addEventListener(n.visibilityChange, n.captureFocusEvent, !1)
            };
            this.initializeVisibilityProperties = function () {
                for (var r = n.hiddenStrings, i = 0, t = 0; t < r.length; t++) typeof document[r[t]] != "undefined" && (i = t);
                n.visibilityChange = n.visibilityStrings[i];
                n.hidden = n.hiddenStrings[i]
            };
            this.captureFocusEvent = function () {
                n.updateVisibilityChangeTime();
                document[n.hidden] || n.captureEngagementTime()
            };
            this.updateVisibilityChangeTime = function () {
                document[n.hidden] ? n.captureVisibilityEvent(VisibilityType.Blur) : n.captureVisibilityEvent(VisibilityType.Focus)
            };
            this.onBlur = function () {
                n.captureVisibilityEvent(VisibilityType.Blur)
            };
            this.onFocus = function () {
                n.captureVisibilityEvent(VisibilityType.Focus)
            };
            this.captureVisibilityEvent = function (t) {
                n.visibilityEvents.push({
                    type: t,
                    time: n.now()
                })
            };
            this.captureEngagementTime = function (t) {
                if (t === void 0 && (t = !0), !n.lastEventTimeStamp) {
                    n.engagementTime = n.engagementTimeIntervalMs;
                    n.lastEventTimeStamp = n.now();
                    return
                }
                var i = n.now() - n.lastEventTimeStamp;
                if (n.lastEventTimeStamp = n.now(), t && n.firstEngagementTime === 0 && (n.firstEngagementTime = n.now()), i > 0 && i < n.engagementTimeIntervalMs) {
                    clearTimeout(n.timeoutId);
                    n.engagementTime += i;
                    return
                }
                n.startTimer()
            };
            this.captureMouseMove = function () {
                n.captureEngagementTime(!1)
            };
            this.startTimer = function () {
                n.timeoutId = setTimeout(function () {
                    n.engagementTime += n.engagementTimeIntervalMs
                }, n.engagementTimeIntervalMs)
            };
            this.getFocusAwayTime = function () {
                var i = n.visibilityEvents,
                    t = -1,
                    s, h, o;
                if (i.length === 0) return 0;
                for (var r = t, u = 0, f = t, e = 0; u < i.length;) i[u].type === VisibilityType.Blur && r === t && (r = u), s = f === t && r !== t, i[u].type === VisibilityType.Focus && s && (f = u), h = r !== t && f !== t, h && (o = i[f].time - i[r].time, o > 0 && (e += o), r = t, f = t), u = u + 1;
                return r === i.length - 1 && (e += n.now() - i[r].time), e
            };
            this.getEngagementTime = function () {
                return n.engagementTime
            };
            this.getStartTime = function () {
                return n.startTime
            };
            this.getFirstEngagementTime = function () {
                return n.firstEngagementTime
            };
            this.startSoftNavigationCapture = function () {
                n.captureSoftNavigation = !0
            };
            this.resetSoftNavigationCapture = function () {
                n.resetEngagementMetrics();
                n.visibilityEvents = []
            };
            this.resetEngagementMetrics = function () {
                n.engagementTime = 0;
                n.lastEventTimeStamp = n.now();
                n.firstEngagementTime = 0
            };
            this.clear = function () {
                n.eventManager.clear()
            };
            this.captureEngagementTime(!1);
            this.eventManager.add("scroll", document, this.captureEngagementTime);
            this.eventManager.add("resize", window, this.captureEngagementTime);
            this.eventManager.add("mouseup", document, this.captureEngagementTime);
            this.eventManager.add("keyup", document, this.captureEngagementTime);
            this.eventManager.add("mousemove", document, this.captureMouseMove);
            this.eventManager.add("focus", window, this.onFocus);
            this.eventManager.add("blur", window, this.onBlur);
            this.eventManager.add("focus", document, this.onFocus);
            this.eventManager.add("blur", document, this.onBlur)
        }
        return n
    }(),
    profiler = new RProfiler;
window.RProfiler = profiler;
window.WindowEvent = WindowEvent;
function init() { window.RProfiler.addInfo('tracepoint', 'bltoken', window.inpEventName); }    window.RProfiler ? init() : window.addEventListener("GlimpseLoaded", init);
profiler.dispatchCustomEvent("GlimpseLoaded");
document.onreadystatechange = function () {
    document.readyState === "complete" && profiler.attachIframe()
};
    </script>


</head>
<body>
<h1>The button Element</h1>

<button type="button" onclick="alert('Hello world!')">Click Me!</button>
<p><a href="https://www.w3schools.com/">Visit W3Schools.com!</a></p>
 
<h1>Heading that works post adding RUM tag added button</h1>
<p>test paragraph for INP post adding RUM tag added button</p>

</body>
</html>
