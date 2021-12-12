---
title: easyui控件
author: 不好听
img: /2018/11/26/easyui-shi-jian-kong-jian-xuan-ze-yue-fen/images/easyui.PNG
top: false
categories: easyui
tags:
  - easyui
  - datebox
date: 2018-11-26 19:30:14
---
### 1.easyui-datebox
截止到目前的版本（[1.6.10](http://www.jeasyui.com/demo/main/index.php?plugin=DateBox)）,easyui并没有提供一种选择月份的时间控件，但是在项目中却会经常有此类需求。  
下面提供一种解决办法，这种实现方式利用了easyui的onShowPanel、parser、formatter方法。

首先，写一个时间输入框：
`<input class="easyui-datebox" id="monthSpan" style="width:150px;"data-options="editable:false">`  
然后，在javascript代码中，加入：  
{% codeblock %} 
var p = $('#monthSpan').datebox('panel'),
	tds = false,
	span = p.find('span.calendar-text');
$('#monthSpan').datebox({    
	onShowPanel: function() {
		span.trigger('click');
		if (!tds)    
			setTimeout(function() {
				tds = p.find('div.calendar-menu-month-inner td');
               	tds.click(function(e) {    
                   	e.stopPropagation();
                	var year = /\d{4}/.exec(span.html())[0], month = parseInt($(this).attr('abbr'), 10) + 1;    
                   	$('#monthSpan').datebox('hidePanel').datebox('setValue', year + '-' + month);
               	});    
			}, 0);    
	},    
	parser: function(s) {
		if (!s)    
    		return new Date();    
    	var arr = s.split('-');    
    	return new Date(parseInt(arr[0], 10), parseInt(arr[1], 10) - 1, 1);    
	},    
	formatter: function(d) {    
		var month = d.getMonth();  
		if(month<=10){  
			month = "0"+month;  
		}  
		if (d.getMonth() == 0) {    
			return d.getFullYear()-1 + '-' + 12;    
		} else {    
			return d.getFullYear() + '-' + month;    
		}    
	}
});   
{% endcodeblock %}  

最终的效果如下图所示：  
![example](images/1126.png "选择月份")   

### 2.easyui-datetimebox  
下面修改控件默认的按钮事件： 
 先写html：   
 `<input id="beginTime" class="easyui-datetimebox"  editable=false />` 
 再写js事件：   
{% codeblock %} 
 setJinTianTime('#beginTime');
 
function setJinTianTime(dom){
	var today = function (){
	    //当前的时间
	    var dd = new Date();
	    var y = dd.getFullYear();  
	    var m = dd.getMonth()+1;  
	    var d = dd.getDate(); 
	    var str = y+'-'+(m<10?('0'+m):m)+'-'+(d<10?('0'+d):d)+' 00:00:00'; 
	    return str; 
	};
	
	var buttons = $.extend([], $.fn.datetimebox.defaults.buttons);
	buttons.splice(0, 1, 
			{text: '今天', handler: function(target){
				var date = today();
				$(target).datetimebox('setValue', date).datetimebox('setText', date).datetimebox("hidePanel");
			}
	});
	$(dom).datetimebox({buttons: buttons});
}  
{% endcodeblock %}   

### 3.easyui combogrid 
 定义多列搜索事件，例子如下：  
{% codeblock %} 
var startDevs = ...(先初始化数据);

$(dom).combogrid({
	panelWidth:300,
	idField: 'deviceCode',
	textField: 'deviceName',
	columns:[[
		{field:'deviceName',title:'设备名称',width:200},
		{field:'ip',title:'设备IP',width:120}
	]],
	onSelect: function(rowIndex, rowData){ },
	keyHandler: {
		up: function(e){},
		down: function(e){},
		left: function(e){},
		right: function(e){},
		enter: function(e){
			if(e.keyCode == 13) {
				var value = $(e.target).val();
				doSearch(value, startDevs, ["deviceName","ip"], $(this));
			}
		},
		query: function(q,e){}
	}
});
//加载静态数据
$(dom).combogrid("grid").datagrid('loadData', startDevs);  

//搜索名称和ip
function doSearch(q, data, searchList, ele){
	ele.combogrid('grid').datagrid('loadData', []);
	if(q == ""){
		ele.combogrid('grid').datagrid('loadData', data);
		return;
	}
	var rows = [];
	$.each(data, function(i, obj){
		for(var p in searchList){
			var v = obj[searchList[p]];
			if (!!v && v.toString().indexOf(q) >= 0){
				rows.push(obj);
				break;
			}
		}
	});
	ele.combogrid('grid').datagrid('loadData', rows);
	ele.combogrid('showPanel');
}
{% endcodeblock %} 