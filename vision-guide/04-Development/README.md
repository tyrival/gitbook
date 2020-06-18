# 第4章 二次开发组件

用户可以按照规范编写组件的配置文件，并将其注册到系统中，即可对图表、地图组件进行扩展，配置文件规范如下：

   **注意：配置文件中不允许包含任何注释**

```js
/**
 基本格式为:
 (function () {
    return {
        style: {},      // 对象，声明组件的基本样式，可为空。
        option: {},     // 对象，声明组件本身的配置，必填。
        data: [],       // 数组，组件载入的默认数据，必填。
        event: [],      // 数组，组件的事件注册，可为空。
        proxy: {},      // 对象，组件默认的数据源配置，可为空。
        transform: {},  // 对象，数据映射规则，必填。
        trigger: [],    // 数组，组件使用的触发器，可为空。
        handler: function(config)   // 函数，将 data 数据转化为合适的格式，并赋值给 option 中的相应属性，必填。
    }
 })()
 */
(function () {
    return {
        style: {
            // 宽度，默认值：400
            width: 400,
            // 高度，默认值：300
            height: 300,
            // 透明度，默认值：1
            opacity: 1,
            // 初始的可见性，'visible'/'hidden'，默认值：'visible'
            visibility: 'visible'
        },
        /**
         * option 是 Echarts 渲染的配置项，详细参考：https://echarts.apache.org/zh/option.html#title
         */
        option: {
            title: {
                show: false,
                text: '柱状图',
                textStyle: {
                    color: '#fff',
                    fontSize: '22',
                    fontWeight: 'normal'
                },
            },
            dataset: {
                dimensions: null,
                source: null,
            },
            backgroundColor: 'rgba(0,0,0,0)',
            grid: {
                top: 50,
                left: 50,
                right: 50,
                bottom: 50
            },
            legend: {
                show: true,
                data: ['数据0'],
                textStyle: {
                    fontSize: 12,
                    color: '#ffffff'
                },
                x: 'center',
                y: 'bottom',
            },
            tooltip: {
                show: true,
                trigger: 'axis',
                backgroundColor: 'rgba(0,0,0,0.6)',
                axisPointer: {
                    type: 'shadow',
                    label: {
                        show: true,
                        backgroundColor: '#0286ff'
                    }
                },
                textStyle: {
                    color: '#ffffff',
                    fontSize: 16,
                    fontWeight: 'normal',
                },
            },
            xAxis: [{
                type: 'category',
                gridIndex: 0,
                data: null,
                axisTick: {
                    alignWithLabel: true
                },
                axisLine: {
                    lineStyle: {
                        color: '#0c3b71'
                    }
                },
                axisLabel: {
                    show: true,
                    interval: 0,
                    rotate: 0,
                    color: 'rgb(170,170,170)',
                    fontSize: 16
                }
            }],
            yAxis: [{
                type: 'value',
                gridIndex: 0,
                splitLine: {
                    show: false
                },
                axisTick: {
                    show: false
                },
                axisLine: {
                    lineStyle: {
                        color: '#0c3b71'
                    }
                },
                axisLabel: {
                    show: true,
                    color: 'rgb(170,170,170)',
                    formatter: '{value}'
                },
                max: 100

            }, {
                type: 'value',
                splitLine: {
                    show: false
                },
                axisLine: {
                    show: false
                },
                axisTick: {
                    show: false
                },
                axisLabel: {
                    show: false
                }
            }],
            series: [{
                name: '数据0',
                type: 'bar',
                barWidth: '20%',
                barGap: '40%',
                xAxisIndex: 0,
                yAxisIndex: 0,
                label: {
                    normal: {
                        show: true,
                        position: 'top',
                        textStyle: {
                            color: '#2bff4e',
                            fontSize: 16
                        }
                    }
                },
                itemStyle: {
                    normal: {
                        barBorderRadius: 0,
                        isSingle: true,
                        color: '#ff0202'
                    }
                }
            }],
            markLines: [
                {
                    show: true,
                    markLine: {
                        data: [{yAxis: 50}],
                        lineStyle: {
                            width: '1',
                            opacity: '1'
                        }
                    }
                }
            ]

        },
        /**
         * data 是组件初始化时的默认数据
         */
        data: [
            {name: '周一', series: 80},
            {name: '周二', series: 87},
            {name: '周三', series: 51},
            {name: '周四', series: 81},
            {name: '周五', series: 23},
            {name: '周六', series: 45},
            {name: '周日', series: 33}
        ],
        /**
         * event 会被自动转换为事件面板上的配置项。其中每个元素表示一个事件，当事件触发时，组件会根据配置，将组件的内部
         * 属性值向外传递给指定的全局变量，当其他组件与该全局变量进行了绑定，则会触发这些组件的变化，例如：组件的显示与隐
         * 藏，组件从远端接口重新获取数据。
         * 其中各属性的含义如下：
         * - name: Echarts的组件事件，详细参考：https://echarts.apache.org/zh/api.html#events
         * - comment: 事件中文说明，会出现在事件配置面板上
         * - binders: 事件触发时，向外传递数据的规则，其中各属性含义如下：
         *      - name: 当未声明 prop 属性时生效，表示对外传递数据对象的属性名，例如下面的第 2 条绑定规则,
         *              {name: 'series', comment: '数据名', state: null},
         *              表示柱图中某个柱子，或者饼图中某个扇形被点击，假设点击的是 data 中的第 1 个数据对象，即
         *              {name: '周二', series: 87}，
         *              此时对外传递的值为 series 属性对应的 87
         *      - comment: 对外传值的中文说明，会显示在事件配置面板上
         *      - prop: 当 prop 声明时，则事件对外传值时，不再从 data 的数据对象中取值并对外传递，而是从 Echarts
         *              的事件函数的参数 params 中取值，例如下面的第 3 条绑定规则，
         *              {name: 'data', comment: '参数对象', prop: 'data', state: null}
         *              表示对外传递属性 params.data 的值，params 的具体说明参考上面 Echarts 鼠标事件在线文档
         *      - state: 当事件触发时，用来接收对外传值的全局变量属性名，这个全局变量名由用户进行配置，所以 state 的
         *              的值固定为 null
         */
        event: [{
            name: 'click',
            comment: '点击事件',
            binders: [
                {name: 'name', comment: '数据名', prop: 'name', state: null},
                {name: 'series', comment: '数据值', state: null},
                {name: 'data', comment: '参数对象', prop: 'data', state: null},
            ]
        }],
        /**
         * proxy 表示数据代理，可以不声明，或声明 type: 1，表示默认数据源为本地静态数据，即从上面的 data 属性获取数据。
         * 高级配置项可联系开发人员获取。
         */
        proxy: {
            type: 1,
        },
        /**
         * transform 表示数据映射规则，这个规则会在数据配置面板上自动转换为配置界面，通常与 data 中的数据属性保持一致。
         */
        transform: {
            mapper: {
                name: null,
                series: null
            }
        },
        /**
         * trigger 触发器，此处的配置会自动转化为事件面板上的触发器配置界面，其中各属性的含义如下：
         * - show: 表示此触发器是否在触发器配置界面显示，通常设置为 true 即可
         * - name: 显示在触发器配置界面的说明
         * - binder: 此处是由用户进行设置的触发器绑定规则，默认为 null 即可
         * - prop: 当触发器相关的内/外部属性发生变更时，触发组件的哪个属性发生变更
         * - defaultValue: 当用户未对该触发器进行设置时，prop 指向属性的默认值
         * - isScript: 表示 binder 中的配置是作为 javascript 脚本，还是作为字符串。
         *           为 true 时，表示 binder 的内容会作为脚本进行执行，执行结果赋予该组件的相应属性;
         *           为 false 时，表示 binder 的内容会作为字符串直接赋予该组件的相应属性，例如 Iframe 组件的 URL 属性
         *           就是利用此机制进行实现。
         * - mapper: 表示 binder 计算得到的结果分别对应什么值，该值才是最终被赋予组件的相应属性的值，此属性主要用于规范值的
         *          范围，避免用户在 binder 中声明过于复杂的结果，或因拼写错误导致脚本执行失败。其中各属性的含义如下：
         *          - key: binder 计算出的结果
         *          - value: binder 计算结果对应的值，最终赋予组件的相应属性
         *          - comment: 触发器面板上的说明文字
         */
        trigger: [{
            show: true,
            name: '可见性',
            binder: null,
            prop: "config.style.visibility",
            defaultValue: true,
            isScript: true,
            mapper: [
                {key: true, value: 'visible', comment: '返回 true 时显示'},
                {key: false, value: 'hidden', comment: '返回 false 时隐藏'},
            ]
        }],
        /**
         * handler 表示组件的所有配置都加载完成后（包括从远端获取数据），在组件渲染之前，所做的最后一次操作，
         * 通常，远端获取的数据属性与 transform 的默认属性差距较大，需要根据 transform.mapper 中的映射
         * 规则，对数据进行转换处理，并赋予 option 中的相应属性，然后才可使组件成功渲染。
         * 高级用法：由于 handler 是在渲染之前执行，所以还可以对组件的其他属性进行适合的修改，从而实现更高级的功
         * 能。例如：可以根据远端获取的数据，计算饼图每个扇形的渐变色范围。
         * - config: 此组件的完整配置项，基本结构与此JS脚本的返回值一致。
         */
        handler: function (config) {
            /*
             此示例是将数据进行映射转换，然后赋值给 echarts 的 option.dataset 配置项，而不是较早版本的
             echarts 所使用的 series.data 属性。
             */
            let dimensions = []
            // 获取用户配置的 transform.mapper 属性，作为 echarts 渲染时使用数据的维度
            for (let key in config.transform.mapper) {
                let dim = config.transform.mapper[key] || key
                if (!config.data[0].hasOwnProperty(dim)) {
                    dim = null
                }
                dimensions.push(dim)
            }
            config.option.dataset = {
                dimensions: dimensions,
                source: config.data
            }
            // 高级用法参考：使用 option.markLine 中声明的默认值，对 option.series 中每个系列的 markLine 
            // 属性进行赋值
            for (let i in config.option.series) {
                if (config.option.markLines[i].show) {
                    config.option.series[i].markLine = config.option.markLines[i].markLine;
                }
            }
        }
    }
})()
```