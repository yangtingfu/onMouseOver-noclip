# onMouseOver-noclip
jQuery鼠标悬停方向感知，穿墙效果
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<title>Examples</title>
<meta name="description" content="">
<meta name="keywords" content="">
<link href="" rel="stylesheet">
<script src="http://www.jq22.com/jquery/jquery-1.10.2.js"></script>
<script>
;(function($){
    $.fn.wear=function(options){
        //判断参数是否合法
        if(!isValid(options)){
            alert('参数不合法，请重新设置!');
            return this;
        }
        var opts = $.extend({},defaults,options)//覆盖默认参数
        , Num = opts.colNum*opts.rowNum, Tag = opts.tag, Sty = opts.style,tagSty = opts.tagStyle,boxWidth = this.width(),boxHeight=this.height();
        //返回对象 链式操作
        return this.each(function(index,ele){
            var _this = $(this);
            for(var i=0;i<Num;i++){
                var obj = $(Tag);//定义内部块标签
                
                //自定义标签内容
                var makeup = obj.html();
                makeup = $.fn.wear.format(makeup);
                Sty.border=Sty.border||'1px solid #fff';
                var borderW = Sty.border.match(/\d+px/g)[0].match(/\d+/g)[0];
                
                obj.html(makeup).css({
                    'border':Sty.border,
                    'width':boxWidth/opts.colNum-2*Sty.margin-2*Sty.padding-2*borderW,
                    'height':boxHeight/opts.rowNum-2*Sty.margin-2*Sty.padding-2*borderW,
                    'background':Sty.background,
                    'overflow':'hidden',
                    'float':'left',
                    'position':'relative',
                    'margin':Sty.margin,
                    'padding':Sty.padding,
                    'opacity':Sty.opacity,
                    'border-radius':Sty.radius
                });
                
                var mask_tag = $('<i>');//定义穿墙层
                mask_tag.addClass('mask');
                var makeup2 = mask_tag.html();
                makeup2 = $.fn.wear.formatMask(makeup2);
                mask_tag.html(makeup2).css({
                    'font-style':'normal',
                    'width':obj.outerWidth(),
                    'height':obj.outerHeight(),
                    'background':tagSty.background,
                    'position':'absolute',
                    'left':-borderW-obj.outerWidth(),
                    'top':-borderW,
                    'opacity':tagSty.opacity,
                    'border-radius':Sty.radius
                });
                obj.append(mask_tag);
                _this.append(obj);

            }
            _this.children().each(function(){
                __this = $(this);
                hoverGo(__this.get(0),{w:obj.outerWidth(),h:obj.outerHeight(),time:opts.time});
            })
        });
    };
    //设置默认参数
    var defaults={
        style:{
            margin:0,
            padding:0,
            border:'',
            radius:0,
            opacity:1,
            background:'#ccc'
        },
        colNum:4,
        rowNum:4,
        tag:'<div>',
        tagStyle:{
            
            opacity:0.6,
            background:'#000'
        },
        time:700
    };
    //公共的格式化方法
    $.fn.wear.format = function(str){
        return str;
    }
    $.fn.wear.formatMask = function(str){
        return str;
    }
    //公共方法
    function isValid(options){
        return !options||(options&&typeof options ==='object')?true:false;
    }
    
    function getStyle(obj,sName){
    return (obj.currentStyle||getComputedStyle(obj,false))[sName];
    }
    function startMove(obj,json,options){
    options=options||{};
    options.time = options.time||700;
    options.type = options.type||'ease-out';
    var start = {};
    var dis = {};
    for(var name in json){
        start[name] = parseFloat(getStyle(obj,name));
        if(isNaN(start[name])){
            switch(name){
                case 'top':
                    start[name] = obj.offsetTop;
                    break;
                case 'left':
                    start[name] = obj.offsetLeft;
                    break;
                case 'width':
                    start[name] = obj.offsetWidth;
                    break;
                case 'height':
                    start[name] = obj.offsetHeight;
                    break;
                case 'opacity':
                    start[name] = 1;
                    break;
                case 'borderWidth':
                    start[name] = 0;
                    break;
            }
        }
        dis[name] = json[name]-start[name];
    }
    var count = Math.floor(options.time/30);
    var n = 0;
    clearInterval(obj.timer);
    obj.timer = setInterval(function(){
        n++;
        for(var name in json){
            switch(options.type){
                case 'linear':
                    var cur = start[name]+dis[name]*n/count;
                    break;
                case 'ease-in':
                    var a = n/count;
                    var cur = start[name]+dis[name]*Math.pow(a,3);
                    break;
                case 'ease-out':
                    var a = 1-n/count;
                    var cur = start[name]+dis[name]*(1-Math.pow(a,3));
                    break;
            }
            if(name=='opacity'){
                obj.style.opacity=cur;
                obj.style.filter='alpha(opacity:'+cur*100+')';
            }else{
                obj.style[name] = cur+'px';
            }
        }
        if(n==count){
            clearInterval(obj.timer);
            options.end&&options.end();
        }
    },30);
    }

    function a2d(n){
        return n*180/Math.PI;
    }
    function hoverDir(obj,oEvent){
        var x = obj.offsetLeft+obj.offsetWidth/2-oEvent.clientX;
        var y = obj.offsetTop+obj.offsetHeight/2-oEvent.clientY;
        return Math.round((a2d(Math.atan2(y,x))+180)/90)%4;
    }

    function hoverGo(obj,json){
        var oS = obj.getElementsByClassName('mask')[0];
    obj.onmouseover=function(ev){
        var oEvent = ev||event;
        var oFrom = oEvent.fromElement||oEvent.relatedTarget;
        if(obj.contains(oFrom))return;
        var dir = hoverDir(obj,oEvent);
        switch(dir){
            case 0:
                oS.style.left = json.w+'px';
                oS.style.top = 0;
                break;
            case 1:
                oS.style.left = 0;
                oS.style.top = json.h+'px';
                break;
            case 2:
                oS.style.left = -json.w+'px';
                oS.style.top = 0;
                break;
            case 3:
                oS.style.left = 0;
                oS.style.top = -json.h+'px';
                break;
        }
        startMove(oS,{top:0,left:0},{time:json.time});
    };
    obj.onmouseout=function(ev){
        var oEvent = ev||event;
        var oTo = oEvent.toElement||oEvent.relatedTarget;
        if(obj.contains(oTo))return;
        var dir = hoverDir(obj,oEvent);
        switch(dir){
            case 0:
                startMove(oS,{left:json.w,top:0},{time:json.time});
                break;
            case 1:
                startMove(oS,{left:0,top:json.h},{time:json.time});
                break;
            case 2:
                startMove(oS,{left:-json.w,top:0},{time:json.time});
                break;
            case 3:
                startMove(oS,{left:0,top:-json.h},{time:json.time});
                break;
        }
   
    };
}
})(jQuery);
</script>
<style>
    *{
        margin:0;
        padding:0;
    }
</style>
<script>
    $(function(){
        //允许coder自定义自己的矩形内部结构
        $.fn.wear.format=function(str){
            return '<em>'+str+'</em>';
        }
        //允许coder自定义自己的遮罩层内部结构
        $.fn.wear.formatMask=function(str){
            return '<div>'+str+'</div>';
        }
        $('div').wear({
            style:{         //矩形样式
                margin:10,
                padding:10,
                border:'1px solid #000',
                radius:10,
                opacity:1,
                background:'#f56e6e'
            },
            
            colNum:4,//一行数量
            rowNum:4,//一列数量
            tag:'<span>',//定义矩形标签
            tagStyle:{      //遮罩层样式
                opacity:.8,
                background:'#000'
            },
            time:500 //遮罩层运动的速度,时间越大越慢
        })
    });
</script>
</head>
<body>
<div id="wear" style="width:600px;height:600px;margin:0px auto;"></div>
</body>
</html>
