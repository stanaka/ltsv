---
layout: default
title: FAQ of Labeled Tab-separated Values (LTSV)
og_url: http://ltsv.org/faq.html
twitter_description: FAQ of Labeled Tab-separated Values (LTSV) format
description: FAQ of Labeled Tab-separated Values (LTSV)
---
## FAQ

### What is LTSV?
It is a format of text, called "Labeled Tab-Separated Values". CSV, TSV, or JSON are the same as this. LTSV is useful for "logs", especially access logs of httpd.

Its spec is http://ltsv.org. Just updating now.

LTSV is neither more nor less than a format of logs.

### Is LTSV just values which are named and separated by tab?
Yes, it is.

<pre>
127.0.0.1 - frank [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html" "Mozilla/4.08 [en] (Win98; I ;Nav)"
</pre>

This log should be like:

<pre>
host:127.0.0.1&lt;TAB&gt;ident:-&lt;TAB&gt;user:frank&lt;TAB&gt;time:[10/Oct/2000:13:55:36 -0700]&lt;TAB&gt;req:GET /apache_pb.gif HTTP/1.0&lt;TAB&gt;status:200&lt;TAB&gt;size:2326&lt;TAB&gt;referer:http://www.example.com/start.html&lt;TAB&gt;ua:Mozilla/4.08 [en] (Win98; I ;Nav)
</pre>

### Why is just a format of logs talked a lot now?
Because it is a simple solution, like an egg of Columbus. Although the combined format used by access logs of apache is unexpectedly difficult to parse, and fragile against to add a new value, everyone use it by a historical reason. LTSV can solve these problems by a very small changing of format.

Recently the importance of log analysis is increasing, so you often add to logs a custom values not contained by a standard format, or you are sometimes forced to output logs which are frequently ordered to change its format. LTSV is suitable for these cases, too.

### What is the good points of LTSV?
* Easy to parse. For example, <pre>Hash[gets.split("\t").map{|f| f.split(":", 2)}]</pre> in ruby.
* Don't need any special parser.
* Don't need any special formatter to output logs. You can configure it with standard config files of apache or nginx.
* Open against adding a new field. In short, there is no influence on any existing programs when you add a new column.
* Easy to cook after parsing because values are named.
* Line-oriented, so it is easy to combine with other programs.

### What does it mean "open against expansion"?
Lets's suppose that you have following LTSV logs,

<pre>
host:127.0.0.1&lt;TAB&gt;ident:-&lt;TAB&gt;user:frank&lt;TAB&gt;
</pre>

and suppose you have hundreds scripts parsing and processing these logs, like

<pre>
#!/usr/bin/env ruby
while gets
  record = Hash[$_.split("\t").map{|f| f.split(":", 2)}]
  # do something to record
end
</pre>

Oneday, you find that there isn't a time field so you want to add it to logs.

<pre>
time:[10/Oct/2000:13:55:36 -0700]&lt;TAB&gt;host:127.0.0.1&lt;TAB&gt;ident:-&lt;TAB&gt;user:frank&lt;TAB&gt;
</pre>

Then, will these hundreds scripts break and become not working? No. If you use the combined format and parse logs with the regular expression, these scripts may not work.

You can also add the time field into the head, the tail, or anywhere. If the scripts are wrote considering that the record hash can be added a new value, they can handle the time field right after you add it to logs.

### What is the weak points of LTSV?
Comparing with the combined format:

* A bit worse to read by human

  * apart from whether the combined format is good for reading...
* Increase the size because of the amount of names of fields.

That's all. All weak points can be solved or you don't need to mind them. (see below)

### JSON is structured and looks better
From the point of view of naming the data, JSON or MessagePack are better, but parsing them is not easy. You need some ingenuity to output a special format from an existing software, like apache or nginx.

The selling point of LTSV is the balance of hassle free transition from the non-expandable format of logs.

### Doesn't it have to have "escaping" in its spec?
The spec of LTSV are just only not using ":" in the keys and separating by tab. There are some reasons why escaping is not in the spec.

* Strictly considering escaping makes parsing difficult.
* Aren't there cases when tabs appear in the strings like User-Agent? No, there aren't, because of vulnerability management.

In ltsv.org, there are many implementation of LTSV for various languages, but you don't have to use them. You can parse LTSV format with a simple process like this.

<pre>
#!/usr/bin/env ruby
while gets
  record = Hash[$_.split("\t").map{|f| f.split(":", 2)}]
  p record
end
</pre>

There is also a room of discussion of an external spec, like "strict-LTSV".

### Can I use it for something besides access logs?
Of course. LTSV is just a format like CSV, TSV, and JSON, so you can use it everywhere.

### Can I choose any name for labels as I please?
Yes, you can.

It is also nice to adopt the "Recommendations for labeling" on ltsv.org. You don't need to mind and it is convenient for doing anything if the names are the same.

### Output becomes hard to read.
You should use a kind of filter like ltsview. Implementation is very easy.

You can also tail logs formatted with the filter, like this.

<pre>
$ tail -f access_log | ltsview
</pre>

If you want to watch logs with the combined format, you can develop a filter converting from LTSV to combined.

The easiness of implementing a filter is a result of the spec of LTSV, based on the idea that is line-oriented, self-explanatory, and open for expansion.

### Is a bit larger size of logs all right?
It is my opinion without consensus, but

* It is not a big deal because the size of strings of requested URI, User-Agent, or referer is much larger than the size of labels.
* The large scale services in which they can't ignore a little increasing of the size of logs have often other solution for analyzing and storing the logs. 
  * Analyze logs with MapReduce (Hadoop or Amazon EMR), or store logs to DataWareHouse, etc.
  * The increase of the size by labels is offset by storing logs to database through fluentd or something.

At least Hatena uses LTSV for more than three years, so there is no problem in the same scale as Hatena.

### Is LTSV a spec for only fluentd?
No.

LTSV is just a format, not related to any other softwares. The reason why LTSV is talked together with fluentd is that fluentd is often used to process access logs. LTSV makes the fluentd configuration simple and DRY, so a troublesome problem (worse especially in long-term system) for administrators is solved by LTSV.

### I want to convert existing combined logs to LTSV.
There is a perl script for that.

You can use it only perl without any modules. It is a good point of LTSV that you can process logs without any special implementation.

### Who is the judge of the spec?
There is no rule like someone is the judge. Everyone who is interested in LTSV works as he or she likes. I also have no rights of LTSV.

 @stanaka has the domain of ltsv.org and he is the first originator, so something works with him. But the repository of ltsv.org is public and everyone can contribute it.

It is the internet!

### How to follow the movement?
It is better to search twitter with ltsv.

### Is this movement only in Japan?
Yes, or no. We want to spread it globally.

## Acknowledgement
This FAQ is originally written in [Japanese](http://d.hatena.ne.jp/naoya/20130209/1360381374)
