---
title: easyui遮罩
author: 不好听
img: /2018/11/23/easyui-zhe-zhao/images/easyui.PNG
top: false
toc: false
categories: easyui
tags:
  - easyui
  - 遮罩
date: 2018-11-23 20:27:22
---

### easyui的页面加载问题
在用easyui做页面的过程中，总是会出现页面加载慢，在显示一堆乱七八糟的没渲染的页面后，才显示正常。是否会有一种日了狗的感觉？！  
![日了狗](images/2016322722279139.jpg "日了狗")  
此时，我们可以在页面上加上遮罩，页面加载完成之后，再显示。 
![loading](images/108f3e73c794eba2555d2834272f0836.gif "loading")  
只需要在页面中引入一个js脚本就可以了，不需其他任何操作，脚本内容如下： 
{% codeblock %} 
//获取浏览器页面可见高度和宽度 
var _PageHeight = document.documentElement.clientHeight, _PageWidth = document.documentElement.clientWidth; 
//计算loading框距离顶部和左部的距离（loading框的宽度为215px，高度为61px）
var _LoadingTop = _PageHeight > 61 ? (_PageHeight - 61) / 2 : 0, _LoadingLeft = _PageWidth > 215 ? (_PageWidth - 215) / 2 : 0; 
//加载gif地址
var Loadimagerul="../commons/img/base_loading.gif";
//在页面未加载完毕之前显示的loading Html自定义内容
var _LoadingHtml = '<div id="loadingDiv"  style="position:absolute;left:0;width:100%;height:' + 	
	_PageHeight +  'px;top:0;background:#f3f8ff;opacity:1;filter:alpha	(opacity=80);z-index:10000;">'+
	'<div style="position: absolute; 	cursor1: wait; left: ' + _LoadingLeft + 'px; top:' + 	
	_LoadingTop + 'px; width:100px;; height: 57px; line-height: 	57px; padding-left: 50px; '+
	'padding-right: 5px; background: #fff 	url('+Loadimagerul+') no-repeat scroll 5px 12px; border: 2px 	'+
	'solid #95B8E7; color: #696969; font-family:\'Microsoft YaHei	\';">加载中...</div></div>';
//呈现loading效果 
document.write(_LoadingHtml);
//监听加载状态改变
document.onreadystatechange = completeLoading; 
//加载状态为complete时移除loading效果
function completeLoading() {
	if (document.readyState == "complete") { 
			var loadingMask = document.getElementById('loadingDiv'); 
			loadingMask.parentNode.removeChild(loadingMask);
	 } 
}
{% endcodeblock %}  
有没有一种很容易的感觉？  

不过，需要注意的是：
当页面加载的内容过多时，页面停留在loading的时间会很长，影响体验；此时应该考虑使用懒加载。
