# JDATA2

比赛网址：https://jdata.jd.com/html/detail.html?id=2

队伍：WTF
队伍成员： 崔世文，文超，梁策远


中国大数据算法大赛-用户购买时间预测
比赛攻略
B榜Rank：rank4
队伍名称：  WTF
队伍成员： 梁策远，许文超，崔世文
第一部分：赛题解读
数据理解：本赛题基于京东10万用户在过去一年的对某类商品的浏览和购买的情况，来预测未来一个月内是否发生购买，以及购买的时间。
 
目标解读：
S1：用来评价用户是否购买的指标。
S2：用来评价用户购买的日期。评分公式中，平方误差项在分母，因此预测值偏小会更符合评分的倾向

第二部分：算法核心设计思想

算法核心：
1.	样本构建
算法核心就是合理使用滑动窗口，提取特征时使用滑动窗口， 构造样本时又可以使用滑动窗口，而滑动窗口的大小， 以及滑动的次数都要通过建立合理的线下测试集来判断。
2.特征选择和获取：
1)	关键特征
主要是用户的行为特征：
1.	用户的最后几次购买的时间间隔及相关统计特征（主要为mean，min，max，std）
2.	用户最后几次购买的花费
3.	用户最后一次action/order时间与下个月1号的时间距离
4.	用户以往购买数量及相关统计特征（标签月份前k天的相关统计，k=5，10，20，30，60，… , 270, 300）
5.	用户等级
6.	用户偏好特征，使用历史数据，训练用户的(操作，购买)在（sku_id, para_2, para_3）的偏好，对应一个5维度的用户向量（lda, nmf, svd），相当于用户属性
7.	用户转化率（用户自身转化率，以及所购买商品转化率的平均值）
8.	用户购买商品间隔时间的统计特征（主要为mean，min，max，std）
9.	其他cate【1，46，71，83】的关键特征

2)	特征思考和构建方式
我们从两个角度思考特征，一是用户相关，二是商品相关。
对于用户信息，我们将其分为两种信息，一种是近期信息，这种信息体现用户近期行为习惯，主要存在与最近几次的购买或者其他行为（浏览，关注等）。从数据可以看出大部分用户购买目标商品具有周期性，而并非一次性购买，因此最近一次购买的时间和数量会是很重要信息，同时根据实际生活场景，当次或者当月用于购买目标商品的花费也是重要的信息。第二种是长期信息，这种信息能体现用户的消费水平和习惯，主要通过提取用户在特定时间窗口内（比如1、3、5、7个月内）的一些特征，比如购买数量，购买过多少种不同商品等。
对于商品信息， 它的作用是侧面反映出用户的属性，比如消费水平。针对某个用户所购买的所有目标商品的相关信息（price, para_1, para_2, para_3）等进行统计（mean, min,  max, median等），提取出此部分信息。

3)	特征工程技巧
1、利用用户和商品（商品属性）的共现信息，构建用户商品（商品属性）共现矩阵，通过矩阵分解算法，得到用户的K维度隐向量表示，表征用户商品购买偏好。
2、利用B榜选定训练集时序前的A榜数据训练模型，将该模型对于B榜的训练集和测试集的预测作为新的一维特征。
3、利用B榜选定训练集时序前的B榜部分数据训练模型，将该模型对于B榜的训练集和测试集的预测作为新的一维特征。

4)	特征处理
使用pca，lda，nmf对商品，para2，para3特征进行了维度变换
第三部分：关键代码
我们团队3名队员均采用Lightgbm模型，此模型效果要优于其他大部分模型。
 样本构建及特征提取
