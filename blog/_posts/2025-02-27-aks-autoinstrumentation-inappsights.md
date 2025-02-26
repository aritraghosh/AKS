---
title: "Auto Instrumentation for Azure Monitor Application Insights in AKS"
description: "Simplify application monitoring in Azure Kubernetes Service without code changes"
date: 2025-02-27
author: Aritra Ghosh, Abinet Abinet
categories: 
- developer
- operations
---

_Simplify monitoring your AKS applications with zero code changes_

<hr>

## Introducing Auto Instrumentation for Azure Monitor Application Insights

Today, we're excited to announce the public preview of Auto Instrumentation for Azure Monitor Application Insights in Azure Kubernetes Service (AKS). This powerful new capability allows developers and operations teams to get deep visibility into their Kubernetes workloads without modifying a single line of code.

For teams running modern, containerized applications, understanding performance and troubleshooting issues has traditionally required significant instrumentation work. Auto Instrumentation changes this paradigm by automatically collecting comprehensive telemetry from your Java and Node.js applications running in AKS.

## Why Auto Instrumentation?

* **Zero code changes required**: Deploy monitoring without touching your source code or rebuilding containers
* **Simplified operations**: Enable monitoring for entire namespaces with a single configuration
* **Full Application Insights capabilities**: Get distributed tracing, performance metrics, and detailed diagnostics automatically
* **OpenTelemetry standards**: Built on the Azure Monitor OpenTelemetry Distro for future-proof monitoring

## How It Works

Auto Instrumentation works by dynamically injecting the Azure Monitor OpenTelemetry Distro into your application pods at runtime. This happens automatically when you deploy applications to your prepared AKS cluster.

Here's the simplified flow:

1. You enable Auto Instrumentation on your AKS cluster
2. You create an Instrumentation custom resource with your monitoring preferences
3. You deploy or restart your applications
4. The Auto Instrumentation components automatically inject the monitoring agents
5. Telemetry flows to your Application Insights resource

![Auto Instrumentation Architecture Diagram](/api/placeholder/800/400)

## Auto Instrumentation in Action: A Real-World Example

To demonstrate the power of Auto Instrumentation, we deployed a sample microservices e-commerce application in AKS. This application consists of multiple services written in Java and Node.js, including:

- A frontend web service
- A product catalog service
- An order processing service
- A user authentication service
- A payment processing service

After enabling Auto Instrumentation on our AKS cluster and configuring our Instrumentation custom resource, we simply restarted our deployments and immediately started seeing rich telemetry in Application Insights - all without changing a single line of code.

### Inside the Application Insights Experience

Once your applications are running with Auto Instrumentation enabled, telemetry will begin flowing to Application Insights. Navigate to your Application Insights resource in the Azure portal to view:

* Application Map showing service dependencies
* Performance metrics for your applications
* End-to-end distributed traces
* Detailed logs and exceptions

![Application Map showing service connections](/api/placeholder/800/500)

When we open our Application Insights resource in the Azure portal, the first thing we see is the Application Map. This automatically generated topology view shows how our e-commerce services interact with each other, including external dependencies like databases and third-party APIs.

In our application, we can immediately see that the product catalog service is being called by both the frontend and the order processing service. We also notice an unexpectedly high dependency on our Redis cache from the authentication service - something we wouldn't have discovered without this visibility.

## Key Monitoring Features in Action

### 1. Distributed Tracing: Following a Customer Order

When a customer reported a slow checkout experience, we used distributed tracing to investigate. By searching for their specific order ID in Application Insights, we found the end-to-end transaction:

![Distributed trace example](/api/placeholder/800/300)

The trace revealed that the order took 4.2 seconds to complete, with most of the time (3.8 seconds) spent in the payment processing service. Drilling into this span, we discovered that a third-party payment gateway was responding slowly. 

Each span in the trace provided detailed context:
- HTTP method, URL, and status codes
- Database queries
- Service-to-service calls
- Custom context like order IDs and user IDs
- Performance metrics at each step

### 2. Performance Metrics: Identifying Memory Issues

After running with Auto Instrumentation for a few days, we noticed our product catalog service was occasionally becoming unresponsive. The performance dashboard showed a clear pattern:

![Performance metrics dashboard](/api/placeholder/800/400)

The memory utilization chart revealed a sawtooth pattern typical of memory leaks, with usage climbing until the JVM garbage collector ran. With this insight, we could address the root cause in our caching layer rather than just increasing the container memory limits.

### 3. Log Correlation: Debugging Authentication Failures

After deploying a new version of our authentication service, some users reported intermittent login failures. With log correlation enabled in our Instrumentation configuration, we could see application logs directly in Application Insights:

![Log correlation example](/api/placeholder/800/350)

The logs showed exception messages from our authentication service, correlated with the exact trace IDs of the failed requests. This correlation revealed that only users with certain special characters in their usernames were affected - a validation issue that our unit tests had missed.

The log correlation feature was enabled with a simple annotation in our deployment:

```yaml
monitor.azure.com/enable-application-logs: "true"
```

### 4. Dependency Mapping: Database Connection Issues

Our application map automatically detected an issue with database connections in the order processing service:

![Dependency mapping with issues](/api/placeholder/800/400)

A red dependency line showed high failure rates when connecting to the order database. Clicking on this dependency revealed detailed metrics including:
- 42% failure rate
- Average response time of 1.2 seconds
- Error distribution by type

This visibility led us to discover that our database connection pool was misconfigured, causing intermittent timeouts during peak loads.

## Real-World Deployment Scenarios

In our e-commerce application, we used different Auto Instrumentation approaches based on team needs:

### Production Environment: Namespace-Wide Monitoring

For our production namespace, we wanted comprehensive visibility across all services. Using a single `default` Instrumentation resource, we enabled monitoring for all Java and Node.js applications with consistent settings:

![Namespace-wide monitoring dashboard](/api/placeholder/800/400)

The result was a unified view of our entire application stack, with consistent naming conventions and configuration. This made it easy for our operations team to monitor the overall health of the system and quickly identify issues during high-traffic events like flash sales.

### Development Environment: Per-Team Customization

In our development namespace, different teams wanted different monitoring configurations. The payment team needed detailed SQL query tracking, while the frontend team wanted to focus on browser performance metrics.

Using per-deployment configuration with annotations, each team customized their monitoring:

```yaml
# Payment service deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: "payment-team-config"
```

This flexibility enabled each team to optimize their monitoring experience while still working within a shared cluster infrastructure.

## Business Impact: From Reactive to Proactive

The most significant benefit of Auto Instrumentation for our team has been the shift from reactive to proactive monitoring. Before implementing this solution, we often discovered issues only after users reported them, leading to longer resolution times and customer frustration.

With Auto Instrumentation, we've seen tangible improvements:

- **85% reduction in MTTR (Mean Time to Resolution)** - When issues occur, we can pinpoint the exact service and dependency causing the problem
- **40% decrease in P1 incidents** - Many issues are caught and resolved before they impact users
- **23% improvement in API performance** - Identifying and fixing slow endpoints and inefficient database queries
- **Zero code changes required** - All these benefits without modifying our application code

![Incident reduction chart](/api/placeholder/800/350)

## Getting Started Today

Auto Instrumentation for Azure Monitor Application Insights is available today in preview for all AKS clusters in the Azure public cloud. It supports Java and Node.js applications, with more languages coming in the future.

To get started:
1. Ensure you have an AKS cluster and an Application Insights resource
2. Enable the preview feature and prepare your cluster
3. Create an Instrumentation custom resource with your monitoring preferences
4. Deploy or restart your applications

Visit the [official documentation](https://learn.microsoft.com/azure/azure-monitor/app/azure-kubernetes-auto-instrumentation) for detailed setup instructions.

## Next Steps

- [Read the full documentation](https://learn.microsoft.com/azure/azure-monitor/app/azure-kubernetes-auto-instrumentation)
- [Learn about Application Insights capabilities](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview)
- [Explore OpenTelemetry in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/app/opentelemetry-overview)

We're excited to see how Auto Instrumentation transforms your Kubernetes monitoring experience. Share your feedback with us through the Azure portal or on our [GitHub repository](https://github.com/Azure/AKS)!