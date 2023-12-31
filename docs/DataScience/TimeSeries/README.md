---
title: 时序教程六部曲~ Kaggle时间序列教程
---

Kaggle官方提供的[学习课程](https://kaggle.com/learn)，最近更新了Time Series时间序列的知识点。

Time Series课程包括如下章节：

- `Linear Regression With Time Series`
- `Trend`
- `Seasonality`
- `Time Series as Features`
- `Hybrid Models`
- `Forecasting With Machine Learning`

## 时序线性回归
https://www.kaggle.com/ryanholbrook/linear-regression-with-time-series

时间序列是一组随时间记录的观测结果，按照固定的频率记录。
[![pi3gTd1.png](https://z1.ax1x.com/2023/11/09/pi3gTd1.png)](https://imgse.com/i/pi3gTd1)
我们首先使用线性回归对输入的记录进行加权求和，得到我们的预测结果：
$$
target = weight * time + bias
$$

如果一个序列的值可以从它出现的时间开始预测，那么它就是时间相关的。为了增加模型的有效性，我们增加了Lag特征，将历史数据作为输入的特征。
$$
target = weight * lag + bias
$$

## 趋势
https://www.kaggle.com/ryanholbrook/trend
时间序列的趋势成分表示序列平均值的持续的、长期的变化。趋势是序列中移动最慢的部分，代表重要时间尺度最大的部分。
[![pi3gLRO.png](https://z1.ax1x.com/2023/11/09/pi3gLRO.png)](https://imgse.com/i/pi3gLRO)
为了了解时间序列的趋势，我们可以使用Moving Average Plots消除该系列中的任何短期波动，从而只保留长期变化。

对趋势进行建模，则课将时间加入建模：
$$
target = a * time + b
$$

如果趋势是二次的(抛物线)，通过可以模型变换来拟合趋势：
$$
target = a * time ** 2 + b * time + c
$$

## 季节性
https://www.kaggle.com/ryanholbrook/seasonality
当一个时间序列的均值有规律的周期性变化时，我们就说这个时间序列具有季节性。
[![pi32PFP.png](https://z1.ax1x.com/2023/11/09/pi32PFP.png)](https://imgse.com/i/pi32PFP)
Seasonal Plots显示的是时间序列的片段，这些片段相对于某个共同的时期。

傅里叶特征是一对正弦和余弦曲线，如果我们将一组正弦/余弦曲线添加到我们的训练数据中，算法将计算出适合的季节性成分的权重。

## 时序特征
https://www.kaggle.com/ryanholbrook/time-series-as-features

如果使用目标序列的过去值作为特征，历史趋势会变得更加清晰，如下图所示。序列依赖性的一种特别常见的表现方式，序列中某一时刻的值如何依赖于前一时刻的值有关。

[![pi32FW8.png](https://z1.ax1x.com/2023/11/09/pi32FW8.png)](https://imgse.com/i/pi32FW8)
为了研究时间序列中可能的序列依赖性(如周期)，我们需要创建该序列的Lag特征。Lag Plot显示了其相对于滞后的值。时间序列中的序列依赖性通常会通过观察滞后图而变得明显。

偏自相关告诉你一个滞后的相关性，绘制部分自相关可以帮助您选择使用哪些延迟特性。


## 超模型
https://www.kaggle.com/ryanholbrook/hybrid-models

线性回归擅长推断趋势，但无法进行特征交叉。XGBoost擅长特征交叉，但不擅长预测趋势。
[![pi32EQg.png](https://z1.ax1x.com/2023/11/09/pi32EQg.png)](https://imgse.com/i/pi32EQg)
许多时间序列可以用这三个成分加上一些本质上不可预测的、完全随机的误差的相加模型来描述：
$$
serious = trend + seasons + cycles + error
$$

## 机器学习预测
https://www.kaggle.com/ryanholbrook/forecasting-with-machine-learning

在设计预测模型之前，有两件事需要确定:

在做出预测(特写)时，可以获得哪些信息?
您需要预测值(目标)的时间段。

[![pi32ZLj.png](https://z1.ax1x.com/2023/11/09/pi32ZLj.png)](https://imgse.com/i/pi32ZLj)

预测的起点是你做预测的时间。您可以将预测起点看作是最后一次有训练数据可供预测的时间。

预测步长是你做预测的时间。我们经常用时间步数来描述预测:比如“1步”预测或“5步”预测。