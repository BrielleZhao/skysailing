```javascript
 //拖拽gif
    function addDragEvent() { //新建一个返回对象的函数
        var VVG = {};  //命名空间
        VVG.DOM = {
            $: function (id) { //创建方便的选择符
                return typeof id == "string" ? document.getElementById(id) : id;
            },
            bindEvent: function (node, type, func) { //事件绑定方法
                if (node.addEventListener) {
                    node.addEventListener(type, func, false);
                } else if (node.attachEvent) {
                    node.attachEvent("on" + type, func);
                } else {
                    node["on" + type] = func;
                }
            },
            getEvent: function (event) { //获取事件
                return event ? event : window.event;
            },
            getTarget: function (event) { //获取事件目标
                return event.target || event.srcElement;
            }
        }
        var DragDrop = function () { //新建一个返回对象的函数
            var box = VVG.DOM.$("imptActivitiesEnsu3dMapBox"); //获取外围BOX
            var dragBox = VVG.DOM.$("imptActivitiesEnsu3dMap");//获取需要拖动的BOX
            var dragging = null; //初始化对象
            var startX, startY;
            var ispressdown = null;
            function drag(event) { //事件执行函数
                event = VVG.DOM.getEvent(event);
                var target = VVG.DOM.getTarget(event);

                switch (event.type) {//判断事件类型
                    case "mousedown":
                        if (target.id == "imptActivitiesEnsu3dMap") { //当事件对象的ID等于要拖动的BOX的ID时
                            ispressdown = true;
                            dragging = target; //赋值到拖动目标
                            startX = event.clientX;
                            startY = event.clientY;
                        }
                        break;
                    case "mousemove":
                        if (dragging && ispressdown) { //当有拖动目标时执行

                            var moveX = event.clientX - startX;
                            var moveY = event.clientY - startY;
                            var imgtoleft = parseInt(dragging.style.left);//距离父容器边框距离
                            var imgtotop = 0;
                            var imgtoright = box.offsetWidth - (dragging.width + imgtoleft);
                            var imgtobottom = 0;
                            var paceleft = 0, pacetop = 0;
                            if (moveX < 0) {//向左
                                paceleft = moveX > imgtoright ? moveX : imgtoright;
                            }
                            else {
                                paceleft = moveX + imgtoleft < 0 ? moveX : (-imgtoleft);
                            }
                            dragging.style.left = parseInt(dragging.style.left) + paceleft + "px";
                        }
                        break;
                    case "mouseup":
                        // 清空拖动目标
                        ispressdown = false;
                        dragging = null;
                        startX = null;
                        startY = null;
                        break;
                }
            };
            return {
                dragStart: function () {
                    // 绑定事件
                    VVG.DOM.bindEvent(document, "mousedown", drag);
                    VVG.DOM.bindEvent(document, "mousemove", drag);
                    VVG.DOM.bindEvent(document, "mouseup", drag);
                }
            }
        }();
        DragDrop.dragStart();
    }
```
