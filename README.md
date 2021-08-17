# gorm-opentracing [migrated]

[![Go Report Card](https://goreportcard.com/badge/github.com/yeqown/gorm-opentracing)](https://goreportcard.com/report/github.com/yeqown/gorm-opentracing) [![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/yeqown/gorm-opentracing)

opentracing support for gorm2. 

> ### **!!! This repository has be migrated to [github.com/go-gorm/opentracing](https://github.com/go-gorm/opentracing).**

### Features

- [x] Record `SQL` in `span` logs.
- [x] Record `Result` in `span` logs.
- [x] Record `Table` in `span` tags.
- [x] Record `Error` in `span` tags and logs.
- [x] Register `Create` `Query` `Delete` `Update` `Row` `Raw` tracing callbacks. 

### Get Started

I assume that you already have an opentracing Tracer client started in your project.

```go
func main() {
	var db *gorm.DB
	
	db.Use(gormopentracing.New())
}
```

Otherwise, you need to deploy distributed tracing server(jaeger, zipkin for example), then
you need to boot tracer client in yours project and set tracer to opentracing.

```go
import (
	"github.com/opentracing/opentracing-go"
	"github.com/uber/jaeger-client-go"
	"github.com/uber/jaeger-client-go/config"
	jaegerlog "github.com/uber/jaeger-client-go/log"
)

func bootTracerBasedJaeger() {
	// jaeger tracer configuration
	cfg := &config.Configuration{
		Sampler: &config.SamplerConfig{
			Type:  jaeger.SamplerTypeConst,
			Param: 1,
		},
		ServiceName: "gormopentracing",
		Reporter: &config.ReporterConfig{
			LogSpans: true,
			//LocalAgentHostPort:  "127.0.0.1:6381",
			BufferFlushInterval: 100 * time.Millisecond,
			CollectorEndpoint:   "http://127.0.0.1:14268/api/traces",
		},
	}

	// jaeger tracer client 
	tracer, _, err := cfg.NewTracer(
		config.Logger(jaegerlog.StdLogger),
		config.ZipkinSharedRPCSpan(true),
	)
	if err != nil {
		log.Printf("failed to use jaeger tracer plugin, got error %v", err)
		os.Exit(1)
	}
	
	// set into opentracing
	opentracing.SetGlobalTracer(tracer)
}
```

### Plugin options

```go
// WithLogResult log result into span log, default: disabled.
func WithLogResult(logResult bool)
```

### Snapshots

<img src="./assets/shot1.png" width="100%"/>

<img src="./assets/shot2.png" width="100%"/>
