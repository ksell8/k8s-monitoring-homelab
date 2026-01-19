# Who is this for?

This project is meant for folks who want to get acclimated with GitOps, but who don't want to pay a cloud provider to maintain a control plane--yet.  We will use kind (kubernetes in docker) to run this.

Kubernetes can be big and scary.  Hell, I'm scared of it, and I've worked with it.  Control planes are so hard to manage only cloud providers dare support it at scale.  This is my attempt to make it less scary.  For myself and for you dear reader.  

Mistakes are good.  If you recognize them it means you are learning.

# How much can I run in this setup?

Not a lot.  It depends on your machine.  This is meant to allow you to quickly bootstrap experiments. 

Each experiment will have a CPU and RAM estimate.  Plan your experimental kustomizations accordingly.  

The control plane itself will take 2-4GB RAM and 1-2 cores in the worst case.  And it grows as you add resources.

Required:

1) [K8s Tools](./docs/k8s-tools.md)
2) [Kind Setup](./docs/kind.md)
3) [Flux Setup](./docs/flux.md)

Experimental:

- [Metrics Server](./docs/metrics-server.md)
- [Kube Prometheus Stack](./docs/kube-prometheus-stack.md)

