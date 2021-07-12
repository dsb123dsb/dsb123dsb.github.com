---
title: 一段replace into 引发的血案
date: 2019-01-13 15:16:02
tags: [mysql]
categories: 基础杂谈
type:
---
>纸上得来终觉浅，绝知此事要躬行

最近项目更新数据库，由于语句使用不善，踩了许多坑，特做此总结引以为戒。

在向表中插入数据的时候，经常遇到这样的情况：1. 首先判断数据是否存在； 2. 如果不存在，则插入；3.如果存在，则更新。

通常使用`replace into`和`insert into on deplicate key update`

<!--more-->

执行的逻辑

1. 遇到PRIMARY KEY或UNIQUE索引的，新记录与旧记录有冲突的（这里实际产生了异常duplicate key error），replace into会把旧记录删除，然后再插入新记录， insert into on deplicate key update则只会更新。

2. 若是新记录没有冲突，就直接插入一条新记录，与insert into一样。

关键问题就在第一条，本人也是这里踩的坑

针对第一种逻辑会有问题

- 把旧记录删除之后，插入的新记录只是插入了那些指定的字段，原本不想更新的字段，直接为默认值了，会导致数据丢失

- 若旧记录的id跟其他表是有关联的，更新后新记录会产生新的id，导致这种关联丢失

- 而且使用replace into会导致自增主键id一直增大，很容易导致id值范围不够用

- 若是数据库存在主从关系，在主机器上进行了replace into操作之后，从机器上对应表的AUTO_INCREMENT是不会更新的，导致从机器转为主机器时，新插入数据会出现异常，直到AUTO_INCREMENT增加到原来主机器的值为止。 

