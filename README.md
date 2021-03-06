# bitflyer4j 
[![Build Status][travis-icon]][travis-page] [![Coverage Status][coverall-icon]][coverall-page] [![Maven Central][maven-icon]][maven-page] [![Javadocs][javadoc-icon]][javadoc-page]

## Overview

**bitflyer4j** (bitFlyer for Java) is a Java wrapper library for the [bitFlyer Lightning](https://lightning.bitflyer.jp/?bf=yolu1tm3&lang=en) API.

[bitFlyer](https://bitflyer.jp?bf=yolu1tm3&lang=en) offers JSON+REST API just like many of the other bitcoin and altcoin exchanges. 
This library aims to capsulize the raw message formats and protocols, and provide a strictly-typed API library in addition to the handly features.

* Strictly-typed method calls, parameters and return types.
* Asynchronous method calls with `java.util.concurrent.CompletableFuture` for background message queueing/throttling and method chaining.
* Out-of-the-box listener callbacks for realtime data subscriptions.
* Private API tokens actually kept private.
* Battle-tested by the author.


## Getting Started

### Download

Use Maven or Gradle to import the library jars and its dependencies from the [Maven Central][maven-page].

#### Maven (`pom.xml`)
```xml
<dependency>
    <groupId>com.after_sunrise.cryptocurrency</groupId>
    <artifactId>bitflyer4j</artifactId>
    <version>${VERSION}</version>
</dependency>
```

### Sample Codes

Copy and paste the following code snippets into the `main` and execute.

#### Query Tick

Query for the latest tick data (price/size for best-bid/ask/last, accumuated volume, etc.).

```java
public class QueryTickSample {

    public static void main(String[] args) throws Exception {

        Bitflyer4j api = new Bitflyer4jFactory().createInstance();

        Tick.Request request = Tick.Request.builder().product("ETH_BTC").build();

        System.out.println(api.getMarketService().getTick(request).get());

        api.close();

    }

}
```

#### Send New Order

Send a new order. Note that this is for the "Child Order" in bitFlyer's terminology. 

```java
public class SendOrderSample {

    public static void main(String[] args) throws Exception {

        Bitflyer4j api = new Bitflyer4jFactory().createInstance();

        OrderCreate.Request request = OrderCreate.Request.builder()
                .product("FX_BTC_JPY").type(ConditionType.LIMIT).side(SideType.BUY)
                .price(new BigDecimal("12345.6789")).size(BigDecimal.ONE).build();

        System.out.println(api.getOrderService().sendOrder(request).get());

        api.close();

    }

}
```

#### Cancel Existing Order

Cancel an existing order. Specify either one of the order id or the acceptance id.

```java
public class CancelOrderSample {

    public static void main(String[] args) throws Exception {

        Bitflyer4j api = new Bitflyer4jFactory().createInstance();

        OrderCancel.Request request = OrderCancel.Request.builder()
                .product("BTCJPY_MAT1WK").orderId("JOR20150707-055555-022222").build();

        System.out.println(api.getOrderService().cancelOrder(request).get());

        api.close();

    }

}
```

#### Subscribe to Realtime Feed

Initiate a subscrption for the steaming market data.

```java
public class RealtimeSample {

    public static void main(String[] args) throws Exception {

        Bitflyer4j api = new Bitflyer4jFactory().createInstance();

        api.getRealtimeService().addListener(new RealtimeListenerAdapter() {
            @Override
            public void onTicks(String product, List<Tick> values) {
                System.out.println("(" + product + ")" + values);
            }
        });

        api.getRealtimeService().subscribeTick(Arrays.asList("BTC_JPY")).get();

        TimeUnit.SECONDS.sleep(30L);

        api.close();

    }

}
```

For the full list of supported features and sample codes, please refer to 
[Bitflyer4jTest](https://github.com/after-the-sunrise/bitflyer4j/blob/master/src/test/java/com/after_sunrise/cryptocurrency/bitflyer4j/Bitflyer4jTest.java).


## Features & Configurations

Below are some of the features and configurations available, which may be useful 
for specific use cases and executing environments.

### Private API Authentication

In order to use the [Private API](https://lightning.bitflyer.jp/docs?lang=en#http-private-api), 
specify the following authentication variables:
  * `bitflyer4j.auth_key`
  * `bitflyer4j.auth_secret`

By default, the library will try to retrieve the environment variables in the following priority:
  1. Runtime properties. (`-Dbitflyer4j.auth_key=... -Dbitflyer4j.auth_secret=...`)
  2. `${HOME}/.bitflyer4j` properties file.
  3. `bitflyer4j-site.properties` file in the classpath.

The library will scan from the top of the list, skipping the ones which are not available/accessible, 
and will use the first one found per variable.

Note that Confidential parameters shall only be stored privately, 
preferably by using the local `${HOME}/.bitflyer4j` properties 
file.  *DO NOT COMMIT, LOG, NOR DISSEMINATE THE CREDENTIALS.* 
 
```properties:${HOME}/.bitflyer4j
# Authentication
bitflyer4j.auth_key=MY_KEY_HERE
bitflyer4j.auth_secret=MY_SECRET_HERE
```
A template of the `.bitflyer4j` file can be downloded from 
[here](https://github.com/after-the-sunrise/bitflyer4j/blob/master/src/test/resources/.bitflyer4j).


### Network Proxy

In case if this library is to be used behind a network proxy, specify the following environment variables.
The variables are fetched by the library in the same manner, as described previously in the authentication section.

```properties:${HOME}/.bitflyer4j
# HTTP Proxy
bitflyer4j.http_proxy_type=HTTP
bitflyer4j.http_proxy_host=127.0.0.1
bitflyer4j.http_proxy_port=8080
```


### HTTP Access Throttling

bitFlyer limits the number of HTTP requests allowed per interval.

In this library, each HTTP calls are queued first, and a background thread takes them out of the queue, 
with a throttling mechanism to avoid DOS-attacking the service and comply with the limit.  Therefore, 
each HTTP calls are designed to return a `java.util.concurrent.CompletableFuture` instance, 
which is completed only after the HTTP request has been actually executed by the background thread.

To make these asynchronous HTTP calls synchronous, simply call the `CompletableFuture#get()` method.


### Other Configurations

Although there should be no need to overwrite the defaut configurations, the following parameters are externalized 
and can be overwritten by the environment variables.  For the complete list of configurable parameters and details, 
refer to the `KeyType` javadoc.


|Property Key                          |Default Value                             |Description                                                                           |
|--------------------------------------|------------------------------------------|--------------------------------------------------------------------------------------|
|bitflyer4j.site                       |local                                     |Optional free-text label to identify the environment which the client app is running. |
|bitflyer4j.auth_key                   |                                          |Authentication key for the private API.                                               |
|bitflyer4j.auth_secret                |                                          |Authentication secret for the private API.                                            |
|bitflyer4j.http_url                   |https://api.bitflyer.jp                   |Endpoint URL of the service.                                                          |
|bitflyer4j.http_proxy_type            |                                          |Proxy type (DIRECT/HTTP/SOCKS) used for HTTP calls. Leave empty to disable proxy.     |
|bitflyer4j.http_proxy_host            |                                          |Address of the proxy server. Proxy type must be specified for the proxy enablement.   |
|bitflyer4j.http_proxy_port            |                                          |Port of the proxy server. Proxy type must be specified for the proxy enablement.      |
|bitflyer4j.http_timeout               |                                          |HTTP socket/read timeout interval in millis. Leave empty for no timeout.              |
|bitflyer4j.http_threads               |4                                         |HTTP threads to utilize for HTTP access.                                              |
|bitflyer4j.http_limit_interval        |60000                                     |HTTP access throttling interval in millis.                                            |
|bitflyer4j.http_limit_criteria_address|500                                       |Number of allowed HTTP access for a single source IP, within the throttling interval. |
|bitflyer4j.http_limit_criteria_private|200                                       |Number of allowed HTTP access for private API calls, within the throttling interval.  |
|bitflyer4j.http_limit_criteria_dormant|10                                        |Number of allowed HTTP access for dormant accounts, within the throttling interval.   |
|bitflyer4j.pubnub_key                 |sub-c-52a9ab50-291b-11e5-baaa-0619f8945a4f|PubNub subscription key for realtime subscription.                                    |
|bitflyer4j.pubnub_reconnect           |LINEAR                                    |PubNub reconnect policy.                                                              |
|bitflyer4j.pubnub_secure              |true                                      |PubNub secure flag for enabling SSL.                                                  |


## Endpoint Paths

Currently implemented API endpoint paths are as follows:

- HTTP Public API
  - [x] Market List : `/v1/markets`
  - [x] Order Book : `/v1/board`
  - [x] Order Book : `/v1/board`
  - [x] Ticker : `/v1/ticker`
  - [x] Execution History : `/v1/executions`
  - [x] Exchange Status : `/v1/gethealth`
  - [x] Chat : `/v1/getchats`
- HTTP Private API
  - Account Detail
    - [x] Permissions : `/v1/me/getpermissions`
    - [x] Balance : `/v1/me/getbalance`
    - [x] Collateral : `/v1/me/getcollateral`
    - [x] Margin : `/v1/me/getcollateralaccounts`
    - [x] Address : `/v1/me/getaddresses`
    - [x] Coin In : `/v1/me/getcoinins`
    - [x] Coin Out : `/v1/me/getcoinouts`
    - [x] Bank : `/v1/me/getbankaccounts`
    - [x] Deposit : `/v1/me/getdeposits`
    - [x] Withdrawal : `/v1/me/getwithdrawals`
  - Account Action
    - [x] Withdraw : `/v1/me/withdraw`
  - Order Action
    - [x] New Child Order `/v1/me/sendchildorder`
    - [x] Cancel Child Order `/v1/me/cancelchildorder`
    - [x] New Parent Order `/v1/me/sendparentorder`
    - [x] Cancel Parent Order `/v1/me/cancelparentorder`
    - [x] Cancel All Orders `/v1/me/cancelallchildorders`
  - Order Detail
    - [x] Child Order List `/v1/me/getchildorders`
    - [x] Parent Order List `/v1/me/getparentorders`
    - [x] Parent Order Detail `/v1/me/getparentorder`
    - [x] Executions `/v1/me/getexecutions`
    - [x] Open Interest Summary `/v1/me/getpositions`
    - [x] Margin Change History `/v1/me/getcollateralhistory`
    - [x] Trading Commission `/v1/me/gettradingcommission`
- Realtime API
  - [x] Order Book Snapshot `lightning_board_snapshot_*`
  - [ ] Order Book Update `lightning_board_*`
  - [x] Ticker `lightning_ticker_*`
  - [x] Execution `lightning_executions_*`

[travis-page]:https://travis-ci.org/after-the-sunrise/bitflyer4j
[travis-icon]:https://travis-ci.org/after-the-sunrise/bitflyer4j.svg?branch=master
[coverall-page]:https://coveralls.io/github/after-the-sunrise/bitflyer4j?branch=master
[coverall-icon]:https://coveralls.io/repos/github/after-the-sunrise/bitflyer4j/badge.svg?branch=master
[maven-page]:https://maven-badges.herokuapp.com/maven-central/com.after_sunrise.cryptocurrency/bitflyer4j
[maven-icon]:https://maven-badges.herokuapp.com/maven-central/com.after_sunrise.cryptocurrency/bitflyer4j/badge.svg
[javadoc-page]:https://www.javadoc.io/doc/com.after_sunrise.cryptocurrency/bitflyer4j
[javadoc-icon]:https://www.javadoc.io/badge/com.after_sunrise.cryptocurrency/bitflyer4j.svg
