![img](../../../_posts/images/routine.png)

- [sqlmap 项目剖析1](https://www.anquanke.com/post/id/262848)

- [sqlmap 项目剖析2](https://www.anquanke.com/post/id/262847)

- [sqlmap 项目剖析3](https://www.anquanke.com/post/id/262849)

  sqlmap 使用一种极其巧妙的方式组合生成一个完整的 payload，一个完整的 payload 由如下几个部分组成：

  ```xml
  <prefix> <test> <comment> <suffix>
  ```

  其中 prefix、comment、suffix 作为 boundary，boundary 用于闭合注入点的前后部分；test 则是最终如果闭合成功后必然执行的语句。

  因此 sqlmap 将 prefix 与 suffix 单独作为 boundaries 保存，而 test 和 comment 则根据注入方式和数据库的不同被划分为六个文件（路径：`/data/xml/payloads`）

  ![img](../../../../../../知识库/blog/source/_posts/images/t01d7dbc46705981404.jpg)

- [sqlmap 项目剖析4](https://www.anquanke.com/post/id/262850)

  布尔注入

  ```
  先发送一个 false 请求，如果结果与原页面相同就判断不存在布尔注入
  如果不相同发送一个 true 请求,如果与原页面相似
  则再发送一个 false 请求，然后把这两次返回的结果取差集，计算出True时的flag标识，也就是sqlmap的-string参数
  ```

  报错注入

  ```
  发送报错payload，如果能够正则匹配到则说明存在报错注入。
  ```

  延时注入

  ```
  先发送N个正常请求，然后计算这N个请求响应时间的标准差和正常请求的最长响应时间
  然后发送一个延时请求，判断是否在区间内
  ```

  联合查询注入

  ```
  用order by和二分法来判断列数
  遍历列数判断哪一列是回显点
  发送payload
  ```

- [sqlmap源码分析与学习](https://www.beysec.com/security/sqlmap-source-1.html)

- [sqlmap 流程脑图](https://www.processon.com/view/5835511ce4b0620292bd7285)

os-shell 原理

- [sqlmap --os-shell原理](https://xz.aliyun.com/t/7942)
- [Sqlmap之os-shell原理分析](https://mp.weixin.qq.com/s?__biz=MzIyMjkzMzY4Ng==&mid=2247485339&idx=1&sn=ea76ee0d56b8a95a118a60d111d48160)

攻防

- [实战sqlmap绕过WAF](https://xz.aliyun.com/t/10385)
- [sqlmap --os-shell反制小思路](https://www.anquanke.com/post/id/261915)
- [入侵检测之sqlmap恶意流量分析](https://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247494179&idx=1&sn=e6c94b87981fda009e7be50c9eb73bf6)

