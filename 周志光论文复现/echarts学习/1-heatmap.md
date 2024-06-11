### matrix的读取方式

#### 代码一：直接保存json数据

> 优点：可读性强
>
> 缺点：不好修改维护

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ECharts 热力图和柱状图</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
    <style>
        #heatmap {
            width: 800px; /* 设置图表的宽度 */
            height: 800px; /* 设置图表的高度 */
            /* margin: auto; 水平居中 */
            position: absolute; /* 绝对定位 */
            top: 0; left: 0; bottom: 0; right: 0; /* 距离四边都为0 */
        }
    </style>
</head>
<body>
    <div id="heatmap"></div>
    <div id="bar-a" style="width: 600px; height: 200px;"></div>
    <div id="bar-b" style="width: 600px; height: 200px;"></div>
    <script>
        const yData =  ["BJ", "TJ", "HE", "SX", "IM", "LN", "JL", "HL", "SH", "JS", "ZJ", "AH", "FJ", "JX", "SD", "HA", "HB", "HN", "GD", "GX", "HI", "CQ", "SC", "GZ", "YN", "SN", "GS", "QH", "NX", "XJ"];
        const xData =  ["BJ", "TJ", "HE", "SX", "IM", "LN", "JL", "HL", "SH", "JS", "ZJ", "AH", "FJ", "JX", "SD", "HA", "HB", "HN", "GD", "GX", "HI", "CQ", "SC", "GZ", "YN", "SN", "GS", "QH", "NX", "XJ"];

        const provinces = ["北京", "天津", "河北", "山西", "内蒙古", "辽宁", "吉林", "黑龙江", "上海","江苏", "浙江", "安徽", "福建", "江西", "山东", "河南", "湖北", "湖南", "广东","广西", "海南", "重庆", "四川", "贵州", "云南", "陕西", "甘肃", "青海", "宁夏", "新疆"]
        
        // matrix 为30*30的二维数组，在这里直接加入json数据
        const matrix = [[]];

        const rows = matrix.length;
        const cols = matrix[0].length;

        // 计算每一列的和
        const colSum = new Array(cols).fill(0);
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                colSum[j] += matrix[i][j];
            }
        }
        console.log(colSum)

        // 计算每一行的和
        const rowSum = new Array(rows).fill(0);
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                rowSum[i] += matrix[i][j];
            }
        }   
        console.log(rowSum)

        // 将二维数组转换为ECharts热力图所需的格式 
        // const data = [];
        // for (let i = 0; i < matrix.length; i++) {
        //     for (let j = 0; j < matrix[i].length; j++) {
        //         data.push([i, j, matrix[i][j]]); // xAxis yAxis 值
        //     }
        // }
        const data = matrix.flatMap((row, i) => row.map((value, j) => [j, i, value]));

        console.log(data)

        // 绘制热力图
        const heatmap = echarts.init(document.getElementById('heatmap'));
        heatmap.setOption({
            title: {
                text: '热力图'
            },
            grid: {
                left: '10%',
                right: '5%',
                top: "10%",
            },
            tooltip: {
                position: 'top',
                formatter: function (params) {
                    return '碳转移' + '<br/>' + 
                        // '行：' + params.data[1] + '<br/>' +
                        // '列：' + params.data[0] + '<br/>' +
                        provinces[params.data[1]] + ' -> ' + provinces[params.data[0]] + ': ' + 
                        params.data[2].toFixed(3); // 值保留3位小数
                }
            },
            xAxis: {
                type: 'category',
                data: xData,
                position: 'top', // 翻转X轴
            },
            yAxis: {
                type: 'category',
                data: yData,
                inverse: true, // 翻转Y轴（翻转前数据在第1象限，翻转后在第4象限） 
            },
            visualMap: { // 就是那个可以拖拽范围的
                min: 0,
                max: Math.max(...matrix.flat()), // 找出多维数组 matirx 中的最大值
                calculable: true, // 是否显示拖拽用的手柄（手柄能拖拽调整选中范围）
                orient: 'vertical', // 设置为垂直
                left: 5,          // 距离容器左侧5像素
                bottom: '39%',    // 容器的底部
                inRange: {
                    color: ['#fecc5c','#fd8d3c','#f03b20','#bd0026']
                },
            },
            series: [{
                type: 'heatmap',
                data: matrix.flatMap((row, i) => row.map((value, j) => [j, i, value])),
                label: {
                    show: false
                },
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }]
        });

        // // 绘制柱状图A
        // const barA = echarts.init(document.getElementById('bar-a'));
        // barA.setOption({
        //     title: {
        //         text: '每列数据和'
        //     },
        //     tooltip: {},
        //     xAxis: {
        //         type: 'category',
        //         data: Array.from({ length: cols }, (_, i) => i + 1)
        //     },
        //     yAxis: {
        //         type: 'value'
        //     },
        //     series: [{
        //         type: 'bar',
        //         data: colSums
        //     }]
        // });

        // // 绘制柱状图B
        // const barB = echarts.init(document.getElementById('bar-b'));
        // barB.setOption({
        //     title: {
        //         text: '每行数据和'
        //     },
        //     tooltip: {},
        //     xAxis: {
        //         type: 'category',
        //         data: Array.from({ length: rows }, (_, i) => i + 1)
        //     },
        //     yAxis: {
        //         type: 'value'
        //     },
        //     series: [{
        //         type: 'bar',
        //         data: rowSums,
        //         label: {
        //             show: true,
        //             position: 'right'
        //         }
        //     }],
        //     grid: {
        //         left: '3%',
        //         right: '10%',
        //         bottom: '3%',
        //         containLabel: true
        //     }
        // });

    </script>
</body>
</html>

```



#### 代码二：定义一个函数来读取json文件

> 优点：代码很短，方便维护
>
> 缺点：不好看懂

```javascript
    // 定义一个函数来读取JSON文件，并返回一个Promise
    function loadJsonData() {
      return fetch('../data/30*30_matrix_2017.json')
        .then(response => {
          if (!response.ok) {
            throw new Error('网络响应错误');
          }
          return response.json();
        })
        .then(jsonData => {
          console.log("JSON数据:", jsonData);
          return jsonData;
        })
        .catch(error => {
          console.error("读取JSON文件时出错:", error);
        });
    }
```

