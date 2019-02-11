# A barebones nodejs app

```haskell
mkdir lft-ngs && cd lft-ngs
npm init
git init
```

## Add `app.js`

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello world\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}`);
});
```

## Debugging

> Note: Do not enable debugging on production

### Enable Inspector

```haskell
node --inspect app.js
```

Node.js process listens for a debugging client on host and port `127.0.0.1:9229` and a UUID. A full url will look something like `ws://127.0.0.1:9229/0f2b7s-9d0v8-e9d0s-8sw3n-8s2n`

### Debugging tools

#### node-inspect

```haskell
node-inspect app.js
```

#### Chrome dev tools

Start up app in debug mode

```haskell
node inspect app.js
```

```haskell
Go to "chrome://inspect"
Click "Open dedicated DevTools for Node"

```

## Profiling

### Node.js built in profiler

It uses the profiler inside V8 which samples the stact at regualr intervals during program execution.
> **`Ticks`** : It records the results of the samples, along with important optimization events such as `jit` compiles, as a series of `Ticks`

```haskell
code-creation,LazyCompile,0,0x2d5000a337a0,396,"bp native array.js:1153:16",0x289f644df68,~
code-creation,LazyCompile,0,0x2d5000a33940,716,"hasOwnProperty native v8natives.js:198:30",0x289f64438d0,~
code-creation,LazyCompile,0,0x2d5000a33c20,284,"ToName native runtime.js:549:16",0x289f643bb28,~
code-creation,Stub,2,0x2d5000a33d40,182,"DoubleToIStub"
code-creation,Stub,2,0x2d5000a33e00,507,"NumberToStringStub"
```

#### Profiling application

##### Start profile

```haskell
NODE_ENV=[Production | Development] node --prof app.js
```

##### Put some load on the server

```haskell
curl -X GET "http://localhost:3000/[1-10000]"
```

##### Tick file

The `--prof` option generates a tick file `isolate-0xnnnnnnn-v8.log`

##### Tick Processor

Convert the tick file into more readable format

```Haskell
node -prof-process "isolate-0xnnnnnnn-v8.log" > processed.txt
```

##### Ticke processed file

```haskell
[Summary]:
   ticks  total  nonlib   name
     79    0.2%    0.2%  JavaScript
  36703   97.2%   99.2%  C++
      7    0.0%    0.0%  GC
    767    2.0%          Shared libraries
    215    0.6%          Unaccounted
```

```haskell
 [C++]:
   ticks  total  nonlib   name
  19557   51.8%   52.9%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
   4510   11.9%   12.2%  _sha1_block_data_order
   3165    8.4%    8.6%  _malloc_zone_malloc
```

```haskell
  19557   51.8%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
  19557  100.0%    v8::internal::Builtins::~Builtins()
  19557  100.0%      LazyCompile: ~pbkdf2 crypto.js:557:16

   4510   11.9%  _sha1_block_data_order
   4510  100.0%    LazyCompile: *pbkdf2 crypto.js:557:16
   4510  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30

   3165    8.4%  _malloc_zone_malloc
   3161   99.9%    LazyCompile: *pbkdf2 crypto.js:557:16
   3161  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30
```

In the above example most of the time is spent in C++ module `node::crypto::PBKDF2`. Modify your code and test it with same load profile as before, you will see similar output as follows.

```haskell
Concurrency Level:      20
Time taken for tests:   12.846 seconds
Complete requests:      250
Failed requests:        0
Keep-Alive requests:    250
Total transferred:      50250 bytes
HTML transferred:       500 bytes
Requests per second:    19.46 [#/sec] (mean)
Time per request:       1027.689 [ms] (mean)
Time per request:       51.384 [ms] (mean, across all concurrent requests)
Transfer rate:          3.82 [Kbytes/sec] received

...

Percentage of the requests served within a certain time (ms)
  50%   1018
  66%   1035
  75%   1041
  80%   1043
  90%   1049
  95%   1063
  98%   1070
  99%   1071
 100%   1079 (longest request)
```