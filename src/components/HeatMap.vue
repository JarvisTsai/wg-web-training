<template>
    <div class="page-view">
        <card-header>訪問量</card-header>
        <div class="content">
            <div class="chart-container">
                <e-charts :options="defaultOptions" ref="chart" autoresize></e-charts>
            </div>
            <div class="instructions">
                <div class="visual-map-container">
                    <div class="visual-map"></div>
                    <div class="visual-map"></div>
                    <div class="visual-map"></div>
                    <div class="visual-map"></div>
                </div>
                <div class="text-map-container">
                    <div class="text-map">0</div>
                    <div class="text-map">10</div>
                    <div class="text-map">20</div>
                    <div class="text-map">30</div>
                    <div class="text-map">40</div>
                </div>
            </div>
        </div>
        <card-footer :show-date-selector="true" :show-view-more="true"></card-footer>
    </div>
</template>

<script>
    import ECharts from "vue-echarts";
    import "echarts";
    import CardFooter from "./CardFooter";
    import CardHeader from "./CardHeader";


    const dayOfWeek = ['週一','週二','週三','週四','週五','週六','週日']
    const hour =  Array.from(Array(24).keys()).map((value,index)=>`${index}:00`)
    const heatMapChartDefaultOptions = {
        tooltip: {
            padding: 10,
            backgroundColor: "#ffffff",
            textStyle: {
                color: "#737373",
            },
            extraCssText: "font-size: 12px; opacity: 0.8; min-width: 156px; display: flex; padding:12px; box-shadow: #E8E8E8 0px 0px 10px;"
        },
        grid: {
            left: "0",
            right: "0",
            top: "15",
            bottom: "5",
            containLabel: true,
        },
        legend: {show: false},
        xAxis: {
            position:'top',
            type: "category",
            axisLine: {
                show: false,
            }, 
            axisTick: {
                show: false,
                interval:1
            },
            axisLabel: {
                color: "#737373",
                margin: 20,
            },
            splitLine: {
                lineStyle: {
                    color: "#d8d8d8",
                    type: "dashed",
                    width:"5"
                },
            },
            data : dayOfWeek
        },
        yAxis: {
            position:'right',
            axisLine: {
                show: false,
            },
            axisLabel: {
                color: "#737373",
                margin: 10,
                interval:1
            },
            axisTick: {
                show: false
            },
            splitLine: {
                lineStyle: {
                    color: "#d8d8d8",
                    type: "dashed",
                    width:"5"
                },
            },
            data : hour
        }, 
        visualMap: {
            min: 0,
            max: 500,
            color:['rgb(0, 127, 128)','rgb(0, 187, 189)','rgb(153, 228, 229)','rgb(217, 245, 245)'],
            calculable: false,
            show:false
        },
        series: [
            {
                name: 'Punch Card',
                type: "heatmap",   
                itemStyle: {
                    emphasis: {
                        shadowBlur: 10,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    },
                },
                label: {
                    normal: {
                        show: false
                    }
                },
            },
        ]
    };

    export default {
        name: "HeatMap",
        components: {ECharts, CardHeader, CardFooter},

        data: function () {
            return {
                defaultOptions: heatMapChartDefaultOptions,
                chartsData: undefined,
            }
        },

        created() {
            window.addEventListener("orientationchange", this.onOrientationchange)
        },

        mounted() {
            this.fetchData();
        },

        beforeDestroy() {
            window.removeEventListener("orientationchange", this.onOrientationchange)
        },

        methods: {
            onOrientationchange: function () {
                this.$refs.chart.resize();
            },

            fetchData: function () {
                this.$refs.chart.showLoading();

                this.getHeapMapData()
            },

            fetchAnalyticsData:function(params){
                const option = {
                    body: JSON.stringify(params),
                    headers: {
                        'content-type': 'application/json'
                    },
                    method: 'POST',
                    mode: 'cors',
                };
                return fetch("http://dashboard.qa.tronclass.com.cn/api/al/heatmap", option)
                    .then(response => response.json())
            },

            getHeapMapData(){
                let params = {"date_ranges":[{"start":"2018-06-30","end":"2018-07-31"}],"dimensions":[{"name":"day-of-week"},{"name":"hour"}],"metrics":[{"name":"tc:page_view"}],"filters":{}}
                this.fetchAnalyticsData(params)
                    .then(this.onFetchDataSuccess)
                    .catch(this.onFetchDataFailure)
            },

            onFetchDataFailure: function (error) {
                this.$refs.chart.hideLoading();
                console.log(error)
            },
            onFetchDataSuccess: function (chartsData) {
                // TODO: look here: data structure
                console.log('--------- look here: data structure ---------');
                console.log(chartsData);
                console.log('--------- look here: data structure ---------');

                this.chartsData = chartsData;
                let options= {
                    dataset: {
                        source: chartsData.result.data[0].rows
                    },
                    visualMap:{
                        max: Math.max.apply(null, chartsData.result.data[0].rows.map(row=>row[2]))
                    },
                }
               
                this.$refs.chart.mergeOptions(options);

                this.$refs.chart.hideLoading();
            },
        },
    }
</script>

<style scoped lang="scss">
    $colors:rgb(217, 245, 245),rgb(153, 228, 229) ,rgb(0, 187, 189),rgb(0, 127, 128);
    @mixin mark($len, $colors) {
        @for $i from 1 through $len {

            &:nth-child(#{$i}) {
                background-color: nth($colors, $i);
            }
        }
    }

    .page-view {
        display: flex;
        flex-direction: column;
        width: 100%;
        height: 100%;

        .content {
            position: relative;
            flex: 1;
            display: flex;
            flex-direction: column;
            padding-bottom: 2rem;

            .chart-container {
                position: relative;
                flex: 1;
                padding: 0 4rem 0 2rem;

                .echarts {
                    width: 100%;
                    height: 100%;
                }

                
               
            }

            .instructions{
                margin-top: 1rem;
                margin-bottom: 1rem;
                padding: 0 7rem 0 2rem;
            }

            .visual-map-container{
                display: flex;
                margin-bottom: 0.5rem;

                .visual-map{
                    flex: 1 1 0%;
                    height: 1rem;
                    margin-right: 0.4rem;

                    background-color: rgb(0, 127, 128);
                    @include mark(4, $colors);
                }
            }

            .text-map-container{
                display: flex;
                justify-content: space-between;

                .text-map{
                    height: 1rem;
                }
            }
        }

    }
</style>

