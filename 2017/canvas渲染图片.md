 js创建canvas并设置图片
 ```javascript
 function setMapRendering(elemid) {
        var canvas = document.getElementById(elemid);
        var canvasheight = $("#imptActivitiesEnsu3dMapBox").height();
        var canvaswidth = $("#imptActivitiesEnsu3dMapBox").width();
        $("#" + elemid).attr("width", canvaswidth);//设置画布尺寸
        $("#" + elemid).attr("height", canvasheight);
        var width = canvasheight * 1920 / 1080;
        if (canvas.getContext) {
            var ctx = canvas.getContext("2d");
            var img = new Image();
            img.src = systemUtil.rootPath+"/static/Noh/Img/map.jpg";
            img.onload = function () {
                ctx.drawImage(img, -100, 0, width, canvasheight);
            //ctx.beginPath();
            //ctx.moveTo(30, 96);
            //ctx.lineTo(70, 66);
            //ctx.lineTo(103, 76);
            //ctx.lineTo(170, 15);
            //ctx.stroke();
            }  
        }
    }
```
