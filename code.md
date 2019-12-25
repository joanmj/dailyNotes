# 总结
## EOS
## combobox下拉列表
1. 页面中增加样式，不加会导致数据无法正确显示
```html
<%@include file="/components/commons/common.jsp"%><!--标签引入的jsp-->
<%@include file="/business/jsp/js.jsp"%>
<link id="css_skin" rel="stylesheet" type="text/css"
	href="<%=contextPath%>/coframe/tools/skins/<%=skin%>/css/style.css" />
<link id="css_icon" rel="stylesheet" type="text/css"
	href="<%=contextPath%>/coframe/tools/icons/icon.css" />
```
2. 服务端直接从数据库查询，返回hashMap的数组就可以了；

##
