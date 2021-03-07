---
title: 'JS笔记'
date: '2021-03-07'
---

``` javascript
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>		
<body>
<script>
	//遍历数组的写法
    cars=["BMW","Volvo","Saab","Ford"];
    for (var i=0;i<cars.length;i++){
        document.write(cars[i] + "<br>");
    }
	//遍历对象的写法
	var apple={
		style:"round",
		color:"red",
		taste:"sweet",
	};
	for(var key in apple){
		document.write(key+":" +apple[key]+ "<br>");
		apple['style']
	}
</script>
</body>
</html>
```




