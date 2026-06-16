---
title: すぐに使える計装
weight: 40
default_lang_commit: 2d89b60b2e09d42ba96757b0afdbc31f54a2b0e7
cSpell:ignore: webflux webmvc
---

<!-- markdownlint-disable blanks-around-fences -->
<?code-excerpt path-base="examples/java/spring-starter"?>

いくつかのフレームワークに対して、すぐに使える計装が利用可能です。

{{< tabpane text=true >}} {{% tab "Properties" %}}

| 機能                     | プロパティ                                      | デフォルト |
| ------------------------ | ----------------------------------------------- | ---------- |
| JDBC                     | `otel.instrumentation.jdbc.enabled`             | true       |
| Logback                  | `otel.instrumentation.logback-appender.enabled` | true       |
| Logback MDC              | `otel.instrumentation.logback-mdc.enabled`      | true       |
| Spring Web               | `otel.instrumentation.spring-web.enabled`       | true       |
| Spring Web MVC           | `otel.instrumentation.spring-webmvc.enabled`    | true       |
| Spring WebFlux           | `otel.instrumentation.spring-webflux.enabled`   | true       |
| Kafka                    | `otel.instrumentation.kafka.enabled`            | true       |
| MongoDB                  | `otel.instrumentation.mongo.enabled`            | true       |
| Micrometer               | `otel.instrumentation.micrometer.enabled`       | false      |
| R2DBC (Reactive JDBC) | `otel.instrumentation.r2dbc.enabled`            | true       |

特定の計装を無効にするには、次のようにします。

```yaml
otel:
  instrumentation:
    logback-appender:
      enabled: false
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

[宣言的設定](../declarative-configuration/)では、計装の有効化または無効化は`otel.distribution.spring_starter.instrumentation`配下の一元化されたリストを使用します。
計装名は`-`（ケバブケース）ではなく`_`（スネークケース）を使用します。

| 機能                     | 名前               | デフォルト |
| ------------------------ | ------------------ | ---------- |
| JDBC                     | `jdbc`             | enabled    |
| Logback                  | `logback_appender` | enabled    |
| Logback MDC              | `logback_mdc`      | enabled    |
| Spring Web               | `spring_web`       | enabled    |
| Spring Web MVC           | `spring_webmvc`    | enabled    |
| Spring WebFlux           | `spring_webflux`   | enabled    |
| Kafka                    | `kafka`            | enabled    |
| MongoDB                  | `mongo`            | enabled    |
| Micrometer               | `micrometer`       | disabled   |
| R2DBC (Reactive JDBC) | `r2dbc`            | enabled    |

特定の計装を無効にするには、次のように書きます。

```yaml
otel:
  distribution:
    spring_starter:
      instrumentation:
        disabled:
          - logback_appender
```

{{% /tab %}} {{< /tabpane >}}

## 計装を選択的に有効化する {#turn-on-instrumentations-selectively}

{{< tabpane text=true >}} {{% tab "Properties" %}}

特定の計装のみを使用するには、まずすべての計装をオフにしてから、計装を1つずつ有効にします。

```yaml
otel:
  instrumentation:
    common:
      default-enabled: false
    jdbc:
      enabled: true
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

[宣言的設定](../declarative-configuration/)では、`default_enabled`を`false`に設定し、使用したい計装を`enabled`にリストします。

```yaml
otel:
  distribution:
    spring_starter:
      instrumentation:
        default_enabled: false
        enabled:
          - jdbc
```

{{% /tab %}} {{< /tabpane >}}

## 共通計装設定 {#common-instrumentation-configuration}

すべてのデータベース計装に共通のプロパティ。

{{< tabpane text=true >}} {{% tab "Properties" %}}

すべてのデータベース計装に対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation:
    common:
      db-statement-sanitizer:
        enabled: true # default: true
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

すべてのデータベース計装に対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation/development:
    java:
      common:
        database:
          statement_sanitizer:
            enabled: true
```

{{% /tab %}} {{< /tabpane >}}

## JDBC計装 {#jdbc-instrumentation}

{{< tabpane text=true >}} {{% tab "Properties" %}}

JDBCに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation:
    jdbc:
      statement-sanitizer:
        enabled: true # default: true
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

JDBCに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation/development:
    java:
      jdbc:
        statement_sanitizer:
          enabled: true
```

{{% /tab %}} {{< /tabpane >}}

## Logback {#logback}

システムプロパティで実験的機能を有効にして、属性をキャプチャできます。

{{< tabpane text=true >}} {{% tab "Properties" %}}

| プロパティ                                       | 型      | デフォルト | 説明                                                                                                                                                                                               |
| ------------------------------------------------ | ------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `experimental-log-attributes`                    | Boolean | false      | 実験的なログ属性`thread.name`と`thread.id`のキャプチャを有効にします。                                                                                                                             |
| `experimental.capture-code-attributes`           | Boolean | false      | [ソースコード属性][source code attributes]のキャプチャを有効にします。ログサイトでソースコード属性をキャプチャすると、パフォーマンスのオーバーヘッドが発生する可能性があることに注意してください。 |
| `experimental.capture-marker-attribute`          | Boolean | false      | Logbackマーカーを属性としてキャプチャすることを有効にします。                                                                                                                                      |
| `experimental.capture-key-value-pair-attributes` | Boolean | false      | Logbackキーバリューペアを属性としてキャプチャすることを有効にします。                                                                                                                              |
| `experimental.capture-logger-context-attributes` | Boolean | false      | Logbackロガーコンテキストプロパティを属性としてキャプチャすることを有効にします。                                                                                                                  |
| `experimental.capture-mdc-attributes`            | String  |            | キャプチャするMDC属性のカンマ区切りリスト。すべての属性をキャプチャするにはワイルドカード文字`*`を使用します。                                                                                     |

```yaml
otel:
  instrumentation:
    logback-appender:
      experimental-log-attributes: false
      experimental:
        capture-code-attributes: false
        capture-marker-attribute: false
        capture-key-value-pair-attributes: false
        capture-logger-context-attributes: false
        capture-mdc-attributes: '*'
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

| プロパティ                                      | 型      | デフォルト | 説明                                                                                                                                                                                               |
| ----------------------------------------------- | ------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `experimental_log_attributes/development`       | Boolean | false      | 実験的なログ属性`thread.name`と`thread.id`のキャプチャを有効にします。                                                                                                                             |
| `capture_code_attributes/development`           | Boolean | false      | [ソースコード属性][source code attributes]のキャプチャを有効にします。ログサイトでソースコード属性をキャプチャすると、パフォーマンスのオーバーヘッドが発生する可能性があることに注意してください。 |
| `capture_marker_attribute/development`          | Boolean | false      | Logbackマーカーを属性としてキャプチャすることを有効にします。                                                                                                                                      |
| `capture_key_value_pair_attributes/development` | Boolean | false      | Logbackキーバリューペアを属性としてキャプチャすることを有効にします。                                                                                                                              |
| `capture_logger_context_attributes/development` | Boolean | false      | Logbackロガーコンテキストプロパティを属性としてキャプチャすることを有効にします。                                                                                                                  |
| `capture_mdc_attributes/development`            | String  |            | キャプチャするMDC属性のカンマ区切りリスト。すべての属性をキャプチャするにはワイルドカード文字`*`を使用します。                                                                                     |

```yaml
otel:
  instrumentation/development:
    java:
      logback_appender:
        experimental_log_attributes/development: false
        capture_code_attributes/development: false
        capture_marker_attribute/development: false
        capture_key_value_pair_attributes/development: false
        capture_logger_context_attributes/development: false
        capture_mdc_attributes/development: '*'
