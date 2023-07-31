---
title: Spark ETL code tips
description: Tips for write Spark ETL application 
language: en
date: 2023-07-31
updated: 2023-07-31
taxonomies:
  categories:
    - Blogs
  tags:
    - Writing
---

Tips for write Spark ETL application
<!-- more -->

## Avoid use withColumn in loop
withColumn is very useful when operate on a column, but you should avoid using it in a loop.




