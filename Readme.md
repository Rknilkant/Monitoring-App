# 📈 Go Monitoring App with Prometheus & Grafana

Welcome to the `demo-app` – a lightweight monitoring stack built in **Go** 🐹 using the **Gin web framework**, with powerful observability enabled via **Prometheus** 📊 and **Grafana** 📉.

> ⚙️ Ideal for learning, dev environments, or bootstrapping monitoring into your microservices.

---

## 🚀 Features

- ✅ REST API built in Go (Gin framework)
- 📡 Custom Prometheus metrics using `prometheus/client_golang`
- 📊 Real-time monitoring via Prometheus
- 📈 Grafana dashboard with auto-provisioned Prometheus datasource
- 🐳 Docker + Docker Compose for easy containerization and orchestration

---

## 📁 Project Structure

```plaintext
.
├── main.go                       # Go API with Prometheus metrics
├── Dockerfile                   # Multi-stage Docker build
├── compose.yml                  # App + Prometheus + Grafana
├── go.mod / go.sum              # Go dependencies
└── Docker/
    ├── prometheus.yml           # Prometheus scrape config
    └── grafana.yml              # Grafana datasource provisioning



## 🛠 Installation Guide

1. Clone the Repository

```bash
https://github.com/Rknilkant/Monitoring-App.git

```

## Deployment

To deploy this project run

```bash
  go mod init demo-app
```

## Create A New File main.go

```bash

package main

import (
    "strconv"

    "github.com/gin-gonic/gin"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// Define metrics
var (
    HttpRequestTotal = prometheus.NewCounterVec(prometheus.CounterOpts{
        Name: "api_http_request_total",
        Help: "Total number of requests processed by the API",
    }, []string{"path", "status"})

    HttpRequestErrorTotal = prometheus.NewCounterVec(prometheus.CounterOpts{
        Name: "api_http_request_error_total",
        Help: "Total number of errors returned by the API",
    }, []string{"path", "status"})
)

// Custom registry (without default Go metrics)
var customRegistry = prometheus.NewRegistry()

// Register metrics with custom registry
func init() {
    customRegistry.MustRegister(HttpRequestTotal, HttpRequestErrorTotal)
}

func main() {
    router := gin.Default()

    // Register /metrics before middleware
    router.GET("/metrics", PrometheusHandler())

    router.Use(RequestMetricsMiddleware())
    router.GET("/health", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Up and running!",
        })
    })
    router.GET("/v1/users", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello from /v1/users",
        })
    })

    router.Run(":8000")
}

// Custom metrics handler with custom registry
func PrometheusHandler() gin.HandlerFunc {
    h := promhttp.HandlerFor(customRegistry, promhttp.HandlerOpts{})
    return func(c *gin.Context) {
        h.ServeHTTP(c.Writer, c.Request)
    }
}

// Middleware to record incoming requests metrics
func RequestMetricsMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        path := c.Request.URL.Path
        c.Next()
        status := c.Writer.Status()
        if status < 400 {
            HttpRequestTotal.WithLabelValues(path, strconv.Itoa(status)).Inc()
        } else {
            HttpRequestErrorTotal.WithLabelValues(path, strconv.Itoa(status)).Inc()
        }
    }
}

```
## Execute

```bash
go mod tidy

```
 
## Run The Application

```bash
go run main.go

```