```

{{% /tab %}} {{< /tabpane >}}

[source code attributes]: /docs/specs/semconv/general/attributes/#source-code-attributes

または、`logback.xml`または`logback-spring.xml`ファイルにOpenTelemetry Logbackアペンダーを追加することで、これらの機能を有効にできます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
            </pattern>
        </encoder>
    </appender>
    <appender name="OpenTelemetry"
        class="io.opentelemetry.instrumentation.logback.appender.v1_0.OpenTelemetryAppender">
        <captureExperimentalAttributes>false</captureExperimentalAttributes>
        <captureCodeAttributes>true</captureCodeAttributes>
        <captureMarkerAttribute>true</captureMarkerAttribute>
        <captureKeyValuePairAttributes>true</captureKeyValuePairAttributes>
        <captureLoggerContext>true</captureLoggerContext>
        <captureMdcAttributes>*</captureMdcAttributes>
    </appender>
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="OpenTelemetry"/>
    </root>
</configuration>
```

## Spring Web自動設定 {#spring-web-autoconfiguration}

[opentelemetry-spring-web-3.1](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/spring/spring-web/spring-web-3.1/library)で定義された`RestTemplate`トレースインターセプターの自動設定を提供します。
この自動設定は、`RestTemplate`ビーンポストプロセッサーを適用することで、Spring `RestTemplate`ビーンを使用して送信されるすべてのリクエストを計装します。
この機能はSpring Webバージョン3.1以降でサポートされています。
OpenTelemetry`RestTemplate`インターセプターの詳細については、[opentelemetry-spring-web-3.1](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/spring/spring-web/spring-web-3.1/library)を参照してください。

以下の`RestTemplate`の作成方法がサポートされています。

<!-- prettier-ignore-start -->
<?code-excerpt "src/main/java/otel/RestTemplateConfig.java"?>
```java
package otel;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

  @Bean
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }
}
```

<?code-excerpt "src/main/java/otel/RestTemplateController.java"?>
```java
package otel;

import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class RestTemplateController {

  private final RestTemplate restTemplate;

  public RestTemplateController(RestTemplateBuilder restTemplateBuilder) {
    restTemplate = restTemplateBuilder.rootUri("http://localhost:8080").build();
  }
}
```
<!-- prettier-ignore-end -->

以下の`RestClient`の作成方法がサポートされています。

<!-- prettier-ignore-start -->
<?code-excerpt "src/main/java/otel/RestClientConfig.java"?>
```java
package otel;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;

@Configuration
public class RestClientConfig {

  @Bean
  public RestClient restClient() {
    return RestClient.create();
  }
}
```

<?code-excerpt "src/main/java/otel/RestClientController.java"?>
```java
package otel;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestClient;

@RestController
public class RestClientController {

  private final RestClient restClient;

  public RestClientController(RestClient.Builder restClientBuilder) {
    restClient = restClientBuilder.baseUrl("http://localhost:8080").build();
  }
}
```
<!-- prettier-ignore-end -->

Javaエージェントと同様に、以下のエンティティのキャプチャを設定できます。

