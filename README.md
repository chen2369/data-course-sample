# 作業1 - 實作「rule-based」的推薦系統

###### tags: `alphacamp`, `recommendation system`

## 預期產出

- 一份文字敘述：Readme，包含題幹敘述中的兩個問題。
	- 清楚記載這個專案的目的和結果，最後的推薦分數是多少，是否有成功
 	- 簡明清楚的使用說明：用了哪些工具和方法？為什麼？
- 兩支連結：包含 colab code、github code。
 	- [colab code][https://colab.research.google.com/drive/1gP-B6XPPVbcmDPjNVNBuY0PLAKRwNKUY?usp=sharing]
 	- [Github code][https://github.com/chen2369/data-course-sample]

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
	- 主類別（main_cat）皆為All beauty一類、類別(category)皆為空值，下述之類別為排名(rank)中所記載的分類，共10類。
	
## 實作方法

- 針對每一種方法，皆針對評論資訊做時間序列的篩選，篩選與2018-09-01距離5/10/30/60/90/180/360/720天的評論資料。
- 針對每一種方法，皆嘗試將k設為10/20/30/40/50。

### 方法一：考慮使用者過去資料

- 針對用戶過去的所有評論資料，依照商品過去購買量排序，依序加入：
	- 其購買過的品項、其他使用者購買品項(also_buy)。
	- 其他使用者瀏覽品項(also_view)。
	- 與此品項相同品牌、類別之品項。
	- 與此品項相同品牌之品項。
	- 與此品項相同類別之品項。
- 加入最熱門的品項
- 將前k項列入該用戶之推薦名單。

### 方法二：Top 10%

- 篩選商品資料中，各類別在過去最熱賣的10％商品，並依照購買量排序。
- 依序加入其商品，以及其使用者也購買之商品(also_buy)。
- 將前k項列入該用戶之推薦名單。

### 方法三：Hottest 最熱賣商品

- 篩選商品資料中，過去最熱賣的商品，並依照購買量排序。
- 將前k項列入該用戶之推薦名單。

## 專案結果

### 方法一：考慮使用者過去資料

- 在此方法中，可以發現在k=10時，篩選近5-10天的表現最佳，然而隨著k的上升，篩選天數提高能夠有更好的表現。
- 某些時候k上升反而沒有更好的表現，是因為在依序加入商品時，有因k值不同而選取不同比例造成。
- 在k=10時，篩選近5-10天能有約16％的推薦準度。
- k若上升至30/40時，篩選天數兩個月能有25％的推薦準度。

| Days\ k | 10 | 20 | 30 | 40 | 50 |
| --- | -------- | -------- | -------- | -------- | -------- |
| 5   |	0.159322 | 0.166102 | 0.166102 | 0.179661 | 0.108475 |
| 10  |	0.157627 | 0.206780 | 0.216949 | 0.222034 | 0.164407 |
| 30  |	0.142373 | 0.216949 | 0.247458 | 0.240678 | 0.174576 |
| 60  |	0.140678 | 0.223729 | 0.250847 | 0.259322 | 0.183051 |
| 90  |	0.135593 | 0.220339 | 0.218644 | 0.238983 | 0.172881 |
| 180 |	0.098305 | 0.150847 | 0.186441 | 0.223729 | 0.137288 |
| 360 |	0.098305 | 0.125424 | 0.155932 | 0.189831 | 0.105085 |
| 720 |	0.098305 | 0.111864 | 0.138983 | 0.145763 | 0.054237 |

### 方法二：Top 10%

- 在此方法中，可以發現在多數情況中，推薦準度均在8-9%之間。
- Days的變動均沒有太大的變化，推測可能是排名是依據過去所有的購買量來排序，故變動不會對準度有太大的影響。
- k的變動沒有太大的影響，主要是因為在此方法中加入了商品以及其他購買(also view)，若移除其他購買則與方法三結果相近。

| Days\ k | 10 | 20 | 30 | 40 | 50 |
| --- | -------- | -------- | -------- | -------- | -------- |
| 5   | 0.083051 | 0.083051 | 0.083051 | 0.083051 | 0.083051 |
| 10  | 0.083051 | 0.083051 | 0.096610 | 0.096610 | 0.096610 |
| 30  | 0.083051 | 0.083051 | 0.083051 | 0.084746 | 0.084746 |
| 60  | 0.083051 | 0.083051 | 0.083051 | 0.084746 | 0.084746 |
| 90  | 0.083051 | 0.083051 | 0.083051 | 0.083051 | 0.083051 |
| 180 | 0.083051 | 0.083051 | 0.083051 | 0.096610 | 0.096610 |
| 360 | 0.083051 | 0.083051 | 0.083051 | 0.096610 | 0.096610 |
| 720 | 0.083051 | 0.083051 | 0.083051 | 0.083051 | 0.083051 |

### 方法三：Hottest 最熱賣商品

- 在此方法中，可以發現在k=10時，與方法一相同Days約在5-10左右有最好的推薦準度(11%)。
- 與方法一不同的是，隨著k上升，Days仍是5-10左右有最好的推薦準度，在k=50時，能夠有近30%的推薦準度，比方法一有更好的表現。

| Days\ k | 10 | 20 | 30 | 40 | 50 |
| --- | -------- | -------- | -------- | -------- | -------- |
| 5   | 0.113559 | 0.144068 | 0.233898 | 0.255932 | 0.262712 |
| 10  | 0.113559 | 0.147458 | 0.184746 | 0.261017 | 0.306780 |
| 30  | 0.105085 | 0.123729 | 0.155932 | 0.172881 | 0.194915 |
| 60  | 0.105085 | 0.110169 | 0.123729 | 0.147458 | 0.161017 |
| 90  | 0.083051 | 0.105085 | 0.110169 | 0.118644 | 0.140678 |
| 180 | 0.083051 | 0.083051 | 0.083051 | 0.091525 | 0.105085 |
| 360 | 0.083051 | 0.083051 | 0.083051 | 0.091525 | 0.105085 |
| 720 | 0.083051 | 0.083051 | 0.083051 | 0.091525 | 0.105085 |

## 使用工具

- Python
	- package: gzip, json, pandas, numpy, datetime
- Colab
- Github
