## qoutation  
aar文件放到lib文件夹下，build.gradle 添加如下    
implementation(name: 'quotations-release-1.0', ext: 'aar')
  
   
   
## 1.DiscretionaryStockView (自选股View)  
### 使用方法
~~~ xml
布局引用
<com.qoutation.favorite.view.DiscretionaryStockView
        android:id="@+id/discretionaryview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>  
~~~
~~~ java
DiscretionaryStockView discretionaryStockView = ((DiscretionaryStockView) findViewById(R.id.discretionaryview));
        //设置自选股数据
        discretionaryStockView.setDataJson(FAVORITEJSON);
        //设置回调
        discretionaryStockView.setFavoriteCallBack(new FavoriteCallBack() {
            @Override
            public void clickStock(StockBean stockBean) {
                //点击自选股回调,CRM 跳转到相应界面
                startActivity(new Intent(FavoriteActivity.this, StockDetailActivity.class));
            }

            @Override
            public void removeStock(StockBean stockBean) {
               //调用删除自选接口，返回删除成功后调用
               discretionaryStockView.removeStock(stockBean.getAssetId());

            }
             @Override
            public void visibleStocks(List<String> assetIds) {
                //可见股票的股票ID列表回调，通过该列表做MQTT Topic的是否断或连

            }

            @Override
            public void marketChange(int market) {
                ///tab切换市场回调，
                ///market 0 : ALL, 1 : HK, 2 : US,3 : A
            }
        });  
~~~
##### DiscretionaryStockView 公开方法
~~~ java
   /**
     * 自选股
     * 设置json数据
     * @param jsonData 自选股数据json
     */
    public void setDataJson(String jsonData);
    /**
     * 调用CRM服务端删除自选成功后再调用此方法
     * @param assetId 股票代码(00700.HK)
     */
    public void removeStock(String assetId);
    /**
     * 设置接收到的mqtt数据
     * @param mqttData 接收到的MQTT数据
     */
    public void setMqttData(String mqttData);
~~~
##### FAVORITEJSON数据协议如下
~~~ java
[
    {
        "assetId": "00700.HK", // 股票代码
        "name": "腾讯控股", // 股票名称
        "price": "316.200", // 现价 
        "change": "3.000", // 涨跌额
        "changePct": "0.0096" // 涨跌幅
    },
    {
        "assetId": "COIN.US",
        "name": "Coinbase Global, Inc. Class A",
        "price": "316.200",
        "change": "3.000",
        "changePct": "0.0096"
    }
]
~~~
## 2.StockDetailView (个股详情View）
### 使用方法
~~~ xml
布局应用
<com.qoutation.stockdetail.view.StockDetailView
        android:id="@+id/stockdetailview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
~~~
~~~ java  
        mStockDetailView = findViewById(R.id.stockdetailview);
        //设置股票代码
        mStockDetailView.setAssetId("00700.HK");
        //主动获取行情数据以及分时数据，传递给UI组件
        mStockDetailView.setDataJson(STOCKDETAILJSON, KLINEDATA);
~~~
#### StockDetailView 公开方法
~~~ java
 /**
     * 初始化方法
     * @param stockDetailJson 股票详情json数据，必传
     * @param klineData 分时K线json，首次初始化必传
     */
    public void setDataJson(String stockDetailJson, String klineData);
    /**
     * 请求K线数据后调用方法
     *
     * @param klineData         json数据
     * @param chartTypeStr K线tab类型,没有则传空字符
     * @param adjustStr    复权类型，没有则传空字符
     * @param more         是否数据加载更多，默认传false
     */
    public void setKlineJson(String klineData, String chartTypeStr, String adjustStr, boolean more);
    /**
     * 设置接收到的mqtt数据
     *
     * @param mqttData 推送的mqtt数据
     */
    public void updateMqttData(String mqttData);
