---
title: "从挖财导出数据"
date: 2023-05-21T21:28:34+08:00
draft: false
categories: ["tech", "life"]
featuredImage: "cover.webp"
---

我用挖财记账已经8年了，可能是因为习惯，一直没有换用其他 app。不过这段时间越来越受不了开屏广告了，经常由于手机摇动一下或者误触而跳转到广告页面，就开始萌生把数据导出的想法。可是挖财是要会员才允许导出的，用户数据是它产品的一部分，早点跳船是好事。

我一开始是想在挖财的官方网站直接抓数据的，后来找到别人写好的文章，可以直接把数据库导出来，似乎更方便：

[无需会员导出挖财记账数据](https://editst.com/2020/export-wacai-data/)

我按照这个方法，基本上可以行得通，不过挖财应该修改了表名，需要进行一定改动。

## 获取数据库文件

这一步需要 root 了的安卓设备，我的提取过程如下：

1. 装某安卓模拟器
2. 在模拟器里面安装挖财
3. 登录挖财 app，让它同步数据
4. 打开 root 功能
5. 想办法把这个文件提出来放到电脑

> /data/data/com.wacai365/databases/wacai365.so

## 查看数据库

安装 [DB Browser for SQLite](https://sqlitebrowser.org/) 这个软件，打开上面提到的数据库文件。

重要的数据在 **TBL_TRADEINFO** 这个表里面。

## 获取支出数据

```sql
SELECT DISTINCT o.parentName as 支出大类, o.name 支出小类, a.name 帐户, NULL as 币种, '日常' as 项目, NULL as 商家, '非报销' as 报销, datetime(t.date, 'unixepoch', 'localtime') 消费日期, t.money*1.0/100 消费金额, t.money*1.0/100 成员金额, t.comment 备注, '日常账本' as 账本
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
LEFT JOIN TBL_OUTGOCATEGORYINFO o ON t.typeUuid = o.uuid
WHERE t.tradetype = 1
ORDER BY t.date desc;
```

## 获取收入数据

```sql
SELECT DISTINCT i.name 收入大类, a.name 帐户, NULL as 币种, '日常' as 项目, NULL as 付款方, datetime(t.date, 'unixepoch', 'localtime') 收入日期, t.money*1.0/100 收入金额, NULL as 成员金额, t.comment 备注, '日常账本' as 账本
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
LEFT JOIN TBL_INCOMEMAINTYPEINFO i ON t.typeUuid = i.uuid
WHERE t.tradetype = 2
ORDER BY t.date desc;
```

## 获取转账数据

```sql
SELECT DISTINCT datetime(t.date, 'unixepoch', 'localtime') 时间, NULL as 分类, '转账' as 类型, t.money*1.0/100 金额, a.name 帐户1, a2.name 账户2, t.comment 备注, NULL as 账单标记, NULL as 账单图片
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_ACCOUNTINFO a2 ON t.accountUuid2 = a2.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
WHERE t.tradetype = 3
ORDER BY t.date desc;
```

