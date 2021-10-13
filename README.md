# testphp
Test php-fpm ping and phpinfo page performance on Graviton2 (c6g.xlarge) and x86-64 (c5g.xlarge)

# Test Tool - WRK
https://bipulkkuri.medium.com/install-wrk-on-centos-redhat-7d9291de15d7

# Test Architecture

## Docker Host
x86-64:
* Instance Type : c5.xlarge
* AMI ID : ami-091aa67fccd794d5f (ECS Optimized)

Graviton2:
* Instance Type: c6g.xlarge
* AMI ID: ami-009b187c8747c8482 (ECS Optimized)

## Docker Build/Run Command
*x86*
```
$ git clone https://github.com/lubao/testphp.git
$ cd testphp/x86
$ docker build -t testphp .
$ docker run -it -p 8080:80 --cpus=1 --memory=512m --entrypoint /bin/bash testphp:latest
```
*arm64v8*
```
$ git clone https://github.com/lubao/testphp.git
$ cd testphp/arm64v8
$ docker build -t testphp .
$ docker run -it -p 8080:80 --cpus=1 --memory=512m --entrypoint /bin/bash testphp:latest
```
# Result

## PHP-FPM Ping page
Commnad : /usr/local/bin/wrk -t1 -c5 -d60 --latency -v http://localhost:8080/ping
* command issued from docker host

### Summary:
* c5.xlarge  RPS: 7181.01
* c6g.xlarge RPS: 8713.24

### Detail:
* c5.xlarge
```
[ec2-user@ip-10-0-2-25 ~]$ /usr/local/bin/wrk -t1 -c5 -d60 --latency -v http://localhost:8080/ping
wrk 4.1.0-8-ga211dd5 [epoll] Copyright (C) 2012 Will Glozer
Running 1m test @ http://localhost:8080/ping  
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     9.49ms   15.46ms  56.82ms   81.49%
    Req/Sec     7.22k   344.05    12.23k    80.17%
  Latency Distribution
     50%  368.00us
     75%   15.08ms
     90%   37.86ms
     99%   51.77ms
  431111 requests in 1.00m, 132.37MB read
Requests/sec:   7181.01
Transfer/sec:      2.20MB
```

* c6g.xlarge
```
[ec2-user@ip-172-31-88-214 ~]$ /usr/local/bin/wrk -t1 -c5 -d60 --latency -v http://localhost:8080/ping
wrk 4.1.0-8-ga211dd5 [epoll] Copyright (C) 2012 Will Glozer
Running 1m test @ http://localhost:8080/ping
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     9.18ms   15.14ms  56.38ms   81.60%
    Req/Sec     8.76k   511.29    12.31k    69.50%
  Latency Distribution
     50%  313.00us
     75%   14.35ms
     90%   37.02ms
     99%   50.84ms
  522938 requests in 1.00m, 160.56MB read
Requests/sec:   8713.24
Transfer/sec:      2.68MB
```

## phpinfo() page
Commnad : /usr/local/bin/wrk -t1 -c5 -d60 --latency -v $URL
* command issued from docker host

### Summary:
* c5.xlarge  RPS: 1456.03
* c6g.xlarge RPS: 1655.57

### Detail:
* c5.xlarge
```
[ec2-user@ip-10-0-2-25 ~]$ /usr/local/bin/wrk -t1 -c5 -d60 --latency -v http://localhost:8080/
wrk 4.1.0-8-ga211dd5 [epoll] Copyright (C) 2012 Will Glozer
Running 1m test @ http://localhost:8080/
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.09ms   15.60ms  58.01ms   81.32%
    Req/Sec     1.46k    54.56     1.90k    76.37%
  Latency Distribution
     50%    1.83ms
     75%   17.50ms
     90%   39.63ms
     99%   53.52ms
  87508 requests in 1.00m, 4.87GB read
Requests/sec:   1456.03
Transfer/sec:     83.02MB
```

* c6g.xlarge
```
[ec2-user@ip-172-31-88-214 ~]$ /usr/local/bin/wrk -t1 -c5 -d60 --latency -v http://localhost:8080/
wrk 4.1.0-8-ga211dd5 [epoll] Copyright (C) 2012 Will Glozer
Running 1m test @ http://localhost:8080/
  1 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    10.69ms   15.39ms  57.77ms   81.43%
    Req/Sec     1.66k    83.99     2.38k    79.20%
  Latency Distribution
     50%    1.61ms
     75%   16.62ms
     90%   38.77ms
     99%   52.86ms
  99499 requests in 1.00m, 5.54GB read
Requests/sec:   1655.57
Transfer/sec:     94.41MB
```


