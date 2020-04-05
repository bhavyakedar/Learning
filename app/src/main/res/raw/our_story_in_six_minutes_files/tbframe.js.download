/**
 * tbframe.js
 * Taboola frame injector
 * 23 Feb 2014
 * Effie NAdiv
 */

(function () {
    "use strict";
    var MAX_INT = 2147483648,
        publisher,
        currentQueue = [],
        processTimer,
        QUEUE_PROCESS_INTERVAL = 20,
        reIsAbsolute = /^(https?:)\/\//,

        /**
         * resolve relative url to absolute url
         * @param url - relative url
         * @returns - absolute url
         */
            createAbsoluteUrl = function (url) {
            var div = document.createElement("div"),
                encode = function (url) {
                    return url.split("&").join("&amp;").split("<").join("&lt;").split('"').join("&quot;");
                };

            if (reIsAbsolute.test(url)) {
                return url;
            }
            div.innerHTML = '<a href="{url}">.</a>'.supplant({url: encode(url)});
            return div.firstChild.href;
        },

        /**
         * creates an iframe and appends it to the container
         */
            createIframe = function (container) {
            var iframe = document.createElement('iframe');

            iframe.frameBorder = iframe.border = '0';
            iframe.scrolling = 'no';
            iframe.src = "javascript:void(0)";  // jshint ignore:line
            iframe.style.width = "100%";
            //iframe.style.height = "200px";
            container.innerHTML = "";
            return container.appendChild(iframe);
        },

        createLighthouse = function(key,  queue, container) {

            window[key] = {
                iframe:  createIframe(container),
                isIe: 1,
                queue: queue,
                onresize: function() {
                    frameResizer(window[key].iframe, window[key].isIe);
                },
                onrender: function() {
                    frameResizer(window[key].iframe, window[key].isIe);
                }
            };

        },

        /**
         * render a mode
         */
            createMode = function (message) {
            var key;

            if (validateMessages(message)) {
                whenContainerReady(message.container, function(container) {
                    key = genGuid();
                    message.container =  "tbl_widget_container";
                    message.publisher = message.publisher || publisher;
                    message.url = createAbsoluteUrl(message.url || getPageUrl());
                    message.referrer = message.referrer || getReferrer();
                    createLighthouse(key, createQueue(message), container);
                    writeIntoIframe(window[key].iframe, genIframeContent(key, message.publisher));
                }, 500);
            }
        },

        createQueue = function(message) {
            var i,
                p,
                o,
                result = [],
                pageLevel = ["url", "referrer"],
                mandatoryKey = getMandatoryKey(message);

            if (mandatoryKey) {
                pageLevel.push(mandatoryKey);
                for (i = 0; p = pageLevel[i]; i++) {    // jshint ignore:line
                    o = {};
                    o[p] = message[p];
                    result.push(o);
                    delete message[p];
                }
                result.push(message);
                result.push({flush:true});
            }
            return result;
        },

        /**
         * Finds an element inside the page <head> part
         * @param node  - the head node
         * @param tag  - the tag to look foor
         * @param attribute - element must contain this attribute
         * @param content - optional, the attribute must have the content passed
         * @returns the element we look for or null
         */
            findElementByTagAndAttribute = function (node, tag, attribute, content) {
            var list = node.getElementsByTagName(tag),
                len = list.length,
                i;

            content = content || "";
            for (i = 0; len > i; i++) {
                if (list[i].getAttribute(attribute) && (!content || content === list[i].getAttribute(attribute).toLowerCase())) {
                    return list[i];
                }
            }
            return void 0;
        },

        /**
         * Finds an element inside the <head> of the document
         * tag - the requested tag (meta, script, link...)
         * attribute - the tag must have this attribute
         * content - optional, the tag attribute must have this content
         * returns - the found element or null
         */
            findHeadElement = function (tag, attribute, content) {
            var i,
                head = document.getElementsByTagName("head"),
                len = head.length,
                element;


            if (head && head.length) {
                for (i = 0; len > i; i++) {
                    element = findElementByTagAndAttribute(head[i], tag, attribute, content);
                    if (element) {
                        return element;
                    }
                }
            }
            return void 0;
        },

        frameResizer = function(iframe, isIE) {
            var iframeWin = iframe.contentWindow || iframe.contentDocument.parentWindow,
                bodyHeight,
                docHeight;

            if (iframeWin.document.body) {
                bodyHeight = iframeWin.document.body.scrollHeight;
                docHeight = iframeWin.document.documentElement.scrollHeight;
                iframe.style.height = (isIE ? (bodyHeight || docHeight ) + 10 : Math.min(docHeight, bodyHeight) + (bodyHeight === docHeight ? 0 : 10)) + "px";
                if (isIE) {
                    iframe.style.height =  (iframeWin.document.documentElement.scrollHeight || iframeWin.document.body.scrollHeight) ;
                }
            }
        },


        genGuid = function () {
            return  'trc_lighthouse-' + Math.floor(Math.random() * MAX_INT).toString(36);
        },

        /**
         * creates the content of the iframe
         */
            genIframeContent = function(key, publisher) {
            var scriptTag = {open: "<" + "script>", close: "<" + "/script>"},
                text = [];

            text.push("<!doctype html>");

            text.push("<!--[if IE]><html class='ie'><![endif]-->");
            text.push("<!--[if !IE]><!-->");
            text.push(scriptTag.open);
            text.push("if (/*@cc_on!@*/false) {");
            text.push("document.documentElement.className='ie';");
            text.push("}");
            text.push(scriptTag.close);
            text.push("<!--<![endif]-->");

            text.push("<body>");
            text.push("<div id='tbl_widget_container'></div>");

            if (document.domain !== document.location.hostname) {
                text.push(scriptTag.open);
                text.push("document.domain='" + document.domain +  "';");
                text.push(scriptTag.close);
            }

            text.push(scriptTag.open);
            text.push("var _taboola = window.parent['" + key + "'].queue;" );
            text.push("function afterRender(){window.parent['" + key + "'].onrender()}");
            text.push("_taboola.push({onrender: afterRender})");
            text.push(scriptTag.close);

            text.push("<" + "script src='//cdn.taboola.com/libtrc/" + publisher + "/loader.js'>");
            text.push(scriptTag.close);

            text.push(scriptTag.open);
            text.push("window.parent['" + key + "'].isIe = document.documentElement.className == 'ie';");
            text.push("window.onresize = window.parent['" + key + "'].onresize;");
            text.push(scriptTag.close);

            text.push("</body>");

            return text.join("\n");
        },


        /**
         * get the iframe container element
         */
            getContainer = function (container) {
            return container && (typeof container === "string" ? document.getElementById(container) : container);
        },

        /**
         * get the currnet url
         * @returns {String} clean version of current url
         */
            getCurrentUrl = function () {
            // split is used to remove the queryString and the hash part of the current url   ?
            return document.location.href; //.split("?")[0].split("#")[0];
        },

        getMandatoryKey = (function(pageLevelAttributes) {
            return function(message) {
                var i,
                    attr;

                for (i = 0; attr = pageLevelAttributes[i]; i++) {  // jshint ignore:line
                    if (attr in message) {
                        return attr;
                    }
                }

            };
        }(["article", "video", "photo", "search", "category", "home"])),

        /**
         * returns page url
         * if exists  <meta http-equiv="tb:canonical" content="...  return content
         * if exists  <link rel="canonical" href="...  return href
         * if exists  <meta http-equiv="og:url" content="...  return content
         * otherwize return location.href
         * @returns String - unique url
         */
            getPageUrl = function () {
            var url = findHeadElement("meta", "property", "tb:canonical"),
                link,
                ogUrl;

            if (url && url.content) {
                return createAbsoluteUrl(url.content);
            }

            link = findHeadElement("link", "rel", "canonical");
            if (link && link.href) {
                return createAbsoluteUrl(link.href);
            }
            ogUrl = findHeadElement("meta", "property", "og:url");
            return ogUrl && ogUrl.content ? createAbsoluteUrl(ogUrl.content) : getCurrentUrl();
        },

        /**
         * returns the overriding referrer or the document's one
         * @returns {*}
         */
            getReferrer = function () {
            return document.referrer;
        },

        /**
         * is vArg is an array
         * @type {*|Function}
         */
            isArray = Array.isArray || function (vArg) {
            return Object.prototype.toString.call(vArg) === "[object Array]";
        },

        /**
         * print error messages if console.log is present
         */
            log = function (text) {
            if (window.console && console.log) {
                console.log("Taboola tbframe -> " + text);
            }
        },

        /**
         * handles _tbframe.push API
         * @param msg
         */
            pushMessage = function (msg) {
            var i;

            // clear previous calls
            if (processTimer) {
                clearTimeout(processTimer);
            }

            if (msg) {
                if (isArray(msg)) {
                    for (i = 0; i < msg.length; i++) {
                        pushMessage(msg[i]);
                        pushMessage({flush: true});
                    }
                    return;
                }

                if ("publisher" in msg) {
                    publisher = msg.publisher;
                }

                if ("flush" in msg && msg.flush) {
                    createMode(queueToObject(currentQueue));
                    currentQueue = [];
                    return;
                }
                currentQueue.push(msg);
            }

            // scedual a process operation
            // after QUEUE_PROCESS_INTERVAL ms of inactivity
            processTimer = setTimeout(function() {
                createMode(queueToObject(currentQueue));
                currentQueue = [];
            }, QUEUE_PROCESS_INTERVAL);

        },

        queueToObject = function(queue) {
            var i,
                message,
                p,
                result = {};

            for (i = 0; message = queue[i]; i++) {
                for (p in message) {
                    if (message.hasOwnProperty(p)) {
                        result[p] = message[p];
                    }
                }
            }
            return result;
        },

        /**
         * loop thru the queue at loading
         */
            startQueue = function () {
            var queue = window._tbframe = (window._tbframe || []);

            queue.push = pushMessage;
            while (queue.length) {
                pushMessage(queue.shift());
            }
        },

        /**
         * validate that a message has all mandatory members
         */
        validateMessages = (function () {
            var mandatories = ["placement", "mode", "container", "publisher"];

            return function (message) {
                var i, p, mandatoryKey = getMandatoryKey(message);

                if (!mandatoryKey || !message[mandatoryKey]) {
                    log("queue is missing a page level member such as video, article, photo, search, category or home.");
                    return false;
                }
                for (i = 0; p = mandatories[i]; i++) {   // jshint ignore:line
                    if (!message[p]) {
                        log("queue is missing a mandatory member: " + p);
                        return false;
                    }
                }
                return true;
            };
        }()),

        /**
         * retry finding the container
         */
            whenContainerReady = function(container, callback, tries) {
            var element = getContainer(container);

            if (element) {
                return callback(element);
            }

            tries = tries || 0;
            if (tries) {
                setTimeout(function() {
                    whenContainerReady(container, callback, tries - 1);
                }, 10);
            } else {
                log("Could not find container: '{container}'".supplant({container: container}));
            }
        },

    // inject content into iframe
        writeIntoIframe = function(iframe, text) {
            var idoc = iframe.contentDocument || iframe.contentWindow.document;

            idoc.write(text);

            // remember to close 'idoc' to prevent loading animation from spinning for ever.
            setTimeout(function(){
                try {
                    idoc.close();
                } catch (e) {
                }
            }, 0);
        };


    /**
     * general purpose substite template
     *
     * 'helo {name}'.supplant({name: 'John Christiansen'}) --> hello John Christiansen
     */
    String.prototype.supplant = function (o) {
        return this.replace(/{([^{}]*)}/g, function (a, b) {
            var r = o[b];
            return typeof r !== 'undefined' ? r : a;
        });
    };

    startQueue();
}());
