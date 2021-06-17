# wrk - a HTTP benchmarking tool

  wrk is a modern HTTP benchmarking tool capable of generating significant
  load when run on a single multi-core CPU. It combines a multithreaded
  design with scalable event notification systems such as epoll and kqueue.

  An optional LuaJIT script can perform HTTP request generation, response
  processing, and custom reporting. Details are available in SCRIPTING and
  several examples are located in [scripts/](scripts/).

## Basic Usage

    wrk -t12 -c400 -d30s http://127.0.0.1:8080/index.html

  This runs a benchmark for 30 seconds, using 12 threads, and keeping
  400 HTTP connections open.

  Output:

    Running 30s test @ http://127.0.0.1:8080/index.html
      12 threads and 400 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   635.91us    0.89ms  12.92ms   93.69%
        Req/Sec    56.20k     8.07k   62.00k    86.54%
      22464657 requests in 30.00s, 17.76GB read
    Requests/sec: 748868.53
    Transfer/sec:    606.33MB

## Options on top of standard wrk
  1. Number of connections need not be a multiple of number of threads.
  2. Ability to send <n> reuqests per connection (see next section for usage)

     3 connections, 1 request per connection, quit when done:

        bhakta@bhakta-ubuntu:~/dev/github/wrk$ time ./wrk -t2 -c3 http://192.168.1.12:9090/bhakta -r1 -q
        Running 10s test @ http://192.168.1.12:9090/bhakta
          2 threads and 3 connections
          Thread Stats   Avg      Stdev     Max   +/- Stdev
            Latency   830.33us  221.91us   1.07ms   66.67%
            Req/Sec    14.50      6.36    19.00    100.00%
          3 requests in 1.00s, 807.00B read
        Requests/sec:      2.99
        Transfer/sec:     804.79B

        real    0m1.019s
        user    0m0.005s
        sys     0m0.004s


     3 connections, 1 request per connection (quit per duration)

        bhakta@bhakta-ubuntu:~/dev/github/wrk$ time ./wrk -t2 -c3 http://192.168.1.12:9090/bhakta -r1 -d10s
        Running 10s test @ http://192.168.1.12:9090/bhakta
          2 threads and 3 connections
          Thread Stats   Avg      Stdev     Max   +/- Stdev
            Latency     0.87ms   73.37us   0.95ms   66.67%
            Req/Sec    15.00      7.07    20.00    100.00%
          3 requests in 10.01s, 807.00B read
        Requests/sec:      0.30
        Transfer/sec:      80.59B

        real    0m10.018s
        user    0m0.003s
        sys     0m0.007s

     4 connections, 10 request per connection, quit when done

        bhakta@bhakta-ubuntu:~/dev/github/wrk$ time ./wrk  http://192.168.1.12:9090/bhakta -t2 -c4 -r10 -q
        Running 10s test @ http://192.168.1.12:9090/bhakta
          2 threads and 4 connections
          Thread Stats   Avg      Stdev     Max   +/- Stdev
            Latency     2.16ms    1.38ms   5.96ms   85.00%
            Req/Sec   200.00      2.83   202.00    100.00%
          40 requests in 1.00s, 10.51KB read
        Requests/sec:     39.94
        Transfer/sec:     10.49KB

        real    0m1.010s
        user    0m0.003s
        sys     0m0.011s

     7 connections, 10 request per connection, quit when done

        bhakta@bhakta-ubuntu:~/dev/github/wrk$ time ./wrk  http://192.168.1.12:9090/bhakta -t2 -c7 -r10 -q
        Running 10s test @ http://192.168.1.12:9090/bhakta
          2 threads and 7 connections
          Thread Stats   Avg      Stdev     Max   +/- Stdev
            Latency     3.07ms    1.35ms   6.97ms   78.57%
            Req/Sec   350.00     70.71   400.00    100.00%
          70 requests in 1.00s, 18.39KB read
        Requests/sec:     69.89
        Transfer/sec:     18.36KB

        real    0m1.017s
        user    0m0.005s
        sys     0m0.007s

## Command Line Options

    -c, --connections: total number of HTTP connections to keep open with
                       each thread handling N = connections/threads

    -d, --duration:    duration of the test, e.g. 2s, 2m, 2h

    -t, --threads:     total number of threads to use

    -s, --script:      LuaJIT script, see SCRIPTING

    -H, --header:      HTTP header to add to request, e.g. "User-Agent: wrk"

        --latency:     print detailed latency statistics

        --timeout:     record a timeout if a response is not received within
                       this amount of time.

## Options added on top of standard wrk (wg/wrk)
    -r --requests       Number of requests to be sent per connection
    -q --quit-when-done Quit after sending <r> requests. Valid only when <r> is set

## Benchmarking Tips

  The machine running wrk must have a sufficient number of ephemeral ports
  available and closed sockets should be recycled quickly. To handle the
  initial connection burst the server's listen(2) backlog should be greater
  than the number of concurrent connections being tested.

  A user script that only changes the HTTP method, path, adds headers or
  a body, will have no performance impact. Per-request actions, particularly
  building a new HTTP request, and use of response() will necessarily reduce
  the amount of load that can be generated.

## Acknowledgements

  wrk contains code from a number of open source projects including the
  'ae' event loop from redis, the nginx/joyent/node.js 'http-parser',
  and Mike Pall's LuaJIT. Please consult the NOTICE file for licensing
  details.

## Cryptography Notice

  This distribution includes cryptographic software. The country in
  which you currently reside may have restrictions on the import,
  possession, use, and/or re-export to another country, of encryption
  software. BEFORE using any encryption software, please check your
  country's laws, regulations and policies concerning the import,
  possession, or use, and re-export of encryption software, to see if
  this is permitted. See <http://www.wassenaar.org/> for more
  information.

  The U.S. Government Department of Commerce, Bureau of Industry and
  Security (BIS), has classified this software as Export Commodity
  Control Number (ECCN) 5D002.C.1, which includes information security
  software using or performing cryptographic functions with symmetric
  algorithms. The form and manner of this distribution makes it
  eligible for export under the License Exception ENC Technology
  Software Unrestricted (TSU) exception (see the BIS Export
  Administration Regulations, Section 740.13) for both object code and
  source code.
