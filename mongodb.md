# Mongodb

- [Mongodb](#mongodb)
  - [Atlas](#atlas)
  - [Types](#types)
    - [How to use Time in Mongodb](#how-to-use-time-in-mongodb)
  - [Orders](#orders)


## Atlas

DashBoard : https://cloud.mongodb.com/v2/619cd96f435b7205c0014abc#clusters

## Types

### How to use Time in Mongodb

There are three ways to express time:
1. 时间戳 timestamp
2. UTCDateTime
3. 日期字符串

Belows are some methods about time:
1. `Date()` : `'Wed Nov 24 2021 21:18:04 GMT+0800 (China Standard Time)'` Beijing Time, the type is string
2. `new Date()`: `2021-11-24T13:18:25.287Z` GMT Time, the type is object
3. `ISODate()`: `2021-11-24T13:18:25.287Z` GMT Time, the type is object

## Orders

1. switch db: `use myNewDB`
2. find all `documents` from `collections`: `db.observations.find({})`
3. find some `documents` with `equal` conditions: `db.observations.find({"extra":"favorite"})`
4. find some `documents` with `greate or less` conditions: `db.observations.find({"observationTimestamp":{$gt:1637758459}})`