```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ECharts 碳转移图</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
    <style>
        #main {
            width: 1000px; /* 设置图表的宽度 */
            height: 1000px; /* 设置图表的高度 */
            position: relative; /* 使子元素可以使用绝对定位 */
        }

        #heatmap {
            position: absolute;
            width: 80%; 
            height: 80%; 
            /* background-color: lightblue; 背景颜色仅用于演示 */
        }

        #bar-b {
            position: absolute;
            /* top: 0; */
            left: 80%; 
            width: 10%; 
            height: 80%; 
            /* background-color: lightcoral; 背景颜色仅用于演示 */
        }

        #bar-a {
            position: absolute;
            top: 80%; 
            width: 100%; 
            height: 10%; 
            /* background-color: lightgreen; 背景颜色仅用于演示 */
        }
    </style>
</head>

<body>
  <!-- 创建一个用于放置图表的容器 -->
   <div id="main">
        <div id="heatmap"></div>
        <div id="bar-a"></div>
        <div id="bar-b"></div>
   </div>

  
  <script>
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

    // 调用函数并在数据加载完成后初始化ECharts
    loadJsonData().then(datas => {
        const yData =  ["BJ", "TJ", "HE", "SX", "IM", "LN", "JL", "HL", "SH", "JS", "ZJ", "AH", "FJ", "JX", "SD", "HA", "HB", "HN", "GD", "GX", "HI", "CQ", "SC", "GZ", "YN", "SN", "GS", "QH", "NX", "XJ"];
        const xData =  ["BJ", "TJ", "HE", "SX", "IM", "LN", "JL", "HL", "SH", "JS", "ZJ", "AH", "FJ", "JX", "SD", "HA", "HB", "HN", "GD", "GX", "HI", "CQ", "SC", "GZ", "YN", "SN", "GS", "QH", "NX", "XJ"];
        const provinces = ["北京", "天津", "河北", "山西", "内蒙古", "辽宁", "吉林", "黑龙江", "上海","江苏", "浙江", "安徽", "福建", "江西", "山东", "河南", "湖北", "湖南", "广东","广西", "海南", "重庆", "四川", "贵州", "云南", "陕西", "甘肃", "青海", "宁夏", "新疆"]

        const matrix = datas;
        const rows = matrix.length;
        const cols = matrix[0].length;

        // 计算每一列的和(碳流入)
        const colSum = new Array(cols).fill(0);
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                colSum[j] += matrix[i][j];
            }
        }
        console.log("colSum = " + colSum)

        // 计算每一行的和(碳流出)
        const rowSum = new Array(rows).fill(0);
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                rowSum[i] += matrix[i][j];
            }
        }   
        console.log("rowSum = " + rowSum)

        // 将二维数组转换为ECharts热力图所需的格式 
        // const data = [];
        // for (let i = 0; i < matrix.length; i++) {
        //     for (let j = 0; j < matrix[i].length; j++) {
        //         data.push([i, j, matrix[i][j]]); // xAxis yAxis 值
        //     }
        // }

        /////////////////////////////////////
        // 1.绘制热力图 heatmap
        const heatmap = echarts.init(document.getElementById('heatmap'));
        heatmap.setOption({
            title: {
                text: '热力图',
            },
            grid: {
                left: '10%',
                right: '',
                top: "10%",
                bottom: '',
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
                position: 'top', // 翻转X轴到最上面
                axisLabel: {
                    rotate: 270  // 将xAxis英文字体水平旋转270度
                }
            },
            yAxis: {
                type: 'category',
                data: yData,
                inverse: true, // 翻转Y轴（翻转前数据在第1象限，翻转后在第4象限） 
            },
            visualMap: { // 就是那个可以拖拽范围的
                min: 0,
                // max: Math.max(...matrix.flat()), 
                max: 53,
                calculable: true, // 是否显示拖拽用的手柄
                orient: 'vertical', // 设置为垂直
                left: "0",          // 距离容器的左侧
                bottom: '-0.5%',    // 距离容器的底部
                inRange: {
                    color: ['#fecc5c','#fd8d3c','#f03b20','#bd0026']
                },
                // show: false, // 不显示
            },
            series: [{
                type: 'heatmap',
                // 先对每一行进行 map 操作，将每个元素映射为 [j, i, value]，然后将结果展平一层。
                data: matrix.flatMap((row, i) => row.map((value, j) => [j, i, value])), 
                label: {
                    show: false // 不显示标签
                },
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }]
        });


        /////////////////////////////////////
        // 2.绘制底部柱状图 bar_bottom
        const bar_bottom = echarts.init(document.getElementById('bar-a'));
        bar_bottom.setOption({
            title: {
                text: '碳流入柱状图',
                show: false,
            },
            grid: {
                left: '8%',
                right: '20%',
                top: "",
                bottom: '',
            },
            tooltip: {
                trigger: 'axis', // 可选 'item' 或 'axis'
                axisPointer: {
                    type: 'shadow' // 可选 'line' 或 'shadow'
                },
                formatter: function (params) {
                    var result = '';
                    params.forEach(function (item) {
                        result += '碳流入<br/>' + item.name + ": " + item.value.toFixed(3) + '<br/>';
                    });
                    return result;
                }
            },
            xAxis: {
                type: 'category',
                data: xData,
            },
            yAxis: {
                type: 'value',
                inverse: true,
                show: false,
            },
            series: [{
                name: "碳流入",
                data: colSum,
                type: 'bar',
                inverse: true,
                barWidth: '80%', // 设置柱子的宽度
                barGap: '0%',
                itemStyle: {
                    color: '#a0d9d0' // 设置柱状图的颜色为粉红色
                }
            }]
        })

        /////////////////////////////////////
        // 3.绘制右侧柱状图 bar_right
        const bar_right = echarts.init(document.getElementById('bar-b'));
        bar_right.setOption({
            title: {
                text: '碳流出柱状图',
                show: false,
            },
            grid: {
                left: '0', 
                top: '90px', 
                width: '', 
                height: '700px',
            },
            tooltip: {
                trigger: 'axis', 
                axisPointer: {
                    type: 'shadow' 
                },
                formatter: function (params) {
                    var result = '';
                    params.forEach(function (item) {
                        result += '碳流出<br/>' + item.name + ": " + item.value.toFixed(3) + '<br/>';
                    });
                    return result;
                }
            },
            xAxis: {
                show: false,
            },
            yAxis: {
                type: 'category', 
                data: yData, 
                boundaryGap: false, 
                axisLine: {onZero: false},
                show: false,
                inverse: true,
            },
            series: [{
                type: 'bar', 
                data: rowSum,
                barWidth: '80%', // 设置柱子的宽度
                barGap: '0%',
                tooltip:{
                    trigger: 'axis', 
                    axisPointer: {
                        type: 'shadow' 
                    },
                    formatter: function (params) {
                        var result = '';
                        params.forEach(function (item) {
                            result += '碳流出<br/>' + item.name + ": " + item.value.toFixed(3) + '<br/>';
                        });
                        return result;
                    }
                },
                itemStyle: {
                    color: '#fc939e' // 设置柱状图的颜色为粉红色
                }
            }]
        })

    });

  </script>
</body>
</html>

```

