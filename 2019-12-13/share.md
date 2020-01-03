# 分享

## 关于php中strtotime函数边界值问题

```php
echo date("Y-m-d",strtotime("-1 month", strtotime('2018-07-31'))); // 2018-07-01

// first day of 
// last day of
echo date("Y-m-d",strtotime("last day of -1 month", strtotime('2018-07-31'))); // 2018-06-30
```

> 文档阅读：http://www.laruence.com/2018/07/31/3207.html