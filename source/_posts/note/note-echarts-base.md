title: ECharts图表入门级使用
date: 2014-12-15 13:17:11
categories: note
tags: [ECharts]
---

## ECharts图表入门级使用

> ECharts开源来自百度商业前端数据可视化团队，基于html5 Canvas，是一个纯Javascript图表库，提供直观，生动，可交互，可个性化定制的数据可视化图表。创新的拖拽重计算、数据视图、值域漫游等特性大大增强了用户体验，赋予了用户对数据进行挖掘、整合的能力。

### 准备  

首先去[ECharts](http://echarts.baidu.com/doc/example.html)官网下载文件(右上角的下载按钮),下载完后解压，进入到**doc\example\www**目录，  
会看到有有一个名叫js的文件夹，没错就是这里，在这里新建一个html文件，如test.html,内容如下:  

```javascript
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>ECharts示例</title>
<script type="text/javascript" src="js/echarts.js"></script>
</head>
<body>
<div id="main" style="height:600px;width:600px"></div>
<script type="text/javascript">

require.config({
	paths:{
		echarts: "js"//当前目录的js目录，里面有echarts.js和chart文件夹
	}
});

require(['echarts','echarts/chart/pie'],callBackDraw);//用到什么加什么，这里只用了饼图pie

function callBackDraw(ec){
	var myChart = ec.init(document.getElementById("main"));

	var option = { title : {
   		        text: '某站点用户访问来源',
   		        subtext: '纯属虚构',
   		        x:'center'
   		    },
   		    tooltip : {
   		        trigger: 'item',
   		        formatter: "{a} <br/>{b} : {c} ({d}%)"
   		    },
   		    legend: {
   		        orient : 'vertical',
   		        x : 'left',
   		        data:['直接访问','邮件营销','联盟广告','视频广告','搜索引擎']
   		    },
   		    toolbox: {
   		        show : true,
   		        feature : {
   		            mark : {show: true},
   		            dataView : {show: true, readOnly: false},
   		            magicType : {
   		                show: true, 
   		                type: ['pie', 'funnel'],
   		                option: {
   		                    funnel: {
   		                        x: '25%',
   		                        width: '50%',
   		                        funnelAlign: 'left',
   		                        max: 1548
   		                    }
   		                }
   		            },
   		            restore : {show: true},
   		            saveAsImage : {show: true}
   		        }
   		    },
   		    calculable : true,
   		    series : [
   		        {
   		            name:'访问来源',
   		            type:'pie',
   		            radius : '55%',
   		            center: ['50%', '60%'],
   		            data: [
   		                {value:335, name:'直接访问'},
   		                {value:310, name:'邮件营销'},
   		                {value:234, name:'联盟广告'},
   		                {value:135, name:'视频广告'},
   		                {value:1548, name:'搜索引擎'}
   		            ]  
   		        }
   		    ]
   		};

	myChart.setOption(option);
}
</script>
</body>
</html>
```

这里涉及到require，可能会比较麻烦，还有更简单的，**另一种用法**  

进入到**doc\example\www2**目录，新建一个test.html文件，内容如下:  

```javascript
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>ECharts示例</title>
<script type="text/javascript" src="js/echarts-all.js"></script>
</head>
<body>
<div id="main" style="height:600px;width:600px"></div>
<script type="text/javascript">
	var myChart = echarts.init(document.getElementById("main"));

	var option = { title : {
   		        text: '某站点用户访问来源',
   		        subtext: '纯属虚构',
   		        x:'center'
   		    },
   		    tooltip : {
   		        trigger: 'item',
   		        formatter: "{a} <br/>{b} : {c} ({d}%)"
   		    },
   		    legend: {
   		        orient : 'vertical',
   		        x : 'left',
   		        data:['直接访问','邮件营销','联盟广告','视频广告','搜索引擎']
   		    },
   		    toolbox: {
   		        show : true,
   		        feature : {
   		            mark : {show: true},
   		            dataView : {show: true, readOnly: false},
   		            magicType : {
   		                show: true, 
   		                type: ['pie', 'funnel'],
   		                option: {
   		                    funnel: {
   		                        x: '25%',
   		                        width: '50%',
   		                        funnelAlign: 'left',
   		                        max: 1548
   		                    }
   		                }
   		            },
   		            restore : {show: true},
   		            saveAsImage : {show: true}
   		        }
   		    },
   		    calculable : true,
   		    series : [
   		        {
   		            name:'访问来源',
   		            type:'pie',
   		            radius : '55%',
   		            center: ['50%', '60%'],
   		            data: [
   		                {value:335, name:'直接访问'},
   		                {value:310, name:'邮件营销'},
   		                {value:234, name:'联盟广告'},
   		                {value:135, name:'视频广告'},
   		                {value:1548, name:'搜索引擎'}
   		            ]  
   		        }
   		    ]
   		};

	myChart.setOption(option);

</script>
</body>
</html>
```

会看到和第一种方法一样的饼图，具体用哪种，可根据需求选择。  

简单的介绍就到这里，权且当做抛砖引玉。