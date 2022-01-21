# riskTools

##  自述文件

1. 纯R语言风控模型包，整体风格受谢博士scorecard启发(https://github.com/ShichenXie/scorecard)
2. 支持的最优分箱方法：卡方最优分箱、cart决策树最优分箱、BestKs最优分箱
3. 其他分箱方法：聚类分箱、等频分箱、等距分箱、遍历分箱
4. 效果上目前测试不亚于市面上任何包，且支持输出单调趋势、自动调整为单调或正U倒U、自动调整单调
![image](https://user-images.githubusercontent.com/51108054/150460096-49021e73-adaf-4c09-b54a-1f79bb58c2f9.png)
![image](https://user-images.githubusercontent.com/51108054/150460160-d0705a9d-f8ed-4adb-a68c-55bcb995327f.png)
![image](https://user-images.githubusercontent.com/51108054/150460214-42643b5d-65a6-4b7f-8f22-b1a6740d237b.png)
5. 已知限制：卡方最优分箱在数据离度精确到小数位16位之后会报错，卡方分箱和决策树分箱在数据量达到一定量级的时候分箱效率较低 
6. 项目未来计划：
 
  (1). 将脚本改写成Julia语言，并为R和Python提供接口
  
  (2). 将不仅限于分箱、模型，将来会加入一些贷后指标的计算函数
  
  (3). 效率将作为主要优化点，Julia语言应该会比现在纯R语言实现效率上快很多
  
  (4). 之所以没有直接上传源代码是因为代码还在改进过程中，目前比较糟乱，欢迎大家提出使用问题，我将不遗余力的改进
  
  (5). 因项目初期，bug较多，不建议直接线上使用

##

## downloan

1. 下载riskTools.zip，解压后放到R包环境下，如果您找不到的话可以在R中输入.libPaths()即可

##
## example 1

### load data

library(data.table)

data("germancredit", package="scorecard")

data <- germancredit

###  Convert Y labels

data <- transformResponse(data,y = 'creditability')

dataList <- splitDt(data = data,ratio = c(0.7,0.3),seed = 520)

labelList <- lapply(dataList, function(x) x$y)

###  bins

binsChi <- binningsChimerge(
  data = data
  , y = 'y'
)

binsChiDt <- rbindlist(binsChi)

### woe

woeData <- woeTrans(data,y = 'y',bins = binsChi)

woeDataList <- lapply(dataList,function(x) woeTrans(x,y = 'y',bins = binsChi))

### glm

fit = glm(
  y ~ .
  , family = binomial()
  , data = woeDataList$train
)

summary(fit)

### stepwise by aic

fitStepAic <- step(
  fit
  , direction = 'both'
  , k = 2
  , trace = FALSE
)

summary(fitStepAic)

### pred

predList <- lapply(woeDataList, function(x) predict(fitStepAic, x, type='response'))

### performance

modelPerformance <- modelEffect(
  pred = predList
  , label = labelList
  , ord = "desc"
)

modelPerformance

###  score and scorecard

#### Only output the final model score

modelScoreRes <- lapply(predList, function(x) modelScore(x))

#### scorecard

sc <- scoreCard(
  bins = binsChi
  , logitModel = fitStepAic
)

scDt <- rbindlist(sc, fill = T)

riskForm(
 pred = modelScoreRes
 , label = labelList
 , ord = 'asc'
)

##  所有函数介绍
#### binningsBestKS：BestKs最优分箱(数据越离散效果越好，与卡方分箱不相上下)
#### binningsCartReg：cart回归树最优分箱
#### binningsChimerge：卡方最优分箱(大多数情况下表现最好，但是数据量很大的时候建议用一个变量测试一下时间)
#### binningsCluster：聚类最优分箱
#### binningsTraverse:遍历分箱，适用于特定场景，变量变量所有值
#### binningsEqual：等频分箱
#### binningsWidth：等距分箱
#### binsBestKS：BestKs最优分箱-只输出分割点
#### binsCartReg：cart回归树最优分箱-只输出分割点
#### binsCluster：聚类最优分箱-只输出分割点
#### binsEqual：等频分箱-只输出分割点
#### binsWidth：等距分箱-只输出分割点
#### describeData：快速的描述性统计
#### modelEffect：输出模型效果
#### modelScore：转换逻辑回归预测概率为得分
#### riskForm：分箱输出结果效果
#### scoreCard：输出入模变量及其得分
#### splitDt：切分数据集，可选训练测试或K折交叉
#### transformResponse：帮助转换Y标签
#### woePlot：分箱可视化-echarts4r
#### woePlotGG：分箱可视化-ggplot2
#### woeTrans:原始数据转换为woe








