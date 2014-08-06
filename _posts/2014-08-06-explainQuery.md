---
title: Understanding Queries
author: jim
layout: tutorial
---

{% include tutorial-header.html %}

<!-- add some style to fix the xml formatting color -->
<style>
code.xml { color:#93a1a1 }
</style>


### Explaining Queries

We have recently added the ability to ask GeoMesa to explain how it intends to satisfy your query.  This feature is useful for understanding how a query is executed and also (on occasion) to diagnose errors or curious behavior.  GeoMesa maintains a number of indexes; for any given query, the query planner has to decide what query information should be used and how.  

#### Example: Geometric Query 
To see how this works, let's jump into an example.  Suppose we want to query for all the points in the polygon represented by the !["WKT"](http://en.wikipedia.org/wiki/Well-known_text) "POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)))".  We can run the explain function from the command line like so.

{% highlight scala %}
explain --catalog geomesa_catalog --typeName twittersmall --filter "INTERSECTS(geom, POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)))"
{% endhighlight %}

Output:

{% highlight scala %}
Originalfilter is [ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]
Filter is rewritten as [ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]
{% endhighlight %}

As an early step, the planner rewrites complex queries into a more general form in order that further processing can be optimized.  Our first example does not feature any rewriting, and most simple queries should not require any rewriting.

{% highlight scala %}
Scanning ST index table for feature type twittersmall
The geom filters are List([ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]).
The temporal filters are List().
{% endhighlight %}

In the next few lines of the sample output, GeoMesa will be using the ST (spatio-temporal) index and that we have a single topological predicate (and no temporal filters).

{% highlight scala %}
Planning query
Random Partition Planner Vector(00, 01, 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21)
GeoHashKeyPlanner: List(..., s.., su., suv, sv., svj)
DatePlanner: start: 00000101 end: 99991231
{% endhighlight %}

For our default geo-time indexing schema, we need information about the new of random partitions, a list of "dotted" geohash prefixes and date ranges to cover.  On our cloud, we have modified the default number of random partitions since we had 21 tablet servers.  In terms of geometry, we observe that the 15-bit geohashes 'suv' and 'svj' do cover the query polygon.  Since there is no temporal restrictions, we query for all time.  (Time for us ranges from the year 0000 to year 9999.)


{% highlight scala %}
Settings 577 col fams: List(.., 0., 0h, 0j, 0k, 0m, 0n, 0p, 0q, 0r, 0s, 0t, 0u, 0v, 0w, 0x, 0y, 0z, 1., 1h, 1j, 1k, 1m, 1n, 1p, 1q, 1r, 1s, 1t, 1u, 1v, 1w, 1x, 1y, 1z, 2., 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 2b, 2c, 2d, 2e, 2f, 2g, 2h, 2j, 2k, 2m, 2n, 2p, 2q, 2r, 2s, 2t, 2u, 2v, 2w, 2x, 2y, 2z, 3., 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 3b, 3c, 3d, 3e, 3f, 3g, 3h, 3j, 3k, 3m, 3n, 3p, 3q, 3r, 3s, 3t, 3u, 3v, 3w, 3x, 3y, 3z, 4., 4h, 4j, 4k, 4m, 4n, 4p, 4q, 4r, 4s, 4t, 4u, 4v, 4w, 4x, 4y, 4z, 5., 5h, 5j, 5k, 5m, 5n, 5p, 5q, 5r, 5s, 5t, 5u, 5v, 5w, 5x, 5y, 5z, 6., 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 6b, 6c, 6d, 6e, 6f, 6g, 6h, 6j, 6k, 6m, 6n, 6p, 6q, 6r, 6s, 6t, 6u, 6v, 6w, 6x, 6y, 6z, 7., 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 7b, 7c, 7d, 7e, 7f, 7g, 7h, 7j, 7k, 7m, 7n, 7p, 7q, 7r, 7s, 7t, 7u, 7v, 7w, 7x, 7y, 7z, 8., 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 8b, 8c, 8d, 8e, 8f, 8g, 8h, 8j, 8k, 8m, 8n, 8p, 8q, 8r, 8s, 8t, 8u, 8v, 8w, 8x, 8y, 8z, 9., 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 9b, 9c, 9d, 9e, 9f, 9g, 9h, 9j, 9k, 9m, 9n, 9p, 9q, 9r, 9s, 9t, 9u, 9v, 9w, 9x, 9y, 9z, b., b0, b1, b2, b3, b4, b5, b6, b7, b8, b9, bb, bc, bd, be, bf, bg, c., c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, cb, cc, cd, ce, cf, cg, d., d0, d1, d2, d3, d4, d5, d6, d7, d8, d9, db, dc, dd, de, df, dg, dh, dj, dk, dm, dn, dp, dq, dr, ds, dt, du, dv, dw, dx, dy, dz, e., e0, e1, e2, e3, e4, e5, e6, e7, e8, e9, eb, ec, ed, ee, ef, eg, eh, ej, ek, em, en, ep, eq, er, es, et, eu, ev, ew, ex, ey, ez, f., f0, f1, f2, f3, f4, f5, f6, f7, f8, f9, fb, fc, fd, fe, ff, fg, g., g0, g1, g2, g3, g4, g5, g6, g7, g8, g9, gb, gc, gd, ge, gf, gg, h., hh, hj, hk, hm, hn, hp, hq, hr, hs, ht, hu, hv, hw, hx, hy, hz, k., k0, k1, k2, k3, k4, k5, k6, k7, k8, k9, kb, kc, kd, ke, kf, kg, kh, kj, kk, km, kn, kp, kq, kr, ks, kt, ku, kv, kw, kx, ky, kz, p., pk, pm, pq, pr, ps, pt, pu, pv, pw, px, py, pz, r., r2, r3, r6, r7, r8, r9, rb, rc, rd, re, rf, rg, rk, rm, rq, rr, rs, rt, ru, rv, rw, rx, ry, rz, s., s0, s1, s2, s3, s4, s5, s6, s7, s8, s9, sb, sc, sd, se, sf, sg, sh, sj, sk, sm, sn, sp, sq, sr, ss, st, su, sv, sw, sx, sy, sz, u., u0, u1, u2, u3, u4, u5, u6, u7, u8, u9, ub, uc, ud, ue, uf, ug, x., x2, x3, x6, x7, x8, x9, xb, xc, xd, xe, xf, xg, xk, xm, xq, xr, xs, xt, xu, xv, xw, xx, xy, xz, z., z2, z3, z6, z7, z8, z9, zb, zc, zd, ze, zf, zg).
{% endhighlight %}

Next, we see some information about the column families which are set.  Since the query polygon is large relative to 15-bit geohashes, many column families are implicated.

{% highlight scala %}
Configuring batch scanner for ST table:
  Filter [ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]
  STII Filter: [[ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]]
  Interval:  No interval
  Filter: SpatialFilter(POLYGON ((41 28, 41 29, 42 29, 42 28, 41 28)))
  ECQL: None
Query: Query:
   feature type: twittersmall
   filter: [ geom intersects POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)) ]
   [properties:  ALL ].
