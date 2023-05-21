---
title: "Export data from Wacai app"
date: 2023-05-21T21:28:34+08:00
draft: false
categories: ["tech", "life"]
featuredImage: "cover.webp"
---

I have been using Wacai app for 8 years. Maybe because of habit, I have not changed to use other apps. However, recently I can't stand the splash ads. Oftentimes if my phone shakes a little when opening the app, it jumps to the ad page. So I started to think about exporting the data. But this app requires a membership to export data. I guess user data is the product itself. 

At first, I wanted to grab data directly from Wacai's official website. Later I found [an article](https://editst.com/2020/export-wacai-data/) that says the database can be directly exported (with some tools), which seems much more convenient.

Basically the idea still works, just need to change some table names.

## Extract the database file

This step requires an android device with root access.

1. Install an Android emulator.
2. Install Wacai app on the emulator.
3. Login to Wacai and let it syncs.
4. Turn on root feature from the emulator.
5. Copy this file to your computer.

> /data/data/com.wacai365/databases/wacai365.so

## View the database

Install [DB Browser for SQLite](https://sqlitebrowser.org/) to open the database file mentioned above.

The most important data is stored in **TBL_TRADEINFO**。

## Extract expend transactions

```sql
SELECT DISTINCT o.parentName as 支出大类, o.name 支出小类, a.name 帐户, NULL as 币种, '日常' as 项目, NULL as 商家, '非报销' as 报销, datetime(t.date, 'unixepoch', 'localtime') 消费日期, t.money*1.0/100 消费金额, t.money*1.0/100 成员金额, t.comment 备注, '日常账本' as 账本
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
LEFT JOIN TBL_OUTGOCATEGORYINFO o ON t.typeUuid = o.uuid
WHERE t.tradetype = 1
ORDER BY t.date desc;
```

## Extract income transactions

```sql
SELECT DISTINCT i.name 收入大类, a.name 帐户, NULL as 币种, '日常' as 项目, NULL as 付款方, datetime(t.date, 'unixepoch', 'localtime') 收入日期, t.money*1.0/100 收入金额, NULL as 成员金额, t.comment 备注, '日常账本' as 账本
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
LEFT JOIN TBL_INCOMEMAINTYPEINFO i ON t.typeUuid = i.uuid
WHERE t.tradetype = 2
ORDER BY t.date desc;
```

## Extract transfer transactions

```sql
SELECT DISTINCT datetime(t.date, 'unixepoch', 'localtime') 时间, NULL as 分类, '转账' as 类型, t.money*1.0/100 金额, a.name 帐户1, a2.name 账户2, t.comment 备注, NULL as 账单标记, NULL as 账单图片
FROM TBL_TRADEINFO t
INNER JOIN TBL_ACCOUNTINFO a ON t.accountUuid = a.uuid
INNER JOIN TBL_ACCOUNTINFO a2 ON t.accountUuid2 = a2.uuid
INNER JOIN TBL_BOOK b ON t.bookUuid = b.uuid
WHERE t.tradetype = 3
ORDER BY t.date desc;
```

