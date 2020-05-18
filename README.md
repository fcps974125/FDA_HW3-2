# FDA_HW3-2 Report

### Dataset
- [Electrical Grid Stability Simulated](https://archive.ics.uci.edu/ml/datasets/Electrical+Grid+Stability+Simulated+Data+)

### Problen Statement
- 給定變數的12個變數(X), 本次的目標為 **預測此系統是否stable(y)** 並且 **挑選出影響穩定性的重要變數** 。
    - X: tau1, tau2, tau3, tau4, p1, p2, p3, p4, g1, g2, g3, and g4
    - y: stab and stabf
- 為了可以評價每個變數的重要性，我們將這個問題定義為 **regression problem** ，反應變數y為stab，並致力於最小化 residual sum of square。
- 評價模型的指標為R^2(stab)與accuracy(stable if predictive value is negative)

### Data Analysis
1. metadata
    - 變數空間維度: 12
    - 資料筆數: 10000
    - 遺失值: 無
    
    
2. Relation between variable and stab
    - tau: 可以稍微看見弧形的趨勢，可能與stab有**非線性關係**
    - p: 看起來是較無關的變數
    - g: 與stab呈現**正相關**


3. Data correlation
    - 所有變數與stab的相關係數都在0.4以下
    - 變數之間的共線性不高
    - tau與g有機會是重要變數
        
**小結** |　不需要做遺失值處理；tau與g的8個變數可能為重要變數；tau可能與stab有非線性關係；不需要著墨於共線性的問題。

### Modeling
這部分選擇的模型為LASSO with BIC，主要的考量為LASSO利用l1 regularization，可以幫助變數我們做篩選，與我們的目標一致(挑選出影響穩定性的重要變數)。資料經過normalize的處理。隨機選擇5000筆資料作為training data，另外5000筆作為testing data，並且同時考慮training R^2、testing R^2、training accuracy、testing accuracy以及常態假設，作為選擇模型的依據。


1. Fitting raw data
    - 僅利用原始資料，可以發現p的4個變數的係數皆被限縮到0，代表這些變數不顯著(與Data Analysis得到相同的結論)。
    - 從normal probability plot可以發現，殘差與stab有非線性的關係(與Data Analysis得到相同的結論)。
    - 首先嘗試power transform來解決非線性的問題，如2.。
    

2. Fitting data with power transform
    - 利用yeo-johnson transformation後的資料。
    - 與Fitting raw data相同，p的4個變數不顯著。
    - R^2與accuracy皆比Fitting raw data佳。
    - 從normal probability plot可以發現，非線性關係依然存在，猜測非線性關係並非來自個別變數本身，而是變數之間的交互作用。
    - 將資料加入兩兩交互作用項(不包含p以及其交互作用項)，如3.。
    
    
3. Fitting data with interaction term only
    - 變數增加到37個，其中有14個變數為顯著變數。
    - R^2與accuracy有明顯的提升。
    - 從normal probability plot可以發現，常態假設成立，並且沒有非線性問題。
    
    
4. Fitting data with second order interaction
    - 嘗試以增加2次項來提升準確度，R^2與accuracy有明顯的提升，代表非線性關係也可能來自於變數本身。
    - 從normal probability plot可以發現，殘差有heavy-tail的問題，常態假設可能不成立。
    
    
5. Fitting data with third/fourth order interaction
    - 加入更高次項，R^2與accuracy有提升
    - 從normal probability plot可以發現，殘差有嚴重heavy-tail的問題，常態假設不成立。

|model|train R^2|test R^2|train acc|test acc|#variable|
|-|-|-|-|-|-|
|raw|0.6437|0.6461|0.8164|0.8146|8/12|
|power|0.6657|0.6688|0.8208|0.8212|8/12|
|interaction only|0.7320|0.7283|0.859|0.8578|14/37|
|second order|0.8673|0.8668|0.9318|0.9304|21/45|
|third order|0.9255|0.9254|0.9524|0.9482|48/165|
|fourth order|0.9281|0.9282|0.9514|0.9512|60/495|

**小結** | 嘗試6種結構皆為LASSO的迴歸模型，testing R^2和accuracy分別可以達到0.9282與0.9512。加入較高次項的變數並且考慮交互作用，有利於提高預測能力，並且驗證了Data Analysis中提到，變數與stab有非線性的問題。我們觀察到資料擴展到三次項以後，有嚴重的heavy-tail問題，並且在R^2和accuracy的表現上，沒有顯著優於second order的模型，所以我認為second order為最佳的模型。

### Coclusion
這個專題我們試圖回答兩個問題：如何預測系統是否stable以及哪些變數為重要變數。
1. 如何預測系統是否stable?

    從Data Analysis以及Modeling的結果，可以發現變數與stab確實存在非線性的關係，一部份來自於變數本身與stab有二次以上的關係，一部分來自於變數之間的交互作用對於stab有影響。經過Modeling的分析，我認為考慮**變數的二次項**以及**變數之間的二次交互作用項**，就足夠預測系統是否stable，在我們的實驗中，testing R^2和accuracy分別達到0.8668以及0.9304。


2. 哪些變數為重要變數?

    利用LASSO可以做變數選擇的優勢，我們利用BIC作為模型選擇的指標，並且得到顯著影響模型的變數為tau1、tau2、tau3、tau4、tau1^2、tau1tau2、tau1tau3、tau1tau4、tau1g1、tau2^2、tau2tau3、tau2tau4、tau2g2、tau3^2、tau3tau4、tau3g3、tau4^2、tau4g4、g1^2、g2^2、g3^2，共21個。可以發現tau的四個變數在一次項、二次項以及交互作用，都有顯著的影響。g的四個變數則是在二次項以及交互作用有顯著影響。