- [HTTPリクエストおよびレスポンスヘッダー](/docs/zero-code/java/agent/instrumentation/http/#capturing-http-request-and-response-headers)
- [既知のHTTPメソッド](/docs/zero-code/java/agent/instrumentation/http/#configuring-known-http-methods)
- [実験的HTTPテレメトリ](/docs/zero-code/java/agent/instrumentation/http/#enabling-experimental-http-telemetry)

## Spring Web MVC自動設定 {#spring-web-mvc-autoconfiguration}

この機能は、アプリケーションコンテキストに[テレメトリ生成サーブレット`Filter`](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/instrumentation/spring/spring-webmvc/spring-webmvc-5.3/library/src/main/java/io/opentelemetry/instrumentation/spring/webmvc/v5_3/WebMvcTelemetryProducingFilter.java)ビーンを追加することで、Spring WebMVCコントローラーの計装を自動設定します。
フィルターは、リクエストの実行をサーバースパンでデコレートし、HTTPリクエストで受信した場合は受信トレーシングコンテキストを伝搬します。
OpenTelemetry Spring WebMVC計装の詳細については、[opentelemetry-spring-webmvc-5.3計装ライブラリ](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/spring/spring-webmvc/spring-webmvc-5.3/library)を参照してください。

Javaエージェントと同様に、以下のエンティティのキャプチャを設定できます。

- [HTTPリクエストおよびレスポンスヘッダー](/docs/zero-code/java/agent/instrumentation/http/#capturing-http-request-and-response-headers)
- [既知のHTTPメソッド](/docs/zero-code/java/agent/instrumentation/http/#configuring-known-http-methods)
- [実験的HTTPテレメトリ](/docs/zero-code/java/agent/instrumentation/http/#enabling-experimental-http-telemetry)

## Spring WebFlux自動設定 {#spring-webflux-autoconfiguration}

[opentelemetry-spring-webflux-5.3](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/spring/spring-webflux/spring-webflux-5.3/library)で定義されたOpenTelemetry WebClient ExchangeFilterの自動設定を提供します。
この自動設定は、ビーンポストプロセッサーを適用することで、SpringのWebClientとWebClient Builderビーンを使用して送信されるすべての送信HTTPリクエストを計装します。
この機能は、Spring WebFluxバージョン5.0以降でサポートされています。詳細については、[opentelemetry-spring-webflux-5.3](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/spring/spring-webflux/spring-webflux-5.3/library)を参照してください。

以下の`WebClient`の作成方法がサポートされています。

<!-- prettier-ignore-start -->
<?code-excerpt "src/main/java/otel/WebClientConfig.java"?>
```java
package otel;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

  @Bean
  public WebClient webClient() {
    return WebClient.create();
  }
}
```

<?code-excerpt "src/main/java/otel/WebClientController.java"?>
```java
package otel;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.function.client.WebClient;

@RestController
public class WebClientController {

  private final WebClient webClient;

  public WebClientController(WebClient.Builder webClientBuilder) {
    webClient = webClientBuilder.baseUrl("http://localhost:8080").build();
  }
}
```
<!-- prettier-ignore-end -->

## Kafka計装 {#kafka-instrumentation}

Kafkaクライアント計装の自動設定を提供します。

{{< tabpane text=true >}} {{% tab "Properties" %}}

Kafkaに対して実験的なスパン属性のキャプチャを有効にします。

```yaml
otel:
  instrumentation:
    kafka:
      experimental-span-attributes: false # default: false
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

Kafkaに対して実験的なスパン属性のキャプチャを有効にします。

```yaml
otel:
  instrumentation/development:
    java:
      kafka:
        experimental_span_attributes/development: false
```

{{% /tab %}} {{< /tabpane >}}

## Micrometer計装 {#micrometer-instrumentation}

MicrometerからOpenTelemetryへのブリッジの自動設定を提供します。

## MongoDB計装 {#mongodb-instrumentation}

MongoDBクライアント計装の自動設定を提供します。

{{< tabpane text=true >}} {{% tab "Properties" %}}

MongoDBに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation:
    mongo:
      statement-sanitizer:
        enabled: true # default: true
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

MongoDBに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation/development:
    java:
      mongo:
        statement_sanitizer:
          enabled: true
```

{{% /tab %}} {{< /tabpane >}}

## R2DBC計装 {#r2dbc-instrumentation}

OpenTelemetry R2DBC計装の自動設定を提供します。

{{< tabpane text=true >}} {{% tab "Properties" %}}

R2DBCに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation:
    r2dbc:
      statement-sanitizer:
        enabled: true # default: true
```

{{% /tab %}} {{% tab "Declarative Configuration" %}}

R2DBCに対してDBステートメントのサニタイズを有効にします。

```yaml
otel:
  instrumentation/development:
    java:
      r2dbc:
        statement_sanitizer:
          enabled: true
```

{{% /tab %}} {{< /tabpane >}}
