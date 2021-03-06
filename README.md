# nestjs-prom v1.x

A prometheus module for Nest.

__BREAKING CHANGE__

> nestjs-prom v0.2.x has been moved to [stable/0.2](https://github.com/digikare/nestjs-prom/tree/stable/0.2) branch.
>
> To migrate from v0.2 to v1.x please see [Migrate from 0.2.x to 1.x](./doc/migrate_0.2_1.x.md)

## Installation

```bash
$ npm install --save @digikare/nestjs-prom prom-client
```

## How to use

Import `PromModule` into the root `ApplicationModule`

```typescript
import { Module } from '@nestjs/common';
import { PromModule } from '@digikare/nestjs-prom';

@Module({
  imports: [
    PromModule.forRoot({
      defaultLabels: {
        app: 'my_app',
        version: 'x.y.z',
      }
    }),
  ]
})
export class ApplicationModule {}
```

### PromModule.forRoot options

Here the options available for PromModule.forRoot:

|key|type|default|details|
|---|---|---|---|
|defaultLabels|object|`{}`|The defaults labels to apply on every counter|
|useHttpCounterMiddleware|boolean|`true`|Create automatically http_requests_total counter|
|withDefaultController|boolean|`true`|Enable default controller to handle `/metrics` endpoint|
|withDefaultsMetrics|boolean|`true`|Enable defaults metrics for nodejs (call `collectDefaultMetrics()`)|
|withGlobalInterceptor|boolean|`true`|Enable interceptor to catch uncatched exception thrown by your app on an endpoint|


### Setup metric

To setup a metric, the module provide multiple ways to get a metric.

#### Using PromService and Dependency Injection

By using `PromService` service with DI to get a metric.

```typescript
@Injectable()
export class MyService {

  private readonly _counter: CounterMetric;
  private readonly _gauge: GaugeMetric,
  private readonly _histogram: HistogramMetric,
  private readonly _summary: SummaryMetric,

  constructor(
    private readonly promService: PromService,
  ) {
    this._counter = this.promService.getCounter({ name: 'my_counter' });
  }

  doSomething() {
    this._counter.inc(1);
  }

  reset() {
    this._counter.reset();
  }
}
```

#### Using Param Decorator

You have the following decorators:

- `@PromCounter()`
- `@PromGauge()`
- `@PromHistogram()`
- `@PromSummary()`

Below how to use it

```typescript
import { CounterMetric, PromCounter } from '@digikare/nest-prom';

@Controller()
export class AppController {

  @Get('/home')
  home(
    @PromCounter('app_counter_1_inc') counter1: CounterMetric,
    @PromCounter({ name: 'app_counter_2_inc', help: 'app_counter_2_help' }) counter2: CounterMetric,
  ) {
    counter1.inc(1);
    counter2.inc(2);
  }

  @Get('/home2')
  home2(
    @PromCounter({ name: 'app_counter_2_inc', help: 'app_counter_2_help' }) counter: CounterMetric,
  ) {
    counter.inc(2);
  }
}
```

### Metric class instances

If you want to counthow many instance of a specific class has been created:

```typescript
@PromInstanceCounter()
export class MyClass {

}
```

Will generate a counter called: `app_MyClass_instances_total`

### Metric method calls

If you want to increment a counter on each call of a specific method:

```typescript
@Injectable()
export class MyService {
  @PromMethodCounter()
  doMyStuff() {

  }
}
```

Will generate a counter called: `app_MyService_doMyStuff_calls_total` and auto increment on each call

You can use that to monitor an endpoint

```typescript
@Controller()
export class AppController {
  @Get()
  @PromMethodCounter() // will generate `app_AppController_root_calls_total` counter
  root() {
    // do your stuff
  }

  @Get('/login')
  @PromMethodCounter({ name: 'app_login_endpoint_counter' })  // set the counter name
  login() {
    // do your stuff
  }
}
```

### Metric endpoint

The default metrics endpoint is `/metrics` this can be changed with the customUrl option

```ts
@Module({
  imports: [
    PromModule.forRoot({
      defaultLabels: {
        app: 'my_app',
      },
      customUrl: 'custom/uri',
    }),
  ],
})
export class MyModule
```

Now your metrics can be found at `/custom/uri`.

> PS: If you have a global prefix, the path will be `{globalPrefix}/metrics` for
the moment.

## API

### PromModule.forRoot() options

- `withDefaultsMetrics: boolean (default true)` enable defaultMetrics provided by prom-client
- `withDefaultController: boolean (default true)` add internal controller to expose /metrics endpoints
- `useHttpCounterMiddleware: boolean (default false)` register http_requests_total counter

### Decorators

- `@PromInstanceCounter()` Class decorator, create and increment on each instance created
- `@PromMethodCounter()` Method decorator, create and increment each time the method is called
- `@PromCounter()` Param decorator, create/find counter metric
- `@PromGauge()` Param decorator, create/find gauge metric
- `@PromHistogram()` Param decorator, create/find histogram metric
- `@PromSummary()` Param decorator, create/find summary metric

## Auth/security

I do not provide any auth/security for `/metrics` endpoints.
This is not the aim of this module, but depending of the auth strategy, you can
apply a middleware on `/metrics` to secure it.

## TODO

- Update readme
  - Gauge
  - Histogram
  - Summary
- Manage registries
- Adding example on how to secure `/metrics` endpoint
  - secret
  - jwt

## License

MIT licensed
