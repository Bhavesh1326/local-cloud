# Infrastructure as Code

This directory contains all the infrastructure components managed by Argo CD.

## Structure

- rgocd/ - Argo CD bootstrap and configuration
  - ootstrap/ - Initial Argo CD installation
  - projects/ - Argo CD project definitions
  - pplications/ - Application definitions
- istio/ - Service mesh configuration
  - ase/ - Base Istio installation
  - gateway/ - Gateway and VirtualService configurations
- monitoring/ - Observability stack
  - prometheus/ - Prometheus configuration
  - grafana/ - Grafana dashboards
  - loki/ - Log aggregation
  - 	empo/ - Distributed tracing
- security/ - Security configurations
  - policies/ - OPA/Gatekeeper policies
  - 
etwork-policies/ - Network policies
