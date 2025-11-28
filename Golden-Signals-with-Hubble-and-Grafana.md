# Golden Signals with Hubble and Grafana

## Cluster Overview

This document provides a comprehensive analysis of the Kubernetes cluster, focusing on observability through Hubble and Grafana for monitoring Golden Signals (Latency, Traffic, Errors, and Saturation).

### Cluster Facts

**Kubernetes Version:** v1.33.1

**Node Count:** 3 nodes
- 1 control-plane node
- 2 worker nodes

**Container Runtime:** containerd 2.1.1

**Operating System:** Debian GNU/Linux 12 (bookworm)

**Kernel Version:** 6.14.0-1015-gcp

**Cluster Type:** kind (Kubernetes in Docker)

### Node Details

```mermaid
graph TB
    subgraph Cluster["Kubernetes Cluster v1.33.1"]
        CP[kind-control-plane<br/>172.18.0.4<br/>Control Plane]
        W1[kind-worker<br/>172.18.0.2<br/>Worker]
        W2[kind-worker2<br/>172.18.0.3<br/>Worker]
    end
    
    style CP fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style W1 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style W2 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
```

### Resource Summary

| Resource Type | Count |
|--------------|-------|
| Namespaces | 7 |
| Pods | 25 |
| Deployments | 6 |
| DaemonSets | 3 |
| Services | 16 |

### Namespaces

```mermaid
graph LR
    subgraph Namespaces
        NS1[default]
        NS2[kube-system]
        NS3[kube-node-lease]
        NS4[kube-public]
        NS5[local-path-storage]
        NS6[monitoring]
        NS7[cilium-secrets]
    end
    
    style NS2 fill:#fff3e0,stroke:#e65100
    style NS6 fill:#e8f5e9,stroke:#1b5e20
    style NS7 fill:#fce4ec,stroke:#880e4f
```

## Cilium and Hubble Configuration

### Cilium Installation

**Status:** ✅ Installed and Running

**Version:** 1.17.4

**Components:**
- Cilium Agent (DaemonSet): 3 pods running
- Cilium Envoy (DaemonSet): 3 pods running
- Cilium Operator (Deployment): 1 pod running

### Hubble Configuration

**Status:** ✅ Enabled and Operational

**Key Features:**
- Hubble metrics enabled
- Hubble Open Metrics enabled
- Current/Max Flows: 3433/4095 (83.83%)
- Flows/s: 12.86
- Metrics Status: Ok

**Hubble Metrics Exposed:**
- DNS metrics
- Drop metrics
- TCP metrics
- ICMP metrics
- Flow metrics with workload context

**Hubble Metrics Server:** Port 9965 (ClusterIP service: hubble-metrics)

**Note:** Hubble Relay is not deployed in this cluster. Metrics are exposed directly from Cilium agents.

```mermaid
graph TB
    subgraph CiliumArchitecture["Cilium & Hubble Architecture"]
        subgraph ControlPlane["Control Plane Node"]
            CA1[Cilium Agent]
            CE1[Cilium Envoy]
            CO[Cilium Operator]
        end
        
        subgraph Worker1["Worker Node 1"]
            CA2[Cilium Agent]
            CE2[Cilium Envoy]
        end
        
        subgraph Worker2["Worker Node 2"]
            CA3[Cilium Agent]
            CE3[Cilium Envoy]
        end
        
        subgraph HubbleMetrics["Hubble Metrics Collection"]
            HM[hubble-metrics Service<br/>Port 9965]
            HP[hubble-peer Service<br/>Port 443]
        end
        
        CA1 --> HM
        CA2 --> HM
        CA3 --> HM
        
        CA1 --> HP
        CA2 --> HP
        CA3 --> HP
    end
    
    style CA1 fill:#b3e5fc,stroke:#0277bd
    style CA2 fill:#b3e5fc,stroke:#0277bd
    style CA3 fill:#b3e5fc,stroke:#0277bd
    style CO fill:#c5e1a5,stroke:#558b2f
    style HM fill:#ffccbc,stroke:#d84315
    style HP fill:#ffccbc,stroke:#d84315
```

