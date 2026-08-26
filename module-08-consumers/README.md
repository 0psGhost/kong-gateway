# Module 8: Consumer Management

[Previous: Module 7](../module-07-authentication/README.md) | [Course home](../README.md) | [Next: Module 9](../module-09-production-architecture/README.md)

Consumers represent clients, teams, or applications rather than human end users. Credentials identify a Consumer; plugins define what that Consumer may do.

## Inspect and Manage Consumers

```bash
kubectl get kongconsumers,secrets,kongplugins -n kong
kubectl describe kongconsumer learner -n kong
kubectl label kongconsumer learner -n kong tier=trusted
kubectl delete kongconsumer learner -n kong
```

A Consumer-specific plugin can be associated with a Consumer using its annotation or `consumerRef`, depending on the KIC version and policy you need. Keep global policies on the route and client-specific quotas on Consumers.

Use separate Consumers for separate applications so revocation and auditing remain precise. Keep credentials out of Git and rotate them through a controlled process.

## Checkpoint

Create one Consumer per client application, assign a credential Secret, and explain which plugin controls authentication versus authorization or quota.

[Previous: Module 7](../module-07-authentication/README.md) | [Course home](../README.md) | [Next: Module 9](../module-09-production-architecture/README.md)
