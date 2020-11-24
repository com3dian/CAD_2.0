# CAD_OSE 算法文档

文件组成 `nabTestSimulator.py`,  `CAD_OSE.py`, `contextOperatpr.py`



## nabTestSimulator

### 前置参数

`TESTSET = 1` : 示性变量。

`numResultTypes = 1` ，`numReturnedAnomalyValues = 1`，`startAnomalyValueNumber = 0`:  

```python
if TESTSET == 1 :
        maxLeftSemiContextsLenght = 7 # maxLeftSemiContextsLenght is 
        maxActiveNeuronsNum = 15 # 
        numNormValueBits = 3 # 
        baseThreshold = 0.75 # 
```

### 文件接口



### 编码

```python
minValue = float("inf")
maxValue = -float("inf")
# ...
for ...:
    minValue = min(inputDataValue, minValue)# minimum
    maxValue = max(inputDataValue, maxValue)# maximum
# ...
learningPeriod = min( math.floor(0.15 * rowNumber), 0.15 * 5000)
```

`learningPeriod` :   `rowNumber` 是输入 time series 的长度。





### CAD模型

```python
cad = CAD_OSE(  
             minValue = minValue, 
             maxValue = maxValue, # from the last stage
             baseThreshold = baseThreshold, # 1.0 or 0.75
             restPeriod = learningPeriod / 5.0, 
    		# min( math.floor(0.15 * rowNumber), 0.15 * 5000)
             maxLeftSemiContextsLenght = maxLeftSemiContextsLenght, #
             maxActiveNeuronsNum = maxActiveNeuronsNum, #
             numNormValueBits = numNormValueBits
             )

inputData = {"timestamp" : inputDataDate, "value" : inputDataValue}
result = cad.getAnomalyScore(inputData)
```



## CAD_OSE

`__init__(self, minValue, maxValue, baseThreshold, restPeriod, maxLeftSemiContextsLenght, maxActiveNeuronsNum, numNormValueBits)`

```python
self.maxBinValue = 2 ** self.numNormValueBits - 1.0 # encoding
self.fullValueRange = self.maxValue - self.minValue # full range
```

```python
self.minValueStep = self.fullValueRange / self.maxBinValue # encoding
self.leftFactsGroup = tuple()
# 保存当前输⼊对应的1)类context的leftsemiContext

self.potentialNewContexts = []  
self.lastPredictionedFacts = []
self.resultValuesHistory = [ 1.0 ]
```



### CAD_OSE.getAnomalyScore(self, inputData)

```python
# inputData = {"timestamp" : inputDataDate, "value" : inputDataValue}
# numNormValueBits = 3
# self.minValueStep = self.fullValueRange / self.maxBinValue
# ...
normInputValue = int((inputData["value"] - self.minValue) / self.minValueStep) 
# int()
binInputNormValue = bin(normInputValue).lstrip("0b").rjust(self.numNormValueBits,"0")
# str(), '101'
```

**`OutSens`** 

```python
# print(normInputValue, outSens)
0 {0, 2, 4}
1 {1, 2, 4}
2 {0, 3, 4}
3 {1, 3, 4}
4 {0, 2, 5}
5 {1, 2, 5}
6 {0, 3, 5}
7 {1, 3, 5}
```



### CAD_OSE.step(self, inpFacts)

```python
# inpFacts = {1，2，4}
# currSensFacts = (1, 2, 4)
currSensFacts = tuple(sorted(set(inpFacts)))
```

**leftFactsGroup**

保存当前输⼊对应的1)类 `context` 的 `leftsemiContext`.

```python
# init
self.leftFactsGroup = tuple()
# ...
# init
self.leftFactsGroup = set()
# ...
# currSensFacts is like (1, 2, 4)
# currNeurFacts is like {2147483659, 2147483693, 2147483698, ..., 2147483675}
self.leftFactsGroup.update(currSensFacts, currNeurFacts)
# now leftFactsGroup is like {1, 2, 4, 2147483659, ..., 2147483675}
```

跳转 `contextOperator.getContextByFacts`。



## contextOperator

```python
# init potNewZeroLevelContext
potNewZeroLevelContext = tuple([self.leftFactsGroup, currSensFacts])
# potNewZeroLevelContext is like ({1, 2, 4, 2147483659, ..., 2147483675}, (1, 2, 4))
```

### contextOperator.getContextByFacts

```python
newContextFlag = self.contextOperator.getContextByFacts([potNewZeroLevelContext], zerolevel = 1)
```


```python
"""
        The function which determines by the complete facts list whether the context
        is already saved to the memory. If the context is not found the function
        immediately creates such. To optimize speed and volume of the occupied memory
        the contexts are divided into semi-contexts as several contexts can contain
        the same facts set in its left and right parts.
         
        @param newContextsList:         list of potentially new contexts
        
        @param zerolevel:               flag indicating the context type in
                                        transmitted list
                                          
        @return :   depending on the type of  potentially new context transmitted as
                    an input parameters the function returns either:
                    а) flag indicating that the transmitted zero-level context is
                    a new/existing one;
                    or:
                    b) number of the really new contexts that have been saved to the
                    context memory.
"""
# newContextsList is like [({1, 2, 4, 2147483659, ..., 2147483675}, (1, 2, 4))]
```

`context (zerolevel=1)`















