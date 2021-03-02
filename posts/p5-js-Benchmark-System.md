---
title: p5.js Benchmark System
date: 2017-11-05 18:39:18
image: /img/benchmark.jpg
tags:
  - posts
layout: layouts/post.njk
---

![alt text]({{ image | url }} "Code")

I was browsing the open [p5.js](https://github.com/processing/p5.js/issues/1524) issues when I stubbled across [this one](https://github.com/processing/p5.js/issues/1524). There was an open request to add performance benchmarking to the p5.js development tools, that would run benchmarks in multiple real browsers. The goal was to know when performance optimizations were really helping or adding unnecessary complexity.  That looked like a fun problem to work on.  Since performance testing is one thing I really love doing, I decided to take it on.  I learned A lot in the process.  This post explains the main parts of the of the system I put together,  using existing plugins.  It is copied from the [article on the p5.js wiki](https://github.com/processing/p5.js/wiki/Benchmarking-p5.js) .

# Benchmarking p5.js
We have a grunt task that runs performance benchmarks in multiple real browsers on the developers local machine. It will automatically detect which browsers are installed from the following list (Chrome, Firefox, Safari, Edge, IE) and run the benchmarks in all installed browsers and report the results.

Our benchmarking system consists of 3 main components:

1. [Benchmark.js](https://benchmarkjs.com/) - the engine that runs the benchmarks and the framework for defining benchmarks.
2. [karma-benchmark](https://www.npmjs.com/package/karma-benchmark) - The karma plugin for launching and running the benchmarks in multiple real browsers.
3. [grunt-karma](https://www.npmjs.com/package/grunt-karma) - The grunt plugin that allows us to define different karma configuations per grunt command.  This allows us to run single benchmarks or groups of benchmarks.

## Basic instructions:
### Install the new dependancies

```text
npm install
```

### Do a build, the benchmarks expect a local build of p5.js and p5.min.js

```text
grunt
```

### Run a specific benchmark

```text
grunt karma:<target_benchmark>
```

### To run all the benchmarks

```text
grunt karma
```

### Example

```text
grunt karma:random-dev
```

Outputs:
``` text
Chrome 62.0.3202 (Linux 0.0.0)
  p5 random() vs Math.random(): Math.random() at 95811115 ops/sec (1.26x faster than p5 random())
Firefox 56.0.0 (Fedora 0.0.0)
  p5 random() vs Math.random(): Math.random() at 2367566507 ops/sec (1.14x faster than p5 random())

Done, without errors.
```

## Adding new benchmarks
1.  Create a new benchmark file at this path and name format:  bench/*.bench.js
2.  Write the benchmark here is documentation [karma-benchmark](https://github.com/JamieMason/karma-benchmark/blob/master/README.md)
3.  Add the benchmark target to `grunt-karma.js`

### Example benchmark [random-fe-off.bench.js]

Here is an example benchmark that compares p5 random() to Math.random() with Friendly Error System disabled. It has two suites one for instanced mode and one using the global window random().

```js
p5.disableFriendlyErrors = true;

var p5Inst = new p5();

/**
 *  Instance random() vs Math.random()
 */
suite('Friendly Errors: OFF, Instance random() vs Math.random()', function () {
  benchmark('Instance random()', function () {
    return p5Inst.random();
  });

  benchmark('Math.random()', function () {
    return Math.random();
  });
});


/**
 *  Window random() vs Math.random()
 */
suite('Friendly Errors: OFF, Window random() vs Math.random()', function () {
  benchmark('window random()', function () {
    return random();
  });

  benchmark('Math.random()', function () {
    return Math.random();
  });
});
```
Then you would need to add your new benchmark to `grunt-karma.js`

```text
'random-fe-off-dev': {
    options: {
      files: [
        'lib/p5.js',
        'bench/math/random-fe-off.bench.js'
      ]
    }
  }
```
Now you can run your new benchmark with:

```text
grunt karma:random-fe-off-dev
```

## Comparing Prod to Dev
karma-benchmark can load remote files.  So it's easy to include the p5.js prod build and compare it to the dev build.  This is important when you're working on improving performance to compare your changes to what is in production.   To do this simply make two targets in `grunt-karma.js` one for prod and one for dev.

```text
  'random-prod': {
    options: {
      files: [
        'https://cdnjs.cloudflare.com/ajax/libs/p5.js/<%= pkg.version %>/p5.js',
        'bench/random.bench.js',
      ],
    },
  },
  'random-dev': {
    options: {
      files: [
        'lib/p5.js',
        'bench/random.bench.js',
      ],
    },
  },
```

You can see that `random-prod` actually loads the latest build from CDN.  Then to compare you can run both targets using:

```text
grunt karma:random-prod karma:random-dev
```

## Notes
I chose to put the grunt-karma tasks in it's own file `grunt-karma.js`  instead of the main `Gruntfile.js`, because as we add more benchmarks overtime the file could grow quite long, and I wanted to keep the main Gruntfile clean.

