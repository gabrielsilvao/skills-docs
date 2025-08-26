# **Test Project Gameday Simulation**

## Task

In this Test Project, you'll need to **host binaries** containing a simple application. The hosting must meet the pillars of AWS's Well Architect Framework, particularly Operational Excellence, Performance Efficiency, and Reliability.

Consider the cost, deployment speed of each new binary, content delivery latency, and service scalability. Every minute, millions of users are trying to access your website through the endpoint.

For the start of the project, **only one binary will be available**, and during the course of the 2 hours, the other binaries will be made available and **you will need to update the hosted application** and submit the new endpoint of the new version of the application.

## Binaries

[Binary 1](bin/server1)<br />
[Binary 2](bin/server2)<br />
[Binary 3](bin/server3)<br />

## Validation with k6

To validate the first binary, run the following code with `k6 run validate_server1.js`:
```js linenums="1" title="validate_server1.js"
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },
    { duration: '1m', target: 100 },
    { duration: '30s', target: 200 },
    { duration: '1m', target: 200 },
    { duration: '30s', target: 0 },
  ],
  thresholds: {
    http_req_failed: ['rate<0.01'],
    http_req_duration: ['p(95)<500'],
  },
};

function cpuIntensiveOperation() {
  const endpoints = [
    '/calculate?n=1000',
    '/search?q=stress',
    '/render',
  ];
  return endpoints[Math.floor(Math.random() * endpoints.length)];
}

export default function () {
  const url = `http://alb-teste-1609956826.us-east-1.elb.amazonaws.com${cpuIntensiveOperation()}`;
  
  if (Math.random() > 0.5) {
    http.get(url);
  } else {
    http.post(url, JSON.stringify({ data: 'payload-' + Math.random() }), {
      headers: { 'Content-Type': 'application/json' },
    });
  }

  sleep(Math.random() * 2);
}
```

To validate the second binary, run the following code with `k6 run validate_server2.js`:
```js linenums="1" title="validate_server2.js"
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend, Rate } from 'k6/metrics';

const responseTime = new Trend('response_time');
const errorRate = new Rate('error_rate');

export const options = {
  stages: [
    { duration: '1m', target: 50 },
    { duration: '3m', target: 100 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    error_rate: ['rate<0.05'],
  },
  noConnectionReuse: true,
  userAgent: 'k6-load-test/1.0',
  insecureSkipTLSVerify: true,
  tlsVersion: {
    min: 'tls1.2',
    max: 'tls1.3'
  }
};

export default function () {
  const baseUrl = 'https://www.cloudjuliolab.click';
  const endpoints = [
    '/search?q=stress',
    '/calculate?n=25',
    '/status'
  ];

  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json'
    },
    timeout: '60s',
    tags: {
      test_type: 'load_test'
    }
  };

  try {
    const url = `${baseUrl}${endpoints[Math.floor(Math.random() * endpoints.length)]}`;
    
    const res = http.get(url, params);
    
    responseTime.add(res.timings.duration);
    errorRate.add(res.status !== 200);

    check(res, {
      'status is 200': (r) => r.status === 200,
      'response time OK': (r) => r.timings.duration < 1000,
    });

  } catch (error) {
    errorRate.add(1);
    console.error(`Request failed: ${error}`);
  }

  sleep(Math.random() * 2);
}
```

To validate the third binary, run the following code with `k6 run validate_server3.js`:
```js linenums="1" title="validate_server3.js"
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend, Rate } from 'k6/metrics';

export const options = {
  vus: 10,
  duration: '3m',
  insecureSkipTLSVerify: true,
  thresholds: {
    http_req_failed: ['rate<0.05'],
    http_req_duration: ['p(95)<1500']
  }
};

const metrics = {
  matrix: {
    responseTime: new Trend('matrix_time'),
    errorRate: new Rate('matrix_errors')
  },
  monteCarlo: {
    responseTime: new Trend('montecarlo_time'),
    errorRate: new Rate('montecarlo_errors')
  },
  primes: {
    responseTime: new Trend('primes_time'),
    errorRate: new Rate('primes_errors')
  },
  alloc: {
    responseTime: new Trend('alloc_time'),
    errorRate: new Rate('alloc_errors')
  }
};

const BASE_URL = 'https://www.cloudjuliolab.click/';

function testMatrix() {
  const size = Math.floor(Math.random() * 400) + 100;
  const params = {
    timeout: '30s',
    tags: { endpoint: 'matrix' }
  };
  
  const res = http.get(`${BASE_URL}/matrix?size=${size}`, params);
  
  metrics.matrix.responseTime.add(res.timings.duration);
  metrics.matrix.errorRate.add(res.status !== 200);
  
  check(res, {
    'matrix status 200': (r) => r.status === 200,
    'matrix response <2s': (r) => r.timings.duration < 2000
  });
}

function testMonteCarlo() {
  const iterations = Math.floor(Math.random() * 9e6) + 1e6;
  const params = {
    timeout: '45s',
    tags: { endpoint: 'montecarlo' }
  };
  
  const res = http.get(`${BASE_URL}/monte-carlo?iterations=${iterations}`, params);
  
  metrics.monteCarlo.responseTime.add(res.timings.duration);
  metrics.monteCarlo.errorRate.add(res.status !== 200);
  
  check(res, {
    'montecarlo status 200': (r) => r.status === 200,
    'montecarlo response <5s': (r) => r.timings.duration < 5000
  });
}

function testPrimes() {
  const limit = Math.floor(Math.random() * 9e5) + 1e5;
  const params = {
    timeout: '30s',
    tags: { endpoint: 'primes' }
  };
  
  const res = http.get(`${BASE_URL}/primes?limit=${limit}`, params);
  
  metrics.primes.responseTime.add(res.timings.duration);
  metrics.primes.errorRate.add(res.status !== 200);
  
  check(res, {
    'primes status 200': (r) => r.status === 200,
    'primes response <3s': (r) => r.timings.duration < 3000
  });
}

function testAlloc() {
  const mb = Math.floor(Math.random() * 90) + 10;
  const params = {
    timeout: '30s',
    tags: { endpoint: 'alloc' }
  };
  
  const res = http.get(`${BASE_URL}/alloc?mb=${mb}`, params);
  
  metrics.alloc.responseTime.add(res.timings.duration);
  metrics.alloc.errorRate.add(res.status !== 200);
  
  check(res, {
    'alloc status 200': (r) => r.status === 200,
    'alloc response <2s': (r) => r.timings.duration < 2000
  });
}

export default function () {
  const rand = Math.random();
  if (rand < 0.25) {
    testMatrix();
  } else if (rand < 0.5) {
    testMonteCarlo();
  } else if (rand < 0.75) {
    testPrimes();
  } else {
    testAlloc();
  }
  
  sleep(Math.random() * 2);
}
```