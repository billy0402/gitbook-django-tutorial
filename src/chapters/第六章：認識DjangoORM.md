# 第六章：認識 Django ORM

## 開啟 Django Shell

Django Shell 是 Django 提供類似 IPython 的互動式介面，會對於你輸入的程式碼盡量產生立即的回應。
我們可以透過 Django Shell 來測試所寫的程式是否能夠正確執行，基本上 Django 的行為都可以在這裡想辦法重現。
Django Shell 雖然方便，但是他並不會幫你把輸入的東西輸出出來，所以測試完記得把需要的部分複製回你的程式碼當中。
我們在這邊主要是透過 Django Shell 來學習 Django ORM 的語法。

```shell
$ python manage.py shell
```

## import

Django Shell 每次重新開啟時都是全新的狀態，會把記憶體清空，所以請記得每次都要重新進行 import。
這裡主要會使用到 Coffee 的所有 model。
重新進入時，可以透過方向鍵去尋找歷史輸入，忘記輸出時也許有機會找回來，也方便每次重複進行或是做到一半的動作。

```python
>>> from coffees.models import *
```

`>>>` 是代表要輸入的部分，用來跟下方的回應做區分，測試時請輸入符號後方的程式碼部分。
上方的 SQL 指令，為教學用途，表示下面的 ORM 等效的 SQL 語法，實際上不會顯示。
只有查詢透過 print(qs.query)，可以直接看到查詢的 SQL，這部分下面會提到。
下面是以全新的資料為基底，如果有塞入過資料，可能會有欄位不能重複，或是回應不相同的問題。

## 新增單筆 (INSERT ONE)

### 新增產地資料 (為咖啡豆的產地關聯做準備)

基本上新增完會直接回傳新增後對應的 Model 物件。

```python
INSERT INTO "coffees_originplace" ("created_at", "updated_at", "name")
VALUES ('2021-05-01 07:46:34.361803', '2021-05-01 07:46:34.361866', '拉丁美洲');
>>> origin_place = OriginPlace.objects.create(name='拉丁美洲')

>>> origin_place
<OriginPlace: 拉丁美洲>
```

### 新增三筆咖啡豆資料

關聯物件要傳入對應的 Model 物件，ORM 會幫自動把關聯物件的 Primary Key 塞入 Foreign Key。

```python
INSERT INTO "coffees_coffee" ("created_at", "updated_at", "name", "description", "roast", "price", "origin_place_id")
VALUES ('2021-05-01 07:47:12.180462', '2021-05-01 07:47:12.180506', '咖啡一號', '一種咖啡豆', 'Medium', 300, 1);
>>> coffee1 = Coffee.objects.create(
    name='咖啡一號',
    description='一種咖啡豆',
    roast='Medium',
    price=300,
    origin_place=origin_place,
)

>>> coffee1
<Coffee: 咖啡一號>

INSERT INTO "coffees_coffee" ("created_at", "updated_at", "name", "description", "roast", "price", "origin_place_id")
VALUES ('2021-05-01 07:47:28.655375', '2021-05-01 07:47:28.655420', '咖啡二號', '一種咖啡豆', 'Medium', 350, 1);
>>> coffee2 = Coffee.objects.create(
    name='咖啡二號',
    description='一種咖啡豆',
    roast='Medium',
    price=350,
    origin_place=origin_place,
)

>>> coffee2
<Coffee: 咖啡二號>

INSERT INTO "coffees_coffee" ("created_at", "updated_at", "name", "description", "roast", "price", "origin_place_id")
VALUES ('2021-05-01 07:47:54.853298', '2021-05-01 07:47:54.853341', '咖啡三號', '這是一種咖啡豆', 'Medium', 400, 1);
>>> coffee3 = Coffee.objects.create(
    name='咖啡三號',
    description='這是一種咖啡豆',
    roast='Medium',
    price=400,
    origin_place=origin_place,
)

>>> coffee3
<Coffee: 咖啡三號>
```

## 查詢多筆 (SELECT ALL)

### 查詢全部資料

```python
SELECT * FROM "coffees_coffee";
>>> qs = Coffee.objects.all()

>>> qs
<QuerySet [<Coffee: 咖啡一號>, <Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>
```

### 查看 ORM 真正下的 SQL 語法

只有查詢可以直接透過 print(qs.query) 查看所下的 Query，因為增刪改會直接回傳修改後的資訊或物件。

```python
>>> qs.query
<django.db.models.sql.query.Query object at 0x102f92a90>

>>> print(Coffee.objects.all().query)
SELECT "coffees_coffee"."id",
       "coffees_coffee"."created_at",
       "coffees_coffee"."updated_at",
       "coffees_coffee"."name",
       "coffees_coffee"."description",
       "coffees_coffee"."roast",
       "coffees_coffee"."price",
       "coffees_coffee"."origin_place_id"
FROM "coffees_coffee";
```

### 查詢片段資料

QuerySet 可以直接透過切片的語法取得特定的片段。
但 [-1] 這種反向切片用在 QuerySet 上是不合法的。

```python
SELECT * FROM "coffees_coffee" LIMIT 1 OFFSET 2;
>>> Coffee.objects.all()[2]
<Coffee: 咖啡三號>

SELECT * FROM "coffees_coffee" LIMIT 2;
>>> Coffee.objects.all()[:2]
<QuerySet [<Coffee: 咖啡一號>, <Coffee: 咖啡二號>]>

SELECT * FROM "coffees_coffee" LIMIT 1 OFFSET 1;
>>> Coffee.objects.all()[1:2]
<QuerySet [<Coffee: 咖啡二號>]>
```

## 查詢單筆 (SELECT ONE)

這幾個在查詢是 `空值 (DoesNotExist)` 或是 `多筆 (MultipleObjectsReturned)` 時，都會直接拋錯，請確定一定只有一筆。

```python
SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."id" = 1 LIMIT 1;
>>> Coffee.objects.get(pk=1)
<Coffee: 咖啡一號>

SELECT * FROM "coffees_coffee" ORDER BY "coffees_coffee"."id" ASC LIMIT 1;
>>> Coffee.objects.first()
<Coffee: 咖啡一號>

SELECT * FROM "coffees_coffee" ORDER BY "coffees_coffee"."id" DESC LIMIT 1;
>>> Coffee.objects.last()
<Coffee: 咖啡三號>
```

## 透過數值欄位過濾 (SELECT WITH INTERGER FIELD)

這裡用價格欄位示範，將欄位加上雙底線再加上運算語法，依序是 `=`, `< (lt)`, `<= (lte)`, `> (gt)`, `>= (gte)`。

```python
SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."price" = 350;
>>> Coffee.objects.filter(price=350)
<QuerySet [<Coffee: 咖啡二號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."price" < 350;
>>> Coffee.objects.filter(price__lt=350)
<QuerySet [<Coffee: 咖啡一號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."price" <= 350;
>>> Coffee.objects.filter(price__lte=350)
<QuerySet [<Coffee: 咖啡一號>, <Coffee: 咖啡二號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."price" > 350;
>>> Coffee.objects.filter(price__gt=350)
<QuerySet [<Coffee: 咖啡三號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."price" >= 350;
>>> Coffee.objects.filter(price__gte=350)
<QuerySet [<Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>
```

## 排序 (ORDER_BY)

這裡用價格欄位示範，依序是 `由小到大`, `由大到小`, `亂數排序`，

```python
SELECT * FROM "coffees_coffee" ORDER BY "coffees_coffee"."price" ASC;
>>> Coffee.objects.order_by('price')
<QuerySet [<Coffee: 咖啡一號>, <Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>

SELECT * FROM "coffees_coffee" ORDER BY "coffees_coffee"."price" DESC;
>>> Coffee.objects.order_by('-price')
<QuerySet [<Coffee: 咖啡三號>, <Coffee: 咖啡二號>, <Coffee: 咖啡一號>]>

SELECT * FROM "coffees_coffee" ORDER BY RAND() ASC;
>>> Coffee.objects.order_by('?')
<QuerySet [<Coffee: 咖啡三號>, <Coffee: 咖啡一號>, <Coffee: 咖啡二號>]>
```

## 更新資料 (UPDATE)

這裡示範要修改資料的建立日期，方便我們示範後續的日期過濾跟排序。
我們首先要準備要修改的日期資料。

```python
>>> from django.utils.timezone import datetime
>>> days = [1, 15, 31]
>>> date1, date2, date3 = [datetime(2020, 1, day) for day in days]
>>> date1, date2, date3
(datetime.datetime(2020, 1, 1, 0, 0), datetime.datetime(2020, 1, 15, 0, 0), datetime.datetime(2020, 1, 31, 0, 0))
```

這裡可以看到 Django ORM 實際上是惰性的，它會要運算時，才真的去執行 SQL。

```python
>>> coffee1.created_at = date1

>>> coffee1.created_at
datetime.datetime(2020, 1, 1, 0, 0)

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."id" = 1 LIMIT 1;
>>> Coffee.objects.get(pk=1).created_at
datetime.datetime(2021, 5, 1, 7, 47, 12, 180462, tzinfo=<UTC>)

UPDATE "coffees_coffee" SET "created_at" = '2019-12-31 16:00:00' WHERE "coffees_coffee"."id" = 1;
>>> coffee1.save()


>>> coffee2.created_at = date2

UPDATE "coffees_coffee" SET "created_at" = '2020-01-14 16:00:00' WHERE "coffees_coffee"."id" = 2;
>>> coffee2.save()


>>> coffee3.created_at = date3

UPDATE "coffees_coffee" SET "created_at" = '2020-01-30 16:00:00' WHERE "coffees_coffee"."id" = 3;
>>> coffee3.save()
```

## 透過日期欄位過濾 (SELECT WITH DATETIME FIELD)

這裡用剛剛修改的建立日期欄位示範，一樣將欄位加上雙底線再加上運算語法，依序是 `=`, `< (lt)`, `<= (lte)`, `> (gt)`, `>= (gte)`。
我們可以直接用日期字串來比對，而不用建立 datetime 物件。

```python
SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."created_at" = '2020-01-14 16:00:00';
>>> Coffee.objects.filter(created_at='2020-01-15')
<QuerySet [<Coffee: 咖啡二號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."created_at" < '2020-01-14 16:00:00';
>>> Coffee.objects.filter(created_at__lt='2020-01-15')
<QuerySet [<Coffee: 咖啡一號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."created_at" <= '2020-01-14 16:00:00';
>>> Coffee.objects.filter(created_at__lte='2020-01-15')
<QuerySet [<Coffee: 咖啡一號>, <Coffee: 咖啡二號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."created_at" > '2020-01-14 16:00:00';
>>> Coffee.objects.filter(created_at__gt='2020-01-15')
<QuerySet [<Coffee: 咖啡三號>]>

SELECT * FROM "coffees_coffee" WHERE "coffees_coffee"."created_at" >= '2020-01-14 16:00:00';
>>> Coffee.objects.filter(created_at__gte='2020-01-15')
<QuerySet [<Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>
```

## 查詢多種條件 (SELECT WIHT MULTI CONDITION)

我們除了正向的 filter 來進行過濾外，也可以用反向的 exclude 來進行排除。
我們在下 ORM 時，開頭都會使用 Model.objects，實際上會對應到那個 Model 下的 Manager，由 Manager 來轉換成 ORM。
objects 後面的語法是可以無限串接的，Manager 會幫我們控制好，在真正運算時，一次全部串接好，再發給資料庫。

```python
SELECT * FROM "coffees_coffee"
WHERE ("coffees_coffee"."name" LIKE '%咖啡%' ESCAPE '\' AND "coffees_coffee"."price" = 300);
Coffee.objects.filter(name__contains='咖啡', price=300)
<QuerySet [<Coffee: 咖啡一號>]>

SELECT * FROM "coffees_coffee"
WHERE ("coffees_coffee"."name" LIKE '%咖啡%' ESCAPE '\' AND "coffees_coffee"."price" = 300);
Coffee.objects.filter(name__contains='咖啡', price=300)
<QuerySet [<Coffee: 咖啡一號>]>

SELECT * FROM "coffees_coffee"
WHERE ("coffees_coffee"."name" LIKE '%咖啡%' ESCAPE '\' AND NOT ("coffees_coffee"."price" = 300));
Coffee.objects.filter(name__contains='咖啡').exclude(price=300)
<QuerySet [<Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>
```

## 更新多筆 (UPDATE MANY)

這裡會把符合條件的資料都進行欄位更新，並回傳更新筆數。

```python
UPDATE "coffees_coffee" SET "roast" = 'High' WHERE "coffees_coffee"."price" > 300;
Coffee.objects.filter(price__gt=300).update(roast='High')
2
```

## 新增多對多資料 (ADD MANY TO MANT Field)

多對多無法直接塞在 create 的參數裡面，要透過下面這樣用欄位 add 的方式塞進去。

```python
INSERT INTO "coffees_tag" ("created_at", "updated_at", "name")
VALUES ('2021-05-01 07:56:32.746569', '2021-05-01 07:56:32.746610', '經典');
>>> tag_classic = Tag.objects.create(name='經典')

INSERT INTO "coffees_tag" ("created_at", "updated_at", "name")
VALUES ('2021-05-01 07:56:32.754009', '2021-05-01 07:56:32.754050', '水洗法');
>>> tag_wash = Tag.objects.create(name='水洗法')

INSERT OR IGNORE INTO "coffees_coffee_tags" ("coffee_id", "tag_id")
SELECT 1, 1 UNION ALL SELECT 1, 2;
>>> coffee1.tags.add(tag_classic, tag_wash)

SELECT * FROM "coffees_tag"
INNER JOIN "coffees_coffee_tags"
ON ("coffees_tag"."id" = "coffees_coffee_tags"."tag_id")
WHERE "coffees_coffee_tags"."coffee_id" = 1;
>>> coffee1.tags.all()
<QuerySet [<Tag: 經典>, <Tag: 水洗法>]>
```

## 刪除 (DELETE)

刪除時會把關聯的資料也一併清除，刪除基本上是不可逆的，請三思後再進行。
第一種是較為安全的單筆，第二種多筆刪除前，請確定好過濾正確。

```python
DELETE FROM "coffees_coffee_tags" WHERE "coffees_coffee_tags"."coffee_id" IN (1);
DELETE FROM "coffees_coffee" WHERE "coffees_coffee"."id" IN (1);
>>> coffee1.delete()
(3, {'coffees.Coffee_tags': 2, 'coffees.Coffee': 1})

SELECT * FROM "coffees_coffee";
>>> Coffee.objects.all()
<QuerySet [<Coffee: 咖啡二號>, <Coffee: 咖啡三號>]>

SELECT * FROM "coffees_coffee";
DELETE FROM "coffees_coffee_tags" WHERE "coffees_coffee_tags"."coffee_id" IN (2, 3);
DELETE FROM "coffees_coffee" WHERE "coffees_coffee"."id" IN (3, 2);
>>> Coffee.objects.all().delete()
(2, {'coffees.Coffee': 2})

SELECT * FROM "coffees_coffee";
>>> Coffee.objects.all()
<QuerySet []>
```