# 作業3 - 實作「collaborative filtering」的推薦系統

###### tags: `alphacamp`, `recommendation system`

## 預期產出

- 一份文字敘述：Readme，包含題幹敘述中的兩個問題。
	- 清楚記載這個專案的目的和結果，最後的推薦分數是多少，是否有成功
 	- 簡明清楚的使用說明：用了哪些工具和方法？為什麼？
- 兩支連結：包含 colab code、github code。
 	- [Colab code(合併)](https://colab.research.google.com/drive/1NGTXuwxneIBzxnkPHAOm4udT2RnnpGh3?usp=sharing)
		- [User-based](https://colab.research.google.com/drive/1c8y0G-Nd_517QzFbRklvcp5DHWsHuDk8?usp=sharing)
		- [Item-based](https://colab.research.google.com/drive/1wjjCJEZ3f7H87egQ9_Ob2OwFmM23q0Pu?usp=sharing)
		- [Surprise套件](https://colab.research.google.com/drive/1tw_Mcrvj2xFnvuYYjf2gWezY1cM7RYBd?usp=sharing)
 	- [Github code](https://github.com/chen2369/data-course-sample)

## 專案目的

- 將美妝類電商所擁有的資料集（包含評論資訊、商品後設資料）進行分析，以過去資料來推薦使用者會購買的商品。
- 評估方式：以過去資料作為依據，推薦k個商品給最後一個月購買商品的使用者，將使用者購買的商品是否在推薦名單之中作為分數計算。

## 資料集

- 評論資訊
	- 將2000-01-10～2018-09-01的資料作為Training data（370752筆）。
	- 將2018-09-01～2018-09-30的資料作為Testing data（590筆）。
	- 欄位
		- reviewerID - ID of the reviewer, e.g. A2SUAM1J3GNN3B ← 用戶 ID
		- asin - ID of the product, e.g. 0000013714 ← 商品 ID
		- overall - rating of the product ← 用戶對商品的評分
		- unixReviewTime - time of the review (unix time) ← 留下評論的時間戳記
- 商品後設資料
	- 共包含32892筆商品資料。
	- 欄位
		- asin - ID of the product, e.g. 0000031852 ← 商品 ID
		- title - name of the product
		- feature - bullet-point format features of the product
		- description - description of the product
		- price - price in US dollars (at time of crawl)
		- imageURL - url of the product image
		- imageURL - url of the high resolution product image
		- related - related products (also bought, also viewed, bought together, buy after viewing)
		- salesRank - sales rank information
		- brand - brand name
		- categories - list of categories the product belongs to
		- tech1 - the first technical detail table of the product
		- tech2 - the second technical detail table of the product
		- similar - similar product table
	- 清理空值過多等不可用資訊後，保留商品ID、類別排名、品牌、價格、使用者購買/ 瀏覽類似商品。
	- 主類別（main_cat）皆為All beauty一類、類別(category)皆為空值，下述之類別為排名(rank)中所記載的子分類，共10類。
	
## 實作方法

### 資料前處理

#### 類別、數值資料前處理
- 清除過多空值的欄位
- 清除重複商品(asin)的資料
- 清除使用者重複評論同個商品(asin)的資料
- 價格資料：清除多餘的符號、轉換資料型態
- 排名資料：分割字串，將排名與子分類切割

### 探索式資料分析(EDA)

- 分類：共有10個子分類，但幾乎都屬Beauty & Personal Care類別，其他類別並沒有太多購買量
  - Beauty & Personal Care共352006筆、其他加總共429筆
- 品牌：有些品牌較為暢銷，但探索資料後發現其熱銷的多為單項產品，表示並沒有太大的品牌效應
  - 像是有17056筆評論數的Waterpik，其中兩項產品各有8667, 8341筆評論
  - 用戶購買同品牌產品，多為同天的一筆訂單，之後回購的情況並不明顯
- 價格：95%的商品均在50元以下，且高銷量商品均為低價產品
- 排名：排名部分看不出明顯分布樣態
- 評論：5分評論佔據約6成的評論，確實有最高的銷量，但3分以上的商品也有不錯的購買量

#### 探索資料總結

- 將非Beauty & Personal Care類別的商品排除推薦名單外
- 將50元以上的商品排除推薦名單外
- 評價分數作為後續tuning使用的參數

### 推薦方法建立

#### 方法一：User-based Collaborative Filtering

- 針對過去用戶之購買記錄，建立使用者相似性之矩陣，推薦相似性最高的使用者曾購買的商品
- 嘗試過濾掉n(15/30/60/90/180/360)天之內的購買記錄
- 嘗試過濾掉評分在s(0/1/2/3/4/5)分以下的購買記錄
- 嘗試過濾掉出現總次數在3次以下的使用者的購買記錄

#### 方法二：Item-based Collaborative Filtering

- 針對過去用戶之購買記錄，建立物品相似性之矩陣，推薦與過去購買過的商品相似性最高的商品
- 嘗試過濾掉n(15/30/60/90/180/360)天之內的購買記錄
- 嘗試過濾掉評分在s(0/1/2/3/4/5)分以下的購買記錄

#### 方法三：Collaborative Filtering by surprise package

- 針對過去用戶之購買記錄，運用套件建立物品相似性之矩陣，推薦與過去購買過之商品相似性最高的商品
- 嘗試過濾掉n(15/30/60/90/180/360)天之內的購買記錄
- 嘗試過濾掉評分在s(0/1/2/3/4/5)分以下的購買記錄

### 參數調整(parameter tuning)

- 評分篩選：0/ 1/ 2/ 3/ 4/ 5 分以上
- 時間長度：15/ 30/ 60/ 90/ 120/ 150/ 180/ 360 天/ 不篩選

#### 方法一：User-based Collaborative Filtering

- 在方法一中，篩選評分筆數大於等於3的使用者後，使用者由323489人變成4793人。
- 之前實作時已經知道測試資料584名用戶中，只有38名曾經出現，而篩選過後，有進行推薦的僅3名用戶，推薦準度為0%。
- 篩選天數減少後，推薦筆數減少，最終推薦0名用戶，過程之間推薦準度均為0%。
- 嘗試完全不篩選的作法，但Colab的RAM資源不足以運算，且篩選3筆從概念上看來應已是合理的篩選標準，故保留。
- 因其作法僅推薦3名用戶，其他可以補上Content Base所進行的推薦，但實質上來說已經只是單純Content Based的作法，故不再額外進行超參數訓練與分析。

#### 方法二：Item-based Collaborative Filtering

- 在方法二中，由於以物品作為基礎故不針對評分筆數進行篩選。
- 測試資料584名用戶中，最終共針對32名進行推薦。
- 過濾評分在0-4、推薦筆數10至50筆，其推薦準度均為0.169%。
- 篩選天數減少後，推薦筆數減少，減少至120天以下時，推薦準度降至0%。
- 此運算模式較節省資源，即使不進行篩選仍能進行運算。

#### 方法三：Collaborative Filtering by surprise package

- 在方法三中，運用cosine similarity的方式建立相似度矩陣。
- 若不針對資料進行篩選，Colab的RAM資源不足以進行運算。
- 篩選天數在150天以上時，推薦準度與方法二相同，為0.169%。

## 總結

- 584名用戶中，只有38名曾經出現，因此考慮用戶過去資料進行相似推薦並不有效。
- 針對過去舊用戶的推薦準度為0.169%，檢視原始資料，僅成功預測1筆其購買項目。
- 這個資料也顯示在此電商平台中，用戶快速增加的情形，持續有新進用戶不斷加入，舊用戶流動率高，在分析時需將此要素納入考慮。

## 使用工具

- Python
	- package: gzip, json, datetime, pandas, surprise, collections, itertools
- Colab
- Github
