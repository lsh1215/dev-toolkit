---
name: diagram-infra
description: Generate infrastructure diagrams using Python Diagrams library (mingrammer/diagrams).
triggers:
  - "인프라 구성도"
  - "인프라 다이어그램"
  - "infrastructure diagram"
  - "infra diagram"
  - "배포 구성도"
  - "deployment diagram"
  - "cloud architecture"
  - "클라우드 아키텍처"
  - "docker 구성도"
  - "docker diagram"
---

## Infrastructure Diagram Skill (Global)

**MANDATORY**: When this skill is triggered, you MUST use the Python `diagrams` library (mingrammer/diagrams) to generate infrastructure diagrams.
Do NOT use Excalidraw, Mermaid, or ASCII art for infra diagrams.

### Setup

```bash
pip install diagrams
```

Requires Graphviz installed (`brew install graphviz` on macOS).

### Output

- Generate a Python script (`{name}_diagram.py`) that produces a PNG/SVG
- Save the script in the project's `docs/` or `infra/` directory
- Run the script to generate the image file

### Code Conventions

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.onprem.database import MySQL
from diagrams.onprem.queue import Kafka
from diagrams.onprem.monitoring import Prometheus, Grafana
from diagrams.onprem.network import Nginx
from diagrams.programming.framework import Spring
from diagrams.elastic.elasticsearch import Elasticsearch
from diagrams.onprem.container import Docker
```

### Style Guide

- Use `Cluster` to group related components (e.g., "Order Service", "Monitoring Stack")
- Use `Edge` with labels for protocols and data flow descriptions
- Direction: `direction="LR"` for horizontal, `direction="TB"` for vertical
- Graph attributes: `graph_attr={"fontsize": "20", "bgcolor": "white"}`
- Use descriptive node names that match actual service names
- Output format: PNG for docs, SVG for scalable usage

### Example Structure

```python
with Diagram("E-Commerce Platform", show=False, direction="LR"):
    with Cluster("API Layer"):
        gateway = Nginx("API Gateway")

    with Cluster("Services"):
        order = Spring("Order Service")
        payment = Spring("Payment Service")

    with Cluster("Data"):
        db = MySQL("MySQL")
        queue = Kafka("Kafka")

    gateway >> order >> db
    order >> Edge(label="async") >> queue >> payment
```

### Exception

If `diagrams` library cannot be installed or Graphviz is unavailable,
fall back to Excalidraw with infrastructure-style icons and layout.
