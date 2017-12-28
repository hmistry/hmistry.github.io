---
layout: post
title:  ActiveRecord vs Sequel Performance
date:   2017-12-11
categories: software
tags: ruby gems performance
excerpt_separator: <!--more-->
---

I ran some benchmarking tests to evaluate the performance difference, if any, between ActiveRecord and Sequel ORM's - two common ORM's in the Ruby world. I measured two sets of benchmarks: speed, and memory consumed. In addition, each set was measured against two different databases: SQLite3, and PostgreSQL. The results are summarized below.
<!--more-->

**Updated:** (2017-12-27) I updated the benchmarks to account for ActiveRecord lazily typecasting data on attribute access and Sequel typecasting immediately on retrieval from the database. For an apples-to-apples comparison, the benchmarks now force ActiveRecord to typecast the data so the total computation time and memory used is accounted for in the results for both ORM's. I also added a set of benchmarks for Sequel with `sequel_pg` gem for PostgreSQL database - the `sequel_pg` gem replaces the Sequel postgres adapter row fetching and typecasting code with a C version that improves the performance.

<br>
## Setup
1. All the tests were done on a local machine (2013 MacBook Pro) i.e. both the database and ruby script ran on the same machine for simplicity and time.
2. Versions used: Ruby v2.4.2, ActiveRecord v5.1.4, Sequel v5.2.0, SQLite v3.16.0, and PostgreSQL v9.6.5.
3. Used `benchmark-ips` and `memory_profiler` gems to obtain performance measurements.
4. Ran the test on SQLite3 and PostgreSQL databases.
5. The database schema had 3 models - Topic, Post, and Comment - with the following relations - topic has many posts and post has many comments. The database was seeded with 10 topics, 100 posts, and 20,000 comments.
6. Both ORM's were configured to write a log file like in a real world application. Logging did affect the speed - made it slower.
7. Benchmark code is available on Github [here](https://github.com/hmistry/ar_sql).
8. The tests are grouped as follows:
   1. Loading the ORM
   2. Creating records
   3. Updating records
   4. Querying records
   5. Queries - Lazy typecasting & Timestamps

<br>
## Results:
Things to note:
- Speed performance - results are shown for both PostgreSQL and SQLite3 databases.
- Speed is measured in iterations per second, except load time which is in seconds. Higher is better for iterations per second and lower is better for seconds (load time).
- Memory usage performance - only results for PostgreSQL are shown because results for SQLite3 were very similar and did not offer much insights.
- Memory is measured in bytes and number of objects for memory allocated and retained (total of 4 measurements). Lower is better for memory allocation and retention.

**Key:**<br>
AR => ActiveRecord gem v5.1.4<br>
Sql => Sequel gem v5.2.0<br>
Sql + sql_pg => Sequel gem v5.2.0 + Sequel_pg gem v1.8.1 (for PostgreSQL database only)<br>

<br>
### 1. Loading the ORM
Sequel beats ActiveRecord by ~3x in both speed and memory in loading.
- It loads in 0.1s vs 0.3s for ActiveRecord.
- It allocates 6Mb in memory vs 20Mb for ActiveRecord.
- It doesn't defer any memory allocation to the first query run unlike ActiveRecord which defers further memory allocation of 200Kb to the first query run (after require). A typical ActiveRecord query is in the 10's Kb (in this test scenario).

<br>
**ORM Load Time**

![Load Time]({{ "/assets/images/load-time.png" | absolute_url }}){: .post-halfwidth-image }

<br>
**ORM Memory Usage**

![Load Memory]({{ "/assets/images/load-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Memory]({{ "/assets/images/load-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
**ActiveRecord First Run Memory Usage**
<br>
The very first use of any ActiveRecord query (after require) allocates more memory than subsequent queries. This does not happen in Sequel.

![First Run Memory]({{ "/assets/images/init-run-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![First Run Memory]({{ "/assets/images/init-run-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
### 2. Creating records
Measured two aspects in creating a record - one without validations and the other with a presence validation.
- There is little performance degradation due to the validation but this is a simple validation.

Sequel performs better than ActiveRecord in both speed and memory for creating records.
- It is 1.8x-2x faster for PostgreSQL and 1.2x faster for SQLite3.
- It's memory allocation is lower and memory retention is significantly lower than ActiveRecord (see charts).

<br>
**ORM Speed Performance**

Using PostgreSQL:

![Load Time]({{ "/assets/images/create-pg-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/create-val-pg-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
Using SQLite3:

![Load Time]({{ "/assets/images/create-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/create-val-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
**ORM Memory Usage**

Using PostgreSQL:

![Load Memory]({{ "/assets/images/create-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Memory]({{ "/assets/images/create-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Memory]({{ "/assets/images/create-val-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Memory]({{ "/assets/images/create-val-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }


<br>
### 3. Updating records
Measured two different methods of updating a record - one with a hash argument to update an attribute, and the other used method assignment then save to update the record.
- There is little performance difference between the two update methodologies.

Sequel performs better than ActiveRecord in both speed and memory for updating records.
- It is 1.7x-2x faster.
- It's memory allocation is less than half of ActiveRecord and memory retention is 6x-8x lower than ActiveRecord.

<br>
**ORM Speed Performance**

Using PostgreSQL:

![Load Time]({{ "/assets/images/update-pg-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/update2-pg-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
Using SQLite3:

![Load Time]({{ "/assets/images/update-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/update2-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
**ORM Memory Usage**

Using PostgreSQL:

![Load Memory]({{ "/assets/images/update-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Memory]({{ "/assets/images/update-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Memory]({{ "/assets/images/update2-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Memory]({{ "/assets/images/update2-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
### 4. Querying records
Querying records revealed some implementation differences in the ORM's and added some complexity to the benchmarking. For an apples-to-apples comparison, these benchmarks force the ORM's to typecast the data (if you missed the reason, see the updated note above). Measured the following methods of querying the database (AR method / Sequel method):
- `#find(id) / #[id]`
- `#findby() / #find()`
- `#where()`
- `#first`
- Eager load associated model, `#where().includes() / #where().eager()` (retrieves 1 + 100 records)
- `#where().order()` query (retrieves 50 records)


Sequel generally performed better than ActiveRecord in both speed and memory except for one query.
- It is 1.1x-2x faster without `sequel_pg` except for one query where it is 20% slower.
- It's memory allocation and retention is less than ActiveRecord in all but one query.
- Sequel with `sequel_pg` is 1.6x-3.8x faster than ActiveRecord.
- Sequel with `sequel_pg` is significantly lower in memory allocation and has similar, if not same, memory retention as sequel.

<br>
**ORM Speed Performance**

Using PostgreSQL:

![Load Time]({{ "/assets/images/find-pg-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/findby-pg-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/where-pg-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/first-pg-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/eager-pg-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/postwo-pg-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
SQLite3 Results shown in next section (#5).

<br>
**ORM Memory Usage**

Using PostgreSQL:

![Load Time]({{ "/assets/images/find-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/find-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/findby-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/findby-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/where-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/where-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/first-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/first-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/eager-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/eager-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/postwo-pg-mem.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/postwo-pg-obj.png" | absolute_url }}){: .post-halfwidth-image }

<br>
### 5. Queries - Lazy typecasting & Timestamps
These benchmarks explore:
- ActiveRecord's defered typecasting gain/impact on performance (measured the same queries as above - only SQLite3 ones are shown here).
- Typecasting timestamps are slow and is the cause of Sequel slower performance in the one query above.

Lazy typecasting gain/impact:
- Sequel has the same performance whether you force typecasting through attribute access or not.
- ActiveRecord shows a 10-50% reduction in speed due to deferred typecasting when you access the attribute data.
- It is interesting that even with this optimization, ActiveRecord still looses to Sequel in some queries.

Retrieving timestamps impact:
- The `#where().order()` query performance without retrieving timestamp data increases 1.9x for ActiveRecord and 5.8x for Sequel.
- The `#where().order()` query performance of retrieving the timestamp data only (nothing else) is close to retrieving the full record data for both ORM's.
- This shows that the low performance of `#where().order()` query is correlated to retrieving timestamp data for the 50 records.

<br>
**ORM query performance**

Using SQLite3:

![Load Time]({{ "/assets/images/find-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/findby-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/where-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/first-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
![Load Time]({{ "/assets/images/eager-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left  }
![Load Time]({{ "/assets/images/postwo-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }


<br>
**Timestamps performance cost in `#where().order()` query**

<br>
![Load Time]({{ "/assets/images/postwo-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image .post-halfwidth-image-left }
![Load Time]({{ "/assets/images/postwo2-sq3-ips.png" | absolute_url }}){: .post-halfwidth-image }

<br>
## Conclusion:
Summarizing the benchmark results:
- In loading, Sequel beats ActiveRecord by ~3x in speed and memory.
- In creating new records, Sequel performs 1.2x-2x better than ActiveRecord in speed and uses less memory.
- In updating records, Sequel performs 1.7x-2x better than ActiveRecord in speed and uses less than half the memory.
- In querying:
  - Sequel without `sequel_pg` is 1.1x-2x faster than ActiveRecord except for one query where it is 20% slower.
  - Sequel memory allocation and retention is less than ActiveRecord in all but one query.
  - Sequel with `sequel_pg` is 1.6x-3.8x faster than ActiveRecord.
  - Sequel with `sequel_pg` is significantly lower in memory allocation and has similar, if not same, memory retention as sequel (which is lower than ActiveRecord).
  - Retrieving timestamp data can impact speed performance significantly for both ORM's.

In conclusion, the overall performance of Sequel is much better than ActiveRecord both in speed and memory usage by a good margin. The lower memory usage also helps to reduce garbage collection overhead in an application. I liked the fact Sequel does not defer some object allocation when loading the library, and it immediately typecasts the data on retrieval when querying. This means once the query is executed, the data is ready for consumption. A side note on querying - typecasting timestamps are slow and if you don't need it, then don't retrieve it from the database in performance hot spots. Lastly, I was quite impressed with the significant performance boost of Sequel with the `sequel_pg` addon used with PostgreSQL databases - it's definitely a must use addon, especially when the only thing you have to do is add it to the `Gemfile`!

