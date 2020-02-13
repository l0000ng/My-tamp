# My-tamp
油猴插件
// ==UserScript==
// @name       queryFunds
// @namespace  http://use.i.E.your.homepage/
// @version    0.1
// @icon       https://www.zlfund.cn/favicon.ico
// @description  query funds from sina
// @include    http://www.baidu.com
// @include     /https://(\w+\.)?zlfund\.cn\/(trade|fund)\//
// @include     /http://(\w+\.)?jjmmw\.com\/fund\//
// @include     /.*funds?.*/
// @match      http://www.baidu.com
// @run-at      document-end
// @require     http://ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js
// @require     https://raw.github.com/cswuxiang/lib/master/js/core/qufix-1.0.js?v=20131016
// @require     https://raw.github.com/cswuxiang/lib/master/js/ui/drag.js?v=20131015
// @copyright  2012+, You
// ==/UserScript==


var onManual = true;//是否开启手动配置
var codes = ["040015","100026","233007","260104","519033","519679"];
var invest = [2000,2000,2000,2000,2000,2000,2000];
var interval = 1000*10;//每隔多少时间刷新
var timer = null;






GM_addStyle('\
.bg9 {\
background-color: #999999;\
}\
.f10 {\
font-size: 10px;\
}\
.dib {\
display: inline-block;\
}\
.w150 {\
width: 150px;\
}\
.w100 {\
width: 100px;\
}\
.w50 {\
width: 50px;\
}\
ul, ol {\
list-type: none;\
list-style-type: none;\
margin: 0;\
padding-left: 0;\
}\
');


var templateLi = ["<li>",
                  "<span class='dib w150'>{name}</span>",
                    "<span class='dib w150'>{time}</span>",
                  "<span class='dib w50' style=\"color:{color};\">{per}</span>",
                    "<span class='dib w50' style=\"color:{color};\">{money}</span>",
                  "</li>"].join("");    
var dragEl = null;


function creatDragEl(){


    var wrap = ["<div id=\"drag\" style=\"z-index:1000;cursor: move;position:absolute;top:0px;padding:10px;text-align:left;\" class=\"bg9\">",
                "<div id=\"main-content\" style=\"display:none;\">",
            "<ul id=\"itemjj\" class=\"f10\"></ul>",
            "<div style=\"margin-top:20px;\">总的收获金额:<span style=\"float:right;margin-right:30px;\" id=\"total\"></span></div>",
            "</div>",
                "<div id=\"loading\">加载中........</div>",
"</div>"].join("");
    dragEl = $(wrap);
    dragEl.appendTo($('body'));
    
}


function makeMove(el){
    $T.enableSelect( el,false);
new drag(el);
    
}


    
var url = "http://hq.sinajs.cn/?_=1381628403887/list=fu_{code},f_{code}";
var arrdata = [];


//组装url参数
function getParam(code){
return $str.format(url,{code:code});
}
//请求数据
function loadData(){
    
    for(var i = 0;i<codes.length;i++){
 
$T.loadJS(getParam(codes[i]),(function(i){
return function(){
suc(codes[i],i);
}
})(i));
 
     }
    
}
//处理返回来的数据
function suc(code,i){


    var data = {};
    var splits = eval(('hq_str_fu_'+code)).split(",");
    var percent = 0;
    if(i==0){
        arrdata = [];
    }

data['name'] = splits[0];
data['time'] = splits[7]+" "+splits[1];
    percent = parseFloat(parseFloat(splits[6]).toFixed(2));
    data['per'] = percent + "%";


data['money'] = Math.floor(percent*invest[i]/100);
    data['color'] = data['money'] > 0 ? 'red' : 'green';
arrdata.push(data);
    
    if(i === codes.length-1 && arrdata.length === codes.length){
        render(arrdata);
    }
 }
function render(arr){
    arr = arr || [];
    var strs = [],total = 0;
    for(var i=0;i<arr.length;i++){   
       total += arr[i].money;
       strs.push($str.format(templateLi,arr[i]));
   }
 
   document.getElementById("itemjj").innerHTML = strs.join("");
   document.getElementById("total").innerHTML = total;
   document.getElementById("main-content").style.display="";
   document.getElementById("loading").style.display="none";
    
}


var queryURL =  "http://my.jjmmw.com/fundbook/index.jsonp?callback={callback}&book_id={id}";//
var selList = document.getElementById("sel_booklist");


unsafeWindow.jsonp123 = function(data){
    
    for(var i=0;i<data.items.length;i++){
        codes.push(data.items[i].fundcode);
        invest.push(data.items[i].state_json.money);
    }
     //定时请求
    timer = setInterval(function(){
        loadData();
    },interval);
}
function queryMyFunds(){
    var selValue = selList.value;
    $T.loadJS($str.format(queryURL,{id:selValue,callback:'jsonp123'}));  


}






(function(){
    
  creatDragEl();
makeMove(dragEl.get(0));
    if(onManual){
        timer = setInterval(function(){
            loadData();
        },interval);
    }else{//定时请求
        $Event.addEvent(selList,"onchange",function(){
        clearInterval(timer);
    queryMyFunds();
    });
        queryMyFunds();
    }
})()