~~~
#### StockDetailView设置回调
~~~ java
  mStockDetailView.setActionListener(new AllActionListener() {
            @Override
            public void getQuot(List<String> assetIds) {
                //请求行情数据
//                {
//                    "userId": "10000001",
//                    "params": {
//                       "assetIds": [
//                       "00700.HK"
//                      ]
//                   }
//                }
                //回传行情数据
                mStockDetailView.setDataJson(STOCKDETAILJSONPLUS, "");
            }

            @Override
            public void getTimeSharing(String assetId, int periodType, String chartTypeStr) {
                //请求分时K线数据
//                {
//                    "userId": "10000001",
//                        "params": {
//                          "assetId": "00700.HK",
//                          "periodType": 2
//                    }
//                }
                //将获取到的分时数据回传
                mStockDetailView.setKlineJson(TIMESHARING, chartTypeStr, "", false);
            }

            @Override
            public void getHistoryQuot(String assetId, String chartTypeStr, int count, long timeStamp, String adjust, boolean loadMore) {
                //请求日K周K月K年K等K线数据
//                {
//                    "userId": "10000001",
//                    "params": {
//                        "assetId": "00700.HK",
//                        "type": "D",             ->chartTypeStr
//                        "count": 100,
//                        "ts": 1669944660000,     ->timeStamp
//                        "adjust": "F"
//                   }
//                }
                //将获取到的K线数据回传
                mStockDetailView.setKlineJson(KLINEMODELMORE, chartTypeStr, adjust, loadMore);
            }

            @Override
            public void getFiveTs(String assetId, int periodType, String charTypeStr) {
                //请求五日分时K线数据
//                {
//                    "userId": "10000001",
//                    "params": {
//                       "assetId": "00700.HK",
//                       "periodType": 2
//                    }2
//                }
                //将获取到的K线数据回传
                mStockDetailView.setKlineJson(FIVETIMESHARING, charTypeStr, "", false);
            }
        });
    }
~~~
#### STOCKDETAILJSON 数据协议如下  
~~~ java
盘口数据
接口返回result的这一层
[
    {
      "assetId": "00700.HK",
      "name": "腾讯控股",
      "price": "311.200",
      "change": "-5.000",
      "open": "314.600",
      "preClose": "316.200",
      "high": "314.600",
      "low": "310.000",
      "volume": "7812128",
      "turnover": "2441574803",
      "changePct": "-0.0158",
      "ttmPe": "14.05",
      "week52High": "416.600",
      "week52Low": "198.600",
      "hisHigh": "758.500",
      "hisLow": "-21.807",
      "avgPrice": "312.536",
      "turnRate": "0.0008",
      "pb": "3.68",
      "volRate": "1.19",
      "commitTee": "",
      "epsp": "22.15",
      "totalVal": "2983768038673",
      "ampLiTude": "0.0145",
      "flshr": "9587943569",
      "total": "9587943569",
      "status": 7,
      "ts": "1685502733000",
      "lotSize": "100",
      "bid1": "",
      "bidQty1": "",
      "ask1": "",
      "askQty1": "",
      "ttmDps": "2.400",
      "dpsRate": "0.0077",
      "isShortSell": false,
      "type": "1",
      "brokerQueue": "",
      "fmktVal": "2983768038673"
    }
  ]
~~~
#### KLINEDATA 数据协议如下
~~~ java
K线数据
接口返回result的这一层
{
    "lastDayTurnover": "13516781502.37",
    "data": [
      [
        1657641600000,
        "338.600",
        "341.200",
        "335.000",
        "335.400",
        "17134245",
        "5778623253.00",
        "337.800"
      ],
      [
        1657814400000,
        "330.000",
        "332.400",
        "323.200",
        "325.000",
        "26328504",
        "8620244653.00",
        "335.000"
      ]
    ],
    "lastDayVolume": "45550089"
  }
}
~~~
## mqttData数据协议如下
~~~ java
自选股和个股详情的推送数据统一是这个协议
MQTT推送数据
{
    "data": [
        [
         ...
        ]
    ],
    "funId": 2,
    "sendTime": "2023-05-31 10:44:12",
    "topicName": "QUOT/HK/00700.HK/2"
}
~~~
