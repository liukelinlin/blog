---
layout: post
title:  "More neat and efficient SQL approaches to solve coding problem"
date:   2020-01-05 14:21:17 -0500
categories: Database
---
1. Show the 1984 winners and subject ordered by subject and winner name, but list Chemistry and Physics last. Data set table "noble" looks like:

![noble-data](https://liukelinlin.github.io/images/noble-data.jpg)

"order by" is based on columns' order. We can benefit from:

```
subject in ("Chemistry", "Physics") => 0, 1
```

So, the query looks like:

```
select winner, subject
from nobel
where yr = 1984
order by subject in ("Chemistry", "Physics"), subject, winner
```

2. List each continent and the name of the country that comes first alphabetically. Table "world" as follow: 

![continent-data](https://liukelinlin.github.io/images/continent-data.jpg)

The query leverages "ALL":

```
SELECT continent, name
FROM world x
where name <= ALL(SELECT name FROM world y WHERE x.continent = y.continent)
```

3. List odd rows in a table without auto-increment id in MySQL:

![odd-data](https://liukelinlin.github.io/images/oddrows-data.jpg)

```
set @i=0;
select tmp.col, tmp.test
from(
select @i:=@i+1 as id, col, test from nothing
) as tmp
where tmp.id%2 <>0;
```

4. List employee based on salary desc order and department.

![sql](https://liukelinlin.github.io/images/employee-data.jpg)

5. Delete duplicate rows and only keep one.

```
delete p1 from person p1
join person p2
where p1.id < p2.id and p1.full_name = p2.full_name
```