addScanIterator(name:within-NtS{0, priority:200, class:geomesa.core.iterators.SpatioTemporalIntersectingIterator, properties:{geomesa.iterators.aggregator-types=tweet_id:Long:index=false,user_name:String:index=false,user_id:Long:index=false,text:String:index=false,dtg:Date:index=false,*geom:Point:srid=null:index=false, geomesa.iterators.aggregator-types.userdata.geomesa_index_start_time=dtg, geomesa.iterators.aggregator-types.userdata.geomesa_index_end_time=dtg, geomesa.index.filter=INTERSECTS(geom, POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28))), geomesa.index.schema=%~#s%21#r%twittersmall#cstr%0,3#gh%yyyyMMdd#d::%~#s%3,2#gh::%~#s%#id}
Query Planning took 398 milliseconds. 
{% endhighlight %}

The remainder of the output tells us more about how iterators are configured.  In particular, we can see that the SpatioTemporalIntersectingInterator is passed the filter string "INTERSECTS(geom, POLYGON ((41 28, 42 28, 42 29, 41 29, 41 28)))".  Finally, we can see how long GeoMesa took to plan the query.  

#### Example: Attribute and Geometry Query (on an attribute which is indexed) 
For our next example, let's consider querying for the filter "tweet_id = '123456' AND WITHIN(geom, POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)))".

{% highlight scala %}
Originalfilter is [[ tweet_id = 123456 ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
Filter is rewritten as [[ tweet_id = 123456 ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
Scanning attribute table for feature type twitter
{% endhighlight %}

Here we can see that we are going to use the attribute index since we have an attribute equality filter (which should be very fast to satisfy).

{% highlight scala %}
The geom filters are ArrayBuffer([ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]).
The temporal filters are ArrayBuffer().
{% endhighlight %}

Similar to above.

{% highlight scala %}
addScanIterator(name:attrIndexFilter, priority:10, class:geomesa.core.iterators.AttributeIndexFilteringIterator, properties:{geomesa.iterators.aggregator-types=tweet_id:Long:index=true,user_name:String:index=false,user_id:Long:index=false,text:String:index=false,dtg:Date:index=false,*geom:Point:srid=4326:index=false, geomesa.iterators.aggregator-types.userdata.geomesa_index_start_time=dtg, geomesa.iterators.aggregator-types.userdata.geomesa_index_end_time=dtg, geomesa.index.filter=WITHIN(geom, POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)))}
setRange: [tweet_id%00;123456 : [] 9223372036854775807 false,tweet_id%00;123456%00; : [] 9223372036854775807 false)
addScanIterator(name:sffilter->H42w, priority:300, class:geomesa.core.iterators.SimpleFeatureFilteringIterator, properties:{geomesa.iterators.aggregator-types=tweet_id:Long:index=true,user_name:String:index=false,user_id:Long:index=false,text:String:index=false,dtg:Date:index=false,*geom:Point:srid=4326:index=false, geomesa.iterators.aggregator-types.userdata.geomesa_index_start_time=dtg, geomesa.iterators.aggregator-types.userdata.geomesa_index_end_time=dtg, geomesa.feature.encoding=avro, geomesa.index.schema=%~#s%21#r%twitter#cstr%0,3#gh%yyyyMMdd#d::%~#s%3,2#gh::%~#s%#id}
Query Planning took 3 milliseconds.
{% endhighlight %}

Here we can notice that a range is being set for exactly our tweet's id.  As a quick contrast, let's look at the explanation for this query "user_name = 'Jim' AND WITHIN(geom, POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)))".

#### Example: Attribute and Geometry Query (on an attribute which is not indexed) 

{% highlight scala %}
Originalfilter is [[ user_name = Jim ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
Filter is rewritten as [[ user_name = Jim ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
Scanning ST index table for feature type twitter
{% endhighlight %}

Just like in the second example, we had an attribute equality, but we are not using a (presumably faster) attribute-index approach.  Why not?  For our particular twitter table, we decided to index on some attributes, but not others.  For this case, we have indexed 'tweet_id', but not 'user_name'.  


(Suppressed output similar to the first example.)

{% highlight scala %}
Configuring batch scanner for ST table: 
  Filter [[ user_name = Jim ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
  STII Filter: [[ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
  Interval:  No interval
  Filter: SpatialFilter(POLYGON ((45 23, 45 27, 48 27, 48 23, 45 23)))
  ECQL: Some(user_name = 'Jim')
Query: Query:
   feature type: twitter
   filter: [[ user_name = Jim ] AND [ geom within POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23)) ]]
   [properties:  ALL ].
addScanIterator(name:within-);4Gb, priority:200, class:geomesa.core.iterators.SpatioTemporalIntersectingIterator, properties:{geomesa.iterators.aggregator-types=tweet_id:Long:index=true,user_name:String:index=false,user_id:Long:index=false,text:String:index=false,dtg:Date:index=false,*geom:Point:srid=4326:index=false, geomesa.iterators.aggregator-types.userdata.geomesa_index_start_time=dtg, geomesa.iterators.aggregator-types.userdata.geomesa_index_end_time=dtg, geomesa.index.filter=WITHIN(geom, POLYGON ((45 23, 48 23, 48 27, 45 27, 45 23))), geomesa.index.schema=%~#s%21#r%twitter#cstr%0,3#gh%yyyyMMdd#d::%~#s%3,2#gh::%~#s%#id}
addScanIterator(name:sffilter-J{ke_, priority:300, class:geomesa.core.iterators.SimpleFeatureFilteringIterator, properties:{geomesa.iterators.aggregator-types=tweet_id:Long:index=true,user_name:String:index=false,user_id:Long:index=false,text:String:index=false,dtg:Date:index=false,*geom:Point:srid=4326:index=false, geomesa.iterators.ecql-filter=user_name = 'Jim', geomesa.iterators.aggregator-types.userdata.geomesa_index_start_time=dtg, geomesa.iterators.aggregator-types.userdata.geomesa_index_end_time=dtg, geomesa.feature.encoding=avro, geomesa.index.schema=%~#s%21#r%twitter#cstr%0,3#gh%yyyyMMdd#d::%~#s%3,2#gh::%~#s%#id}
Query Planning took 68 milliseconds.
{% endhighlight %}

Here we can see that SimpleFeatureFilteringIterator is being used to apply our attribute filter later (user_name = 'Jim').

As a recap, in our first example, we looked at how the query planner handled a simple geometric query.  In the next two examples, we were able to see the query planner choose to use an attribute index when it was available to run a more efficient query.  In the long-term, we plan to store aggregate data to inform more complex query planning decisions.  As that is built out, we will continue to extend the 'explainQuery' functionality.

