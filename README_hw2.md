# 作業2 - 實作「content-based」的推薦系統

###### tags: `alphacamp`, `recommendation system`

## 預期產出

- 一份文字敘述：Readme，包含題幹敘述中的兩個問題。
	- 清楚記載這個專案的目的和結果，最後的推薦分數是多少，是否有成功
 	- 簡明清楚的使用說明：用了哪些工具和方法？為什麼？
- 兩支連結：包含 colab code、github code。
 	- [colab code](https://colab.research.google.com/drive/1BQbp7WVp10GIXn9nt82xVXcDiHlFQqMH?usp=sharing)
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
- 清除重複asin的資料
- 價格資料：清除多餘的符號、轉換資料型態
- 排名資料：分割字串，將排名與子分類切割

#### 文字資料前處理
- 將標題與描述合併成文字資料
- 移除html tag
- 轉換成小寫(lower)
- 斷詞(tokenize)
- 移除英文停用詞(stopword)
- 詞形還原(lemmatize)
- 觀察最高頻的字詞(ex: skin, hair, oil...)，確認結果無誤

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

### 文字分析

- 本次欲以文字描述作為商品分類、觀察相似性的重要指標
- 將文字資料以TF-IDF演算法進行處理
- 將資料運用分群演算法進行分群（k-means, Agglomerative)
- 將資料運用Cosine Similarity計算相似性

### 推薦方法建立

#### 方法一：考慮使用者過去資料

- 針對有評論記錄的舊用戶，根據其評論記錄進行相似商品推薦，以Cosine Similarity進行運算

#### 方法二：將資料篩除分群後，以評論數量做排序進行推薦

- 如前述探索資料分析進行資料清理
- 以文字分析特徵進行分群，共分為10類
- 將各群依照評論數量進行排序後推薦，選擇準度最高的分群

## 專案結果

### 參數調整(parameter tuning)

- 分群方法：K-means, Agglomerative
- 分群：共10群
- 評分篩選：3/ 4/ 5 分以上
- 時間長度：10/ 15/ 20/ 30/ 60/ 180 天
- 推薦筆數：10/ 20/ 30/ 40/ 50 筆

### 方法一：考慮使用者過去資料

僅不到1成的用戶留有評論記錄，針對這些用戶進行推薦，其推薦準度為0%，故放棄此方法

### 方法二：將資料篩除分群後，以評論數量做排序進行推薦

- 分群方法：Agglomerative較佳，大約高1%左右
- 分群：第1群最佳，約15%上下，其他群最高僅不到3%，甚至有多群接近0%
- 評分篩選：3分以上最佳，約高2-5%
- 時間長度：15天最佳，相較其他時間高至少1-3%

|   k   | 10 | 20 | 30 | 40 | 50 |
| ----- | -------- | -------- | -------- | -------- | -------- |
| score | 0.167797 | 0.191525 | 0.205085 | 0.210169 | 0.210169 |

## 總結

- 在預設k=10時，表現約16.8%，略高於上次的作法
- k提高之後，表現提升的程度低於上次的作法，推論是因為經過分群，筆數上升可以考慮推薦其他分群中較多評論數的商品。
- 584名用戶中，只有38名曾經出現，因此考慮用戶過去資料進行相似推薦並不有效。
- 這個資料也顯示在此電商平台中，用戶快速增加的情形，持續有新進用戶不斷加入，舊用戶流動率高。

## 使用工具

- Python
	- package: gzip, json, datetime, matplotlib, pandas, numpy, nltk, collections, sklearn
- Colab
- Github
