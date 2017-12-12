---
layout: post
title:  ActiveRecord vs Sequel Performance
date:   2017-12-11
categories: software
tags: ruby gems performance
excerpt_separator: <!--more-->
---


I ran some benchmarking tests to evaluate the performance difference, if any, between ActiveRecord and Sequel ORM's - two common ORM's in the Ruby world. There were two sets of benchmarks measured, one was speed, and the other was memory consumed. In addition, each set was measured against two different databases, SQLite3 and PostgreSQL, to see what impact the database might have on the scores. I summarized the setup, results, and conclusion below.
<!--more-->

<br>
## Setup
1. All the tests were done on a local machine (2013 MacBook Pro) i.e. both the database and ruby script ran on the same machine for simplicity and time.
2. Versions used: Ruby v2.4.2, ActiveRecord v5.1.4, Sequel v5.2.0, SQLite v3.16.0, and PostgreSQL v9.6.5.
3. Used `benchmark-ips` and `memory_profiler` gems to obtain performance measurements.
4. The database schema had 3 models - topic, post, and comment. Topic has many posts and post has many comments. The database was seeded with 10 topics, 100 posts, and 20,000 comments.
5. Ran the test on SQLite3 and PostgreSQL. Both were setup with a local database file like a normal application.
6. Both ORM's had to write to a log file like they would in a real world application. Logging did affect the speed - made it slower.

<br>
## Conclusion:
Summarizing the benchmark results:
- For the most part, Sequel has lower memory overhead than ActiveRecord, both in require and transactions. For requiring the ORM - it was 6Mbytes vs 20Mbytes allocated!
- The initial use of any ActiveRecord query or insert has a larger memory allocation (~200Kb) but all subsequent calls are less (~10's Kb).
- For all but one query, Sequel performed better than ActiveRecord for speed.

Overall, I conclude Sequel outperformed ActiveRecord both in performance and memory by a good margin. It was not a 100% win across the board as there were a couple places where Sequel was on par or worse than ActiveRecord. I suspect there might be a performance regression in Sequel in the one query it underperformed ActiveRecord, especially given how well it performed in all the other queries. See the benchmark results below for more details.

<br>

## Benchmark Results:

Note:<br>
AR = ActiveRecord gem v5.1.4<br>
Sql = Sequel gem v5.2.0

<br>
### Measuring ORM with SQLite3 Database
**Performance Benchmark (iterations per second)**

<table>
  <tr>
    <th rowspan="2"></th>
    <th colspan="3">SQLite3 - Iterations per Second</th>
  </tr>
  <tr>
    <th class="col-width-reg">AR</th>
    <th class="col-width-reg">Sql</th>
    <th class="col-width-last">Sql to AR %</th>
  </tr>
  <tr>
    <td>Topic.first</td>
    <td>4,185.0</td>
    <td>6,154.0</td>
    <td>47.0%</td>
  </tr>
  <tr>
    <td>Topic.find(id)</td>
    <td>6,186.0</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Topic.find_by(id: id)<br><strong>Sql:</strong> Topic.find(id: id)</td>
    <td>4,762.0</td>
    <td>5,364.0</td>
    <td>12.6%</td>
  </tr>
  <tr>
    <td>Topic.where(id: id).first</td>
    <td>2,895.0</td>
    <td>4,355.0</td>
    <td>50.4%</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Topic.includes(:posts).where(id: id).to_a<br><strong>Sql:</strong> Topic.eager(:posts).where(id: id).all</td>
    <td>152.4</td>
    <td>156.0</td>
    <td>2.4%</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Post.where(topic: topic).where("updated_at &gt; ?", from).order(updated_at: :desc).to_a<br><strong>Sql:</strong> Post.where(topic: topic).where(Sequel.lit("updated_at &gt; ?", from)).order(:updated_at).reverse.all</td>
    <td>906.0</td>
    <td>310.6</td>
    <td>-65.7%</td>
  </tr>
  <tr>
    <td>topic.update(name: "Hello Bob”)<br>topic.update(name: "Hello Janet”)</td>
    <td>280.1</td>
    <td>499.8</td>
    <td>78.4%</td>
  </tr>
  <tr>
    <td>topic.name = "Hello Bob"<br>topic.save<br>topic.name = “Hello Janet”<br>topic.save</td>
    <td>292.8</td>
    <td>488.9</td>
    <td>67.0%</td>
  </tr>
  <tr>
    <td>Topic.create(name: "Hello Bob")</td>
    <td>632.5</td>
    <td>745.5</td>
    <td>17.9%</td>
  </tr>
  <tr>
    <td>Topic.create(name: "Hello Bob”) with name presence validation set to true</td>
    <td>637.2</td>
    <td>737.2</td>
    <td>15.7%</td>
  </tr>
</table>


**Memory Benchmarks (allocated / retained)**
<table>
  <tr>
    <th rowspan="2"></th>
    <th rowspan="2"></th>
    <th colspan="3">SQLite3 - Memory allocated/retained</th>
  </tr>
  <tr>
    <th class="col-width-reg">AR</th>
    <th class="col-width-reg">Sql</th>
    <th class="col-width-last">Sql to AR %</th>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> require “active_record”<br><strong>Sql:</strong> require “sequel”</td>
    <td>Mem</td>
    <td>19,556,500 / 1,729,382 bytes</td>
    <td>5,783,767 / 657,894 bytes</td>
    <td>-70.4% / -62.0%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>172,388 / 16,092 obj</td>
    <td>53,512 / 6,916 obj</td>
    <td>-69.0% / -57.0%</td>
  </tr>
  <tr>
    <td rowspan="2">Initial run (AR only)</td>
    <td>Mem</td>
    <td>196,006 / 13,357 bytes</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>2,054 / 163 obj</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.first</td>
    <td>Mem</td>
    <td>13,008 / 966 bytes</td>
    <td>18,906 / 7,615 bytes</td>
    <td>45.3% / 688.3%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>171 / 15 objects</td>
    <td>157 / 23 objects</td>
    <td>-8.2% / 53.3%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.find(id)</td>
    <td>Mem</td>
    <td>54,871 / 8,467 bytes</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>510 / 75 objects</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Topic.find_by(id: id)<br><strong>Sql:</strong> Topic.find(id: id)</td>
    <td>Mem</td>
    <td>15,276 / 1,446 bytes</td>
    <td>13,259 / 1,816 bytes</td>
    <td>-13.2% / 25.6%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>220 / 25 objects</td>
    <td>157 / 15 objects</td>
    <td>-28.6% / -40.0%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.where(id: id).first</td>
    <td>Mem</td>
    <td>17,573 / 1,942 bytes</td>
    <td>14,515 / 1,816 bytes</td>
    <td>-17.4% / -6.5%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>241 / 33 objects</td>
    <td>160 / 15 objects</td>
    <td>-33.6% / -54.5%</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Topic.includes(:posts).where(id: id).to_a <br><strong>Sql:</strong> Topic.eager(:posts).where(id: id).all</td>
    <td>Mem</td>
    <td>1,193,078 / 39,405 bytes</td>
    <td>715,990 / 5,480 bytes</td>
    <td>-40.0% / -86.1%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>9,502 / 444 objects</td>
    <td>6,784 / 42 objects</td>
    <td>-28.6% / -90.5%</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Post.where(topic: topic).where("updated_at &gt; ?", from).order(updated_at: :desc).to_a<br><strong>Sql:</strong> Post.where(topic: topic).where(Sequel.lit("updated_at &gt; ?", from)).order(:updated_at).reverse.all</td>
    <td>Mem</td>
    <td>94,686 / 47,996 bytes</td>
    <td>345,961 / 1,744 bytes</td>
    <td>265.4% / -96.4%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>913 / 279 objects</td>
    <td>3,329 / 16 objects</td>
    <td>264.6% / -94.3%</td>
  </tr>
  <tr>
    <td rowspan="2">topic.update(name: "Hello Bob”)<br>topic.update(name: "Hello Janet”)</td>
    <td>Mem</td>
    <td>79,063 / 8,155 bytes</td>
    <td>38,674 / 2,528 bytes</td>
    <td>-51.1% / -72.7%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>1,081 / 108 objects</td>
    <td>472 / 21 objects</td>
    <td>-56.3% / -80.6%</td>
  </tr>
  <tr>
    <td rowspan="2">topic.name = "Hello Bob"<br>topic.save<br>topic.name = “Hello Janet”<br>topic.save</td>
    <td>Mem</td>
    <td>67,504 / 4,750 bytes</td>
    <td>29,236 / 1,792 bytes</td>
    <td>-56.7% / -62.3%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>920 / 54 objects</td>
    <td>335 / 14 objects</td>
    <td>-63.6% / -74.1%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.create(name: "Hello Bob")</td>
    <td>Mem</td>
    <td>34,098 / 5,744 bytes</td>
    <td>24,371 / 2,216 bytes</td>
    <td>-28.5% / -61.4%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>450 / 65 objects</td>
    <td>289 / 16 objects</td>
    <td>-35.8% / -75.4%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.create(name: "Hello Bob”) with name presence validation set to true</td>
    <td>Mem</td>
    <td>33,618 / 5,256 bytes</td>
    <td>35,539 / 2,376 bytes</td>
    <td>6.7% / -54.8%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>441 / 54 objects</td>
    <td>458 / 20 objects</td>
    <td>3.9% / -63.0%</td>
  </tr>
</table>

<br>
### Measuring ORM with PostgreSQL Database
**Performance Benchmark (iterations per second)**

<table>
  <tr>
    <th rowspan="2"></th>
    <th colspan="3">PostgreSQL - Iterations per Second</th>
  </tr>
  <tr>
    <th class="col-width-reg">AR</th>
    <th class="col-width-reg">Sql</th>
    <th class="col-width-last">Sql to AR %</th>
  </tr>
  <tr>
    <td>Topic.first</td>
    <td>2,056.0</td>
    <td>3,491.0</td>
    <td>69.8%</td>
  </tr>
  <tr>
    <td>Topic.find(id)</td>
    <td>2,821.0</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Topic.find_by(id: id)<br><strong>Sql:</strong> Topic.find(id: id)</td>
    <td>2,505.0</td>
    <td>2,799.0</td>
    <td>11.7%</td>
  </tr>
  <tr>
    <td>Topic.where(id: id).first</td>
    <td>1,933.0</td>
    <td>2,392.0</td>
    <td>23.7%</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Topic.includes(:posts).where(id: id).to_a <br><strong>Sql:</strong> Topic.eager(:posts).where(id: id).all</td>
    <td>139.9</td>
    <td>145.2</td>
    <td>3.8%</td>
  </tr>
  <tr>
    <td><strong>AR:</strong> Post.where(topic: topic).where("updated_at &gt; ?", from).order(updated_at: :desc).to_a<br><strong>Sql:</strong> Post.where(topic: topic).where(Sequel.lit("updated_at &gt; ?", from)).order(:updated_at).reverse.all</td>
    <td>795.2</td>
    <td>298.4</td>
    <td>-62.5%</td>
  </tr>
  <tr>
    <td>topic.update(name: "Hello Bob”)<br>topic.update(name: "Hello Janet”)</td>
    <td>355.9</td>
    <td>752.2</td>
    <td>111.3%</td>
  </tr>
  <tr>
    <td>topic.name = "Hello Bob"<br>topic.save<br>topic.name = “Hello Janet”<br>topic.save</td>
    <td>369.4</td>
    <td>738.9</td>
    <td>100.0%</td>
  </tr>
  <tr>
    <td>Topic.create(name: "Hello Bob")</td>
    <td>730.9</td>
    <td>1,354.0</td>
    <td>85.2%</td>
  </tr>
  <tr>
    <td>Topic.create(name: "Hello Bob”) with name presence validation set to true</td>
    <td>764.9</td>
    <td>1,368.0</td>
    <td>78.9%</td>
  </tr>
</table>


**Memory Benchmarks (allocated / retained)**

<table>
  <tr>
    <th rowspan="2"></th>
    <th rowspan="2"></th>
    <th colspan="3">PostgreSQL - Memory allocated/retained</th>
  </tr>
  <tr>
    <th class="col-width-reg">AR</th>
    <th class="col-width-reg">Sql</th>
    <th class="col-width-last">Sql to AR %</th>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> require “active_record” <br><strong>Sql:</strong> require “sequel”</td>
    <td>Mem</td>
    <td>19,556,500 / 1,729,382 bytes</td>
    <td>5,783,767 / 657,894 bytes</td>
    <td>-70.4% / -62.0%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>172,388 / 16,092 obj</td>
    <td>53,512 / 6,916 obj</td>
    <td>-69.0% / -57.0%</td>
  </tr>
  <tr>
    <td rowspan="2">Initial run (AR only)</td>
    <td>Mem</td>
    <td>197,683 / 13,805 bytes</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>2,058 / 166 obj</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.first</td>
    <td>Mem</td>
    <td>14,344 / 1350 bytes</td>
    <td>17,626 / 7,615 bytes</td>
    <td>22.9% / 464.1%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>188 / 20 objects</td>
    <td>132 / 23 objects</td>
    <td>-29.8% / 15.0%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.find(id)</td>
    <td>Mem</td>
    <td>55,397 / 7,977 bytes</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>517 / 66 objects</td>
    <td>N/A</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Topic.find_by(id: id)<br><strong>Sql:</strong> Topic.find(id: id)</td>
    <td>Mem</td>
    <td>16,690 / 1,828 bytes</td>
    <td>11,979 / 1,816 bytes</td>
    <td>-28.2% / -0.7%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>239 / 30 objects</td>
    <td>132 / 15 objects</td>
    <td>-44.8% / -50.0%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.where(id: id).first</td>
    <td>Mem</td>
    <td>18,466 / 1,820 bytes</td>
    <td>13,235 / 1,816 bytes</td>
    <td>-28.3% / -0.2%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>251 / 27 objects</td>
    <td>135 / 15 objects</td>
    <td>-46.2% / -44.4%</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Topic.includes(:posts).where(id: id).to_a <br><strong>Sql:</strong> Topic.eager(:posts).where(id: id).all</td>
    <td>Mem</td>
    <td>1,181,716 / 39,293 bytes</td>
    <td>712,577 / 5,480 bytes</td>
    <td>-39.7% / -86.1%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>9,450 / 437 objects</td>
    <td>6,734 / 42 objects</td>
    <td>-28.7% / -90.4%</td>
  </tr>
  <tr>
    <td rowspan="2"><strong>AR:</strong> Post.where(topic: topic).where("updated_at &gt; ?", from).order(updated_at: :desc).to_a<br><strong>Sql:</strong> Post.where(topic: topic).where(Sequel.lit("updated_at &gt; ?", from)).order(:updated_at).reverse.all</td>
    <td>Mem</td>
    <td>90,464 / 42,846 bytes</td>
    <td>342,792 / 1,744 bytes</td>
    <td>278.9% / -95.9%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>922 / 279 objects</td>
    <td>3,306 / 16 objects</td>
    <td>258.6% / -94.2%</td>
  </tr>
  <tr>
    <td rowspan="2">topic.update(name: "Hello Bob”)<br>topic.update(name: "Hello Janet”)</td>
    <td>Mem</td>
    <td>74,911 / 8,347 bytes</td>
    <td>36,142 / 1,320 bytes</td>
    <td>-51.8% / -84.2%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>1,013 / 109 objects</td>
    <td>415 / 15 objects</td>
    <td>-59.0% / -86.2%</td>
  </tr>
  <tr>
    <td rowspan="2">topic.name = "Hello Bob"<br>topic.save<br>topic.name = “Hello Janet”<br>topic.save</td>
    <td>Mem</td>
    <td>63,196 / 4,748 bytes</td>
    <td>26,748 / 584 bytes</td>
    <td>-57.7% / -87.7%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>856 / 54 objects</td>
    <td>279 / 8 objects</td>
    <td>-67.4% / -85.1%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.create(name: "Hello Bob")</td>
    <td>Mem</td>
    <td>32,662 / 5,744 bytes</td>
    <td>19,844 / 968 bytes</td>
    <td>-39.2% / -83.1%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>424 / 65 objects</td>
    <td>211 / 9 objects</td>
    <td>-50.2% / -86.1%</td>
  </tr>
  <tr>
    <td rowspan="2">Topic.create(name: "Hello Bob”) with name presence validation set to true</td>
    <td>Mem</td>
    <td>32,302 / 5,256 bytes</td>
    <td>31,068 / 1,128 bytes</td>
    <td>-3.8% / -78.5%</td>
  </tr>
  <tr>
    <td>Obj</td>
    <td>418 / 54 objects</td>
    <td>381 / 13 objects</td>
    <td>-8.9% / -75.9%</td>
  </tr>
</table>


