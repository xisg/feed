---
title: That time Oracle tried to have a professor fired for benchmarking their database
url: https://danluu.com/anon-benchmark/
published: "2014-03-05T00:00:00Z"
feed: danluu
guid: https://danluu.com/anon-benchmark/
---

# That time Oracle tried to have a professor fired for benchmarking their database

In 1983, at the University of Wisconsin, Dina Bitton, David DeWitt, and Carolyn Turbyfill created a [database benchmarking framework](http://pages.cs.wisc.edu/~dewitt/includes/benchmarking/vldb83.pdf). Some of their results included (lower is better):

Join without indices

systemjoinAselBjoinABprimejoinCselAselBU-INGRES10.29.69.4C-INGRES1.82.62.1ORACLE\> 300\> 300\> 300IDMnodac\> 300\> 300\> 300IDMdac\> 300\> 300\> 300DIRECT10.29.55.6SQL/DS2.22.22.1

Join with indices, primary (clustered) index

systemjoinAselBjoinABprimejoinCselAselBU-INGRES2.111.669.07C-INGRES0.91.711.07ORACLE7.947.2213.78IDMnodac0.520.590.74IDMdac0.390.460.58DIRECT10.219.475.62SQL/DS0.921.081.33

Join with indicies, secondary (non-clustered) index

systemjoinAselBjoinABprimejoinCselAselBU-INGRES4.493.2410.55C-INGRES1.971.802.41ORACLE8.529.3918.85IDMnodac1.410.811.81IDMdac1.190.591.47DIRECT10.219.475.62SQL/DS1.621.42.66

Projection (duplicate tuples removed)

system100/100001000/10000U-INGRES64.6236.8C-INGRES26.4132.0ORACLE828.5199.8IDMnodac29.3122.2IDMdac22.368.1DIRECT2068.058.0SQL/DS28.828.0

Aggregate without indicies

systemMIN scalarMIN agg fn 100 partsSUM agg fun 100 partsU-INGRES40.2176.7174.2C-INGRES34.0495.0484.4ORACLE145.81449.21487.5IDMnodac32.065.067.5IDMdac21.238.238.2DIRECT41.0227.0229.5SQL/DS19.822.523.5

Aggregate with indicies

systemMIN scalarMIN agg fn 100 partsSUM agg fun 100 partsU-INGRES41.2186.5182.2C-INGRES37.2242.2254.0ORACLE160.51470.21446.5IDMnodac27.065.066.8IDMdac21.238.038.0DIRECT41.0227.0229.5SQL/DS8.522.823.8

Selection without indicies

system100/100001000/10000U-INGRES53.264.4C-INGRES38.453.9ORACLE194.2230.6IDMnodac31.733.4IDMdac21.623.6DIRECT43.046.0SQL/DS15.138.1

Selection with indicies

system100/10000 clustered100/10000 clustered100/100001000/10000U-INGRES7.727.859.278.9C-INGRES3.918.911.454.3ORACLE16.3130.017.3129.2IDMnodac2.09.93.827.6IDMdac1.58.73.323.7DIRECT43.046.043.046.0SQL/DS3.227.512.339.2

In case you're familiar with the database universe as of 1983, at the time, `INGRES` was a research project by Stonebreaker and Wong at Berkeley that had been commercialized. `C-INGRES` is the commercial versionn and `U-INGRES` is the university version. `IDM*` are the `IDM/500` database machine, the first widely used commercial database machine; `dac` is with a "database accelerator" and `nodac` is without. `DIRECT` was a research project in database machines that was started by DeWitt in 1977.

In Bitton et al.'s work, Oracle's performance stood out as unusually poor.

Larry Ellison wasn't happy with the results and it's said that he [tried to have](https://starcounter.com/the-story-of-professor-dewitt/) [DeWitt fired](http://www.citeulike.org/user/marclijour/article/6883245). Given how difficult it is to fire professors when there's actual misconduct, the probability of Ellison sucessfully getting someone fired for doing legitimate research in their field was pretty much zero. It's also said that, after DeWitt's non-firing, Larry banned Oracle from hiring Wisconsin grads and Oracle added a term to their EULA forbidding the publication of benchmarks. Over the years, many major commercial database vendors added a license clause that made benchmarking their database illegal.

Today, Oracle hires from Wisconsin, but Oracle still forbids benchmarking of their database. Oracle's shockingly poor performance and Larry Ellison's response have gone down in history; anti-benchmarking clauses are now often known as "DeWitt Clauses", and they've spread from databases to all software, from [compilers](https://news.ycombinator.com/item?id=7306121) to cloud offerings[1](#fn:1).

Meanwhile, Bitcoin users have created [anonymous markets for assassinations](http://rt.com/news/bitcoin-assassination-market-anarchist-983/) \-\- users can put money into a pot that gets paid out to the assassin who kills a particular target.

Anonymous assassination markets appear to be a joke, but how about anonymous markets for benchmarks? People who want to know what kind of performance a database offers under a certain workload puts money into a pot that gets paid out to whoever runs the benchmark.

With things as they are now, you often see comments and blog posts about how someone was using `postgres` until management made them switch to "some commercial database" which had much worse performance and it's hard to tell if the terrible database was Oracle, MS SQL server, or perhaps another database.

If we look at major commercial databases today, two out of the three big names in commericial databases forbid publishing benchmarks. Microsoft's SQL server eula says:

> You may not disclose the results of any benchmark test ... without Microsoft’s prior written approval

Oracle says:

> You may not disclose results of any Program benchmark tests without Oracle’s prior consent

IBM is notable for actually allowing benchmarks:

> Licensee may disclose the results of any benchmark test of the Program or its subcomponents to any third party provided that Licensee (A) publicly discloses the complete methodology used in the benchmark test (for example, hardware and software setup, installation procedure and configuration files), (B) performs Licensee's benchmark testing running the Program in its Specified Operating Environment using the latest applicable updates, patches and fixes available for the Program from IBM or third parties that provide IBM products ("Third Parties"), and (C) follows any and all performance tuning and "best practices" guidance available in the Program's documentation and on IBM's support web sites for the Program...

This gives people ammunition for a meta-argument that IBM probably delivers better performance than either Oracle or Microsoft, since they're the only company that's not scared of people publishing benchmark results, but it would be nice if we had actual numbers.

_Thanks to Leah Hanson and Nathan Wailes for comments/corrections/discussion._

* * *

1. There's at least one cloud service that disallows not only publishing benchmarks, but even "competitive benchmarking", running benchmarks to see how well the competition does. As a result, there's a product I'm told I shouldn't use to avoid even the appearance of impropriety because I work in an office with people who work on cloud related infrastructure.

An example of a clause like this is the following term in the Salesforce agreement:


> You may not access the Services for purposes of monitoring their availability, performance or functionality, or for any other benchmarking or competitive purposes.


If you ever wondered why uptime "benchmarking" services like cloudharmony don't include Salesforce, this is probably why. You will sometimes see speculation that Salesforce and other companies with these terms know that their service is so poor that it would be worse to have public benchmarks than to have it be known that they're afraid of public benchmarks.
    [\[return\]](#fnref:1)
