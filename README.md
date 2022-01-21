# riskTools

library(data.table)

data("germancredit", package="scorecard")

data <- germancredit

##  Convert Y labels

data <- transformResponse(data,y = 'creditability')

dataList <- splitDt(data = data,ratio = c(0.7,0.3),seed = 520)

labelList <- lapply(dataList, function(x) x$y)

##  bins

binsChi <- binningsChimerge(
  data = data
  , y = 'y'
)

binsChiDt <- rbindlist(binsChi)

## woe

woeData <- woeTrans(data,y = 'y',bins = binsChi)

woeDataList <- lapply(dataList,function(x) woeTrans(x,y = 'y',bins = binsChi))

## glm

fit = glm(
  y ~ .
  , family = binomial()
  , data = woeDataList$train
)

summary(fit)

## stepwise by aic

fitStepAic <- step(
  fit
  , direction = 'both'
  , k = 2
  , trace = FALSE
)

summary(fitStepAic)

## pred

predList <- lapply(woeDataList, function(x) predict(fitStepAic, x, type='response'))

## performance

modelPerformance <- modelEffect(
  pred = predList
  , label = labelList
  , ord = "desc"
)

modelPerformance

##  score and scorecard

### Only output the final model score

modelScoreRes <- lapply(predList, function(x) modelScore(x))

### scorecard

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

