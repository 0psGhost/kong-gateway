# Module 9: Production Architecture

[Previous: Module 8](../module-08-consumers/README.md) | [Course home](../README.md) | [Next: Module 10](../module-10-troubleshooting/README.md)

Minikube, NodePorts, local rate-limit storage, and generated demo credentials are learning conveniences, not production defaults.

## Reference Shape

```text
clients
  |
DNS + TLS + cloud load balancer
  |
Kong data-plane replicas across zones
  |                  ^
upstream services   control plane / config pipeline
                     |
          PostgreSQL or DB-less declarative config
```

## Production Checklist

- Run multiple Kong replicas across zones with PodDisruptionBudgets, anti-affinity, and autoscaling.
- Manage TLS certificates through a controlled workflow.
- Use network policies, least-privilege RBAC, image pinning, and a secret manager.
- Use a tested configuration pipeline, staging environment, and rollback process.
- Monitor latency, error rate, saturation, and `429` responses with metrics, logs, traces, dashboards, and alerts.
- Use PostgreSQL HA in traditional mode, or carefully versioned declarative configuration in DB-less mode.
- Set explicit timeouts, retries, upstream health checks, and capacity limits.
- Back up control-plane state and exercise disaster recovery.
- Keep the Admin API private and protected.

## Design Questions

Before production, decide who owns gateway configuration, how changes are reviewed, where credentials live, how keys are rotated, and what happens when Kong or an upstream is unavailable.

[Previous: Module 8](../module-08-consumers/README.md) | [Course home](../README.md) | [Next: Module 10](../module-10-troubleshooting/README.md)
