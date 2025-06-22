---
title: エクスポーター
weight: 50
description: テレメトリデータを処理してエクスポートする
default_lang_commit: 2f850a610b5f7da5730265b32c25c9226dc09e5f
cSpell:ignore: LOWMEMORY
---

<!-- markdownlint-disable no-duplicate-heading -->

{{% docs/languages/exporters/intro %}}

### 依存関係 {#otlp-dependencies}

OTLP エンドポイント（[OpenTelemetry Collector](#collector-setup)、[Jaeger](#jaeger)、[Prometheus](#prometheus) など）にテレメトリーデータを送信したい場合、データを転送するために次の 2 つの異なるプロトコルから選択できます。

- [HTTP/protobuf](https://pypi.org/project/opentelemetry-exporter-otlp-proto-http/)
- [gRPC](https://pypi.org/project/opentelemetry-exporter-otlp-proto-grpc/)

プロジェクトの依存関係として、それぞれのエクスポーターパッケージをインストールすることから始めます。

{{< tabpane text=true >}} {{% tab "HTTP/Proto" %}}

```shell
pip install opentelemetry-exporter-otlp-proto-http
```

{{% /tab %}} {{% tab gRPC %}}

```shell
pip install opentelemetry-exporter-otlp-proto-grpc
```

{{% /tab %}} {{< /tabpane >}}

### 使用方法 {#usage}

次に、コードで OTLP エンドポイントを指すようにエクスポーターを設定します。

{{< tabpane text=true >}} {{% tab "HTTP/Proto" %}}

```python
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

from opentelemetry import metrics
from opentelemetry.exporter.otlp.proto.http.metric_exporter import OTLPMetricExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader

# サービス名はほとんどのバックエンドで必要です
resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

tracerProvider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(OTLPSpanExporter(endpoint="<traces-endpoint>/v1/traces"))
tracerProvider.add_span_processor(processor)
trace.set_tracer_provider(tracerProvider)

reader = PeriodicExportingMetricReader(
    OTLPMetricExporter(endpoint="<traces-endpoint>/v1/metrics")
)
meterProvider = MeterProvider(resource=resource, metric_readers=[reader])
metrics.set_meter_provider(meterProvider)
```

{{% /tab %}} {{% tab gRPC %}}

```python
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

from opentelemetry import metrics
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader

# サービス名はほとんどのバックエンドで必要です
resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

tracerProvider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(OTLPSpanExporter(endpoint="your-endpoint-here"))
tracerProvider.add_span_processor(processor)
trace.set_tracer_provider(tracerProvider)

reader = PeriodicExportingMetricReader(
    OTLPMetricExporter(endpoint="localhost:5555")
)
meterProvider = MeterProvider(resource=resource, metric_readers=[reader])
metrics.set_meter_provider(meterProvider)
```

{{% /tab %}} {{< /tabpane >}}

## コンソール {#console}

計装をデバッグしたり、開発時にローカルで値を確認したりするために、テレメトリーデータをコンソール（標準出力）に書き出すエクスポーターを使用できます。

`ConsoleSpanExporter` と `ConsoleMetricExporter` は `opentelemetry-sdk` パッケージに含まれています。

```python
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor, ConsoleSpanExporter

from opentelemetry import metrics
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader, ConsoleMetricExporter

# サービス名はほとんどのバックエンドで必要です。
# コンソールエクスポートには必要ありませんが、
# サービス名を設定することをお勧めします。
resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

tracerProvider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(ConsoleSpanExporter())
tracerProvider.add_span_processor(processor)
trace.set_tracer_provider(tracerProvider)

reader = PeriodicExportingMetricReader(ConsoleMetricExporter())
meterProvider = MeterProvider(resource=resource, metric_readers=[reader])
metrics.set_meter_provider(meterProvider)
```

{{% alert title="注意" %}}

各計装種別にはテンポラリティプリセットがあります。これらのプリセットは環境変数 `OTEL_EXPORTER_METRICS_TEMPORALITY_PREFERENCE` で設定できます。
例を挙げましょう。

```sh
export OTEL_EXPORTER_METRICS_TEMPORALITY_PREFERENCE="DELTA"
```

`OTEL_EXPORTER_METRICS_TEMPORALITY_PREFERENCE` のデフォルト値は`"CUMULATIVE"` です。

この環境変数の使用可能な値とそれに対応する設定は次のとおりです。

- `CUMULATIVE`

  - `Counter`: `CUMULATIVE`
  - `UpDownCounter`: `CUMULATIVE`
  - `Histogram`: `CUMULATIVE`
  - `ObservableCounter`: `CUMULATIVE`
  - `ObservableUpDownCounter`: `CUMULATIVE`
  - `ObservableGauge`: `CUMULATIVE`

- `DELTA`

  - `Counter`: `DELTA`
  - `UpDownCounter`: `CUMULATIVE`
  - `Histogram`: `DELTA`
  - `ObservableCounter`: `DELTA`
  - `ObservableUpDownCounter`: `CUMULATIVE`
  - `ObservableGauge`: `CUMULATIVE`

- `LOWMEMORY`
  - `Counter`: `DELTA`
  - `UpDownCounter`: `CUMULATIVE`
  - `Histogram`: `DELTA`
  - `ObservableCounter`: `CUMULATIVE`
  - `ObservableUpDownCounter`: `CUMULATIVE`
  - `ObservableGauge`: `CUMULATIVE`

`OTEL_EXPORTER_METRICS_TEMPORALITY_PREFERENCE` を `CUMULATIVE`、`DELTA`、`LOWMEMORY` 以外の値に設定すると、警告がログに記録され、この環境変数は `CUMULATIVE` に設定されます。

{{% /alert %}}

{{% include "exporters/jaeger.md" %}}

{{% include "exporters/prometheus-setup.md" %}}

### 依存関係 {#prometheus-dependencies}

アプリケーションの依存関係として[エクスポーターパッケージ](https://pypi.org/project/opentelemetry-exporter-prometheus/)をインストールします。

```sh
pip install opentelemetry-exporter-prometheus
```

エクスポーターを使用し、Prometheus バックエンドにデータを送信するように OpenTelemetry 設定を更新します。

```python
from prometheus_client import start_http_server

from opentelemetry import metrics
from opentelemetry.exporter.prometheus import PrometheusMetricReader
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

# サービス名はほとんどのバックエンドで必要です
resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

# Prometheus クライアントを開始
start_http_server(port=9464, addr="localhost")
# スクレイプリクエストに応答するために SDK からオンデマンドでメトリクスを取得する
# PrometheusMetricReader を初期化
reader = PrometheusMetricReader()
provider = MeterProvider(resource=resource, metric_readers=[reader])
metrics.set_meter_provider(provider)
```

上記により、<http://localhost:9464/metrics> でメトリクスにアクセスできます。
Prometheus または Prometheus レシーバーを持つ OpenTelemetry Collector は、このエンドポイントからメトリクスをスクレイプできます。

{{% include "exporters/zipkin-setup.md" %}}

### 依存関係 {#zipkin-dependencies}

トレースデータを [Zipkin](https://zipkin.io/) に送信するには、データを転送するために次の 2 つの異なるプロトコルから選択できます。

- [HTTP/protobuf](https://pypi.org/project/opentelemetry-exporter-zipkin-proto-http/)
- [Thrift](https://pypi.org/project/opentelemetry-exporter-zipkin-json/)

アプリケーションの依存関係としてエクスポーターパッケージをインストールします。

{{< tabpane text=true >}} {{% tab "HTTP/Proto" %}}

```shell
pip install opentelemetry-exporter-zipkin-proto-http
```

{{% /tab %}} {{% tab Thrift %}}

```shell
pip install opentelemetry-exporter-zipkin-json
```

{{% /tab %}} {{< /tabpane >}}

エクスポーターを使用し、Zipkin バックエンドにデータを送信するように OpenTelemetry 設定を更新します。

{{< tabpane text=true >}} {{% tab "HTTP/Proto" %}}

```python
from opentelemetry import trace
from opentelemetry.exporter.zipkin.proto.http import ZipkinExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

zipkin_exporter = ZipkinExporter(endpoint="http://localhost:9411/api/v2/spans")

provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(zipkin_exporter)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
```

{{% /tab %}} {{% tab Thrift %}}

```python
from opentelemetry import trace
from opentelemetry.exporter.zipkin.json import ZipkinExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

resource = Resource.create(attributes={
    SERVICE_NAME: "your-service-name"
})

zipkin_exporter = ZipkinExporter(endpoint="http://localhost:9411/api/v2/spans")

provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(zipkin_exporter)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
```

{{% /tab %}} {{< /tabpane >}}

{{% include "exporters/outro.md" `https://opentelemetry-python.readthedocs.io/en/latest/sdk/trace.export.html#opentelemetry.sdk.trace.export.SpanExporter` %}}

```python
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace.export import SimpleSpanProcessor

processor = SimpleSpanProcessor(OTLPSpanExporter(endpoint="your-endpoint-here"))
```
