### Goader intro

Why another benchmarks utility?
All known to me utilities got flows:

- Too complex, UI or conffile based configuration to achieve wanted by everyone required effect  
- Simple CLI utilities do not simulate real life scenarios  

What I mean by real scenarios:  
I)  At some point latency of your services grows, 500ms per request is not fun. 
So you try to optimize things or even change backend. 
Problem is, that all benchmarks have `concurrent requests` settings, but you dont really have this information  
Information that you have - you got N requests per second and latency rised to X and X is not good.
So when optimizing things it's important to know, what you will get with specific number of evenly distributed requests per second, not concurrent requests

II) You got your service up, you truly beleive, that latency over 200ms is not usable. How many requests per second can it hold to maintain such latency? Yet again, `concurrent requests` does not answer this question

While both this scenarios is good enough reason for this benchmark utility to exist, they can be more complex, like 10 file PUTS, 100 GETS per second, or search for maximum concurrent threads with specific GET to PUT ration

### Features
- Extendable architecture with support of multiple request engines
- Simple url template, http://localhost/post/XXXXX. XXXXX will be replaced by incremental integers, (number of Xs>2)
- Supports different output formatters, atm human|readable

#####Request engines  
`--requests-engine=`  
- `null` Does nothing, useful for testing utility itself. At the moment reaches 700k requests per/sec  
- `sleep` Requster. For testing, sleeps instead of making real requests, also has semaphore of 10 concurrent connections, this emulates database which is bottleneck in the test and often in real life 
- `upload` Uploads (by PUT) and GET files (done, default engine).  
- `disk` Writes/Reads to/from disk. Support for O_DIRECT is planned  
- `http` Planned. Simple single variable method http tester. Planned to support url template or weighted urls files list   
- `s3` planned, to test self hosted object storage with s3 interface  

##### Output formatters
`--output=`  
- `human` Default, human readable output, also displays real time progress  
- `json` Outputs results as json for automatic use by other processes  

##### Other options  
- `-max-requests` Sets maximum requests count, defaults to 10000. (CTRL+C will abort and show stats)  
- `--body-size` Sets size for file sizes/post body sizes. 
Defaults to 160KiB, supports human formats, 1KiB for 1024 bytes, 1k/1kb for 1000 bytes  
- `-rps` Sets reads per seconds  
- `-wps` Sets writes per seconds  
- `-wt`  Sets concurrent write threads  
- `-rt`  Sets concurrent read threads  
- `-max-latency` Searches for maximum threads count with defined maximum latency, supports human format (1s, 300ms)
Combination of `--max-latency` and `-wt/-rt` will set initial threads count, useful if you know where to start, starting from 1 can take time to heat up  
- `-rpw`  Reads per writes in search for max throughput mode   
- `-url` Defines url/file path template. XXXXX in path will be replaced by incremental integers. If no pattern specified same path will be used for all requests   


   
### Examples  
`./goader -rps=30 -wps=30 --requests-engine=upload -url=http://127.0.0.1/post/XXXXX`  
Will maintain 30 writes, 30 reads per second, aborts if cannot maintain such rate, incrementaly increases url(http://127.0.0.1/post/00001, http://127.0.0.1/post/00002 ...)

`./goader --max-latency=300ms --requests-engine=upload -url="http://127.0.0.1/post/XXX"`
Will increase load gradually until reaching specified latency, useful for determining maximum throughput

`./goader --max-latency=300ms -rt=0 --requests-engine=upload -url="http://127.0.0.1/post/XXXXX"`
Same, but uploads only, no GETs

`./goader -rpt=10 --max-latency=300ms --requests-engine=upload -url="http://127.0.0.1/post/XXXXX"`
Will search for best PUT latency with read to write ratio 10, defaults to 1.
Maximum PUTs threads also limited by max-channels param, defaults to 500

`./goader -rt=200 -wt=30 --requests-engine=upload -url="http://127.0.0.1/post/XXXXX"`
Will maintain 200 read, 30 write threads

`./goader -rt=200 -wt=0 --requests-engine=upload -url="http://127.0.0.1/post/XXXXX"`
Same, reads only.

In all load patterns if both reads and writes issued reads will use previously written data, if only read pattern used - then incremental filenames (or from urls list file), with reads only it is expected for files/urls to be there

##### Example output
```
./goader --max-latency=500ms --max-requests=10000 -rpw=10 --requests-engine=disk --url='tmp/XXX' --body-size=4k

 Writes
Average response time: 537.25µs
Percentile 30 - 75.606µs
Percentile 40 - 80.12µs
Percentile 50 - 90.006µs
Percentile 60 - 107.548µs
Percentile 70 - 141.838µs
Percentile 80 - 216.89µs
Percentile 90 - 673.605µs
Percentile 95 - 2.238118ms
Threads with latency below 500ms: 106
Total requests: 1434
Total errors: 0
Average OP/s:  4222
Average good OP/s:  4222


 Reads
Average response time: 299.565µs
Percentile 30 - 15.884µs
Percentile 40 - 16.913µs
Percentile 50 - 18.327µs
Percentile 60 - 20.154µs
Percentile 70 - 23.618µs
Percentile 80 - 32.561µs
Percentile 90 - 79.102µs
Percentile 95 - 1.353748ms
Threads with latency below 500ms: 500
Total requests: 8567
Total errors: 0
Average OP/s:  25068
Average good OP/s:  25068
```





