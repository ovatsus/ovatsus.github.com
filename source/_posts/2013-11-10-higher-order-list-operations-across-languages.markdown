---
layout: post
title: "Higher order list operations across languages"
date: 2013-11-10 21:55
comments: true
categories: F#
---

Did this table out of curiosity, decided to share it:  

<pre>
----------------------------------------------------------------------------------------------
| F#         | C#         | Scala       | Clojure | Python | Ruby     | Haskell   | SQL      |
|------------|------------|-------------|---------|--------|----------|-----------|----------|
| map        | Select     | map         | map     | map    | collect  | map       | Select   |
| filter     | Where      | filter      | filter  | filter | select   | filter    | Where    |
| fold       | Aggregate  | foldLeft    | reduce  | reduce | inject   | foldl     |          |
| foldBack   |            | foldRight   |         |        |          | foldr     |          |
| reduce     | Aggregate  | reduceLeft  | reduce  | reduce | inject   | foldl1    |          |
| reduceBack |            | reduceRight |         |        |          | foldr1    |          |
| collect    | SelectMany | flatMap     | mapcat  |        | flat_map | concatMap | From     |
| exists     | Any        | exists      | some    | any    | any?     | any       | Exists   |
| forall     | All        | forall      | every?  | all    | all?     | all       |          |
| sortBy     | OrderBy    | sortBy      | sort-by | sorted | sort_by  | sortBy    | Order By |
----------------------------------------------------------------------------------------------
</pre>

PS: please correct me if I got something wrong