## Monitoring Stack

### Prometheus and Grafana Setup

**Status:** ✅ Fully Deployed

The cluster has a complete monitoring stack deployed in the `monitoring` namespace:

**Components:**

1. **Grafana**
   - Deployment: prometheus-grafana
   - Service: NodePort on port 32042
   - Pods: 1 running (3 containers)

2. **Prometheus**
   - Operator: prometheus-k8s-operator
   - Prometheus Server: prometheus-k8s-prometheus-0 (StatefulSet)
   - Service: prometheus-k8s-prometheus (Port 9090)

3. **Alertmanager**
   - StatefulSet: alertmanager-prometheus-k8s-alertmanager-0
   - Service: prometheus-k8s-alertmanager (Port 9093)

4. **Exporters**
   - Node Exporter (DaemonSet): 3 pods running
   - Kube State Metrics: 1 pod running

```mermaid
graph TB
    subgraph MonitoringStack["Monitoring Stack Architecture"]
        subgraph DataSources["Data Sources"]
            HM[Hubble Metrics<br/>:9965]
            NE[Node Exporters<br/>DaemonSet x3]
            KSM[Kube State Metrics]
            KL[Kubelet Metrics]
        end
        
        subgraph Collection["Metrics Collection"]
            PO[Prometheus Operator]
            PS[Prometheus Server<br/>:9090]
        end
        
        subgraph Visualization["Visualization & Alerting"]
            GF[Grafana<br/>NodePort :32042]
            AM[Alertmanager<br/>:9093]
        end
        
        HM --> PS
        NE --> PS
        KSM --> PS
        KL --> PS
        
        PO -.manages.-> PS
        PS --> GF
        PS --> AM
    end
    
    style HM fill:#ffccbc,stroke:#d84315,stroke-width:3px
    style PS fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style GF fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
    style AM fill:#ffcdd2,stroke:#c62828
```

## Golden Signals Monitoring

The cluster is configured to monitor the four Golden Signals through Hubble and Prometheus/Grafana:

```mermaid
graph LR
    subgraph GoldenSignals["Golden Signals"]
        L[Latency<br/>Response Times]
        T[Traffic<br/>Request Volume]
        E[Errors<br/>Failed Requests]
        S[Saturation<br/>Resource Usage]
    end
    
    subgraph HubbleMetrics["Hubble Metrics"]
        DNS[DNS Metrics]
        TCP[TCP Metrics]
        ICMP[ICMP Metrics]
        FLOW[Flow Metrics]
        DROP[Drop Metrics]
    end
    
    subgraph PrometheusMetrics["Prometheus Metrics"]
        NODE[Node Metrics]
        POD[Pod Metrics]
        KUBE[Kubernetes State]
    end
    
    DNS --> L
    TCP --> L
    FLOW --> T
    DROP --> E
    TCP --> E
    NODE --> S
    POD --> S
    KUBE --> S
    
    style L fill:#e1bee7,stroke:#6a1b9a
    style T fill:#b2dfdb,stroke:#00695c
    style E fill:#ffccbc,stroke:#d84315
    style S fill:#fff9c4,stroke:#f57f17
```

### 1. Latency

**Measured by:**
- Hubble TCP metrics (connection establishment, RTT)
- Hubble DNS metrics (query response times)
- Flow metrics with timing information

### 2. Traffic

**Measured by:**
- Hubble flow metrics (flows/second: 12.86)
- Flow context with workload names
- Source and destination identity tracking

### 3. Errors

**Measured by:**
- Hubble drop metrics (packet drops)
- DNS failure metrics
- TCP connection failures
- ICMP error messages

### 4. Saturation

**Measured by:**
- Hubble flow capacity: 3433/4095 (83.83% utilized)
- Node exporter metrics (CPU, memory, disk, network)
- Kube state metrics (pod resource requests/limits)

## Cluster Characteristics

### Key Distinguishing Features

1. **CNI Solution:** Cilium 1.17.4 with eBPF-based networking
   - Advanced network observability through Hubble
   - Envoy proxy integration for L7 visibility
   - Native support for network policies

2. **Observability Stack:** Complete monitoring solution
   - Prometheus Operator for metrics collection
   - Grafana for visualization (accessible via NodePort 32042)
   - Alertmanager for alert management
   - Hubble metrics integration for network observability

3. **Development Environment:** kind cluster
   - Lightweight, containerized Kubernetes
   - Suitable for local development and testing
   - Multi-node setup (1 control-plane + 2 workers)

4. **Modern Kubernetes:** Running v1.33.1
   - Latest stable release
   - containerd runtime
   - Debian-based nodes

5. **Network Observability:** Hubble enabled
   - Real-time flow monitoring
   - Protocol-level metrics (DNS, TCP, ICMP)
   - Workload identity tracking
   - High flow utilization (83.83%)

### Service Mesh Capabilities

```mermaid
graph TB
    subgraph ServiceMesh["Cilium Service Mesh Features"]
        subgraph L3L4["L3/L4 Features"]
            NP[Network Policies]
            LB[Load Balancing]
            SC[Service Discovery]
        end
        
        subgraph L7["L7 Features via Envoy"]
            HTTP[HTTP Filtering]
            GRPC[gRPC Support]
            TLS[TLS Termination]
        end
        
        subgraph Observability["Observability"]
            FM[Flow Monitoring]
            PM[Protocol Metrics]
            WI[Workload Identity]
        end
    end
    
    style NP fill:#e3f2fd,stroke:#1565c0
    style FM fill:#fff3e0,stroke:#e65100
    style HTTP fill:#f3e5f5,stroke:#6a1b9a
```

## Deployment Architecture

```mermaid
graph TB
    subgraph KubeSystem["kube-system Namespace"]
        API[kube-apiserver]
        SCHED[kube-scheduler]
        CM[kube-controller-manager]
        ETCD[etcd]
        DNS[CoreDNS x2]
        CILIUM[Cilium DaemonSet x3]
        ENVOY[Cilium Envoy x3]
        CILIUMOP[Cilium Operator]
    end
    
    subgraph Monitoring["monitoring Namespace"]
        GRAF[Grafana]
        PROM[Prometheus]
        ALERT[Alertmanager]
        NODEEXP[Node Exporter x3]
        PROMOP[Prometheus Operator]
        KSM2[Kube State Metrics]
    end
    
    subgraph Storage["local-path-storage Namespace"]
        LPP[Local Path Provisioner]
    end
    
    CILIUM -.exposes.-> HUBBLE[Hubble Metrics :9965]
    HUBBLE --> PROM
    NODEEXP --> PROM
    KSM2 --> PROM
    PROM --> GRAF
    PROM --> ALERT
    
    style CILIUM fill:#b3e5fc,stroke:#0277bd,stroke-width:2px
    style HUBBLE fill:#ffccbc,stroke:#d84315,stroke-width:2px
    style GRAF fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style PROM fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

## Access Information

### Grafana Dashboard
- **Service Type:** NodePort
- **Port:** 32042
- **Access:** `http://<node-ip>:32042`

### Prometheus
- **Service:** prometheus-k8s-prometheus
- **Port:** 9090
- **Type:** ClusterIP

### Hubble Metrics
- **Service:** hubble-metrics
- **Port:** 9965
- **Type:** ClusterIP (headless)

## Summary

This is a well-configured Kubernetes cluster optimized for observability and network monitoring:

- **3-node kind cluster** running Kubernetes v1.33.1
- **Cilium 1.17.4** providing advanced networking with eBPF
- **Hubble enabled** for deep network observability with 83.83% flow utilization
- **Complete Prometheus/Grafana stack** for metrics collection and visualization
- **Golden Signals monitoring** through Hubble metrics (latency, traffic, errors, saturation)
- **25 pods** across 7 namespaces supporting core services and monitoring
- **NodePort access** to Grafana on port 32042 for easy dashboard access

The cluster is production-ready from an observability standpoint, with comprehensive monitoring of both infrastructure and network layers.
