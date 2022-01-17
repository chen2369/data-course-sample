# 推薦系統實作

## 專案目的

- 將美妝類電商所擁有的資料集（包含評論資訊、商品後設資料）進行分析，以過去資料來推薦使用者會購買的商品。
- 評估方式：以過去資料作為依據，推薦k個商品給最後一個月購買商品的使用者，將使用者購買的商品是否在推薦名單之中作為分數計算。

### 實作「Rule-Based」的推薦系統

- [Readme](https://github.com/chen2369/data-course-sample/blob/main/README_hw1.md)
- [Code](https://colab.research.google.com/drive/1gP-B6XPPVbcmDPjNVNBuY0PLAKRwNKUY?usp=sharing)

### 實作「Content-Based」的推薦系統

- [Readme](https://github.com/chen2369/data-course-sample/blob/main/README_hw2.md)
- [Code](https://colab.research.google.com/drive/1BQbp7WVp10GIXn9nt82xVXcDiHlFQqMH?usp=sharing)

### 實作「Collaborative Filtering」的推薦系統

- [Readme](https://github.com/chen2369/data-course-sample/blob/main/README_hw3.md)
- [Code](https://colab.research.google.com/drive/1NGTXuwxneIBzxnkPHAOm4udT2RnnpGh3?usp=sharing)

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
  
  
## 使用工具

- Python
	- package: gzip, json, datetime, pandas, numpy, matplotlib, nltk, scikit-learn, surprise, collections, itertools
- Colab
- Github
