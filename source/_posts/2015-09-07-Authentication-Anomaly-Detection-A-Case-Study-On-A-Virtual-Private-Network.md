title: 'Authentication Anomaly Detection: A Case Study On A Virtual Private Network'
date: 2015-09-07 10:57:34
tags:
categories: paper
---

论文下载地址: [Authentication Anomaly Detection: A Case Study On A Virtual Private Network](https://www3.nd.edu/~dial/papers/MINENET07.pdf)

这篇论文介绍了一种基于EM分类算法，然后通过zscore进行异常检测的思路。分析的对象是vpn的登录日志。

## 聚类与异常检查

基于EM算法对vpn的登录成功的异常进行判定

验证方式：ten-fold cross-validation with 100 iterations

outlier detection: 通过计算z-score来检查clutser-specific的异常。（即分组后利用告诉分布来检测异常）

人工选择z-score的异常的阈值。

<!-- more -->

## 数据与特征提取

从每一条登录成功的日志中获取下面四个属性
* 用户id
* 来源ip
* 时间戳
* Group（vpn账号的用户组，不同的用户组，有不同的权限）

用户id和用户组，是一个多对多的关系，一个用户，可以属于多个组，一个组可能有多个用户

其他属性的提取：

在论文的分析用，也使用了很多从上面属性中抽取的特征。
1. 通过用户id获取用户的组织关系，所属机构等信息
2. 将学校机构的日历表（例如会议时间，课程之类的）与时间戳，可以提取行的特征
3. 从时间戳中，提取一周中的工作日，休息日，一天中的工作时间，下班的时间，一天的课程时间
4. 将用的来源ip分为，校园的内网，外网，校园的管理网络
5. 最后添加一个ip段到地理位置的映射信息，然后计算从ip的地理位置到校园中心的距离，作为一个重要的特性
   在测试数据中，最近的最小距离为0公里，最大为10874公里，平均71.26公里，标准差为441.55公里


## 评估

通过聚类算法，数据被分成了5组

1. Cluster A (Close Weekday): 所有的log都是发生在工作日，大部分的链接都是从校外网附近，但是距离很近，平均距离是0.83公里，平均差为0.45公里。绝大部分的用户都是普通的组的用户，没有任何特权的远程链接。

2. Cluster B (Close Weekend): 这个分组和前面的一组类似，但有两个地方不一样，第一，这部分的vpn链接是发生在周末，第二，距离要稍微远一点， 平局3.45公里，标准方差是6.34公里。

3. Cluster C (Remote Weekday): 这一类分组，距离就比较远，平局200.47 miles, 方差278.41 miles. 这类日志发生在工作日，普通链接需求。

4. Cluster D (Remote Weekend): 这一类分组与cluster C相同，都是在周末的vpn链接，平均距离1240.83公里，标准差1800.48公里

5. Cluster E (Utility): 一类分组与前面的不一样，这一类认证都来自获取特殊权限的组，这一类用户，都是来自哪些被事先定义好的用户组，都没有组织等信息。一般都是发生在工作日，而且请求都来自校园内。目前工作日时间是10.73，标准差3.14

## 分组内的分布
参考论文

## outlier评估

选择z-score <=-3, 在文章的例子里面，没有z-score 大于3的
正常的占了99.7%，

然后由人工，对这个数据进行分析。

* 系统效率
Anomaly detection systems typically serve as a component of a larger intrusion detection infrastructure consisting of signature detection systems and other log analysis tools.

true positive 7
false positive 16
suspicious event: 12

* true positive
4 are 'unusual distant connections'
3 are 'unauthorized use of group accout', 使用shared account，连接到了具有更高权限的用户组。

* 可疑的事件
1. Connection to a privileged VPN group from a remote location2
2. Connection from very distant (> 1000 miles) location
3. connection during very late hours

* false positive

The vast majority (81%) of these false positive events were directly attributable to errors in the geographical IP address data used in this analysis. 

## 可以改进的地方

em聚类算法是一个比较可靠的算法，它可检查较少数的异常，用于更进一步的调查。

未来可以进一步研究的：
1. 本文用到的方法，都是手工的，对实时性要求不高，无法再生产环境中使用，未来需要改进
2. 本文的模型来源于一个单独时间内的数据。但实际上组织内部的行为是变化的，那么就需要随着时间的进行，调整模型。在未来需要研究随着时间，而自主调整的模型。
3. **地理位置到ip的映射，对结果影响比较大，可以通过管理员的知识，和人工调整，改进地理位置的准确性。**
4. 同样的技术，可以用例如网络流量的监控，操作的审查信息等监控。
