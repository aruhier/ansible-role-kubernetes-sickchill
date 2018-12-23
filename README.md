Ansible Role: sickchill for Kubernetes
=====================================

Ansible role to install [sickchill](https://sickchill.github.io/) on Kubernetes.

Role Variables
--------------

```yaml
# Image used
kubernetes_sickchill_image: "linuxserver/sickchill:latest"

# Namespace
kubernetes_sickchill_namespace: "default"
# App name (used as selector)
kubernetes_sickchill_app: "sickchill"
# Deployment name
kubernetes_sickchill_deployment: "sickchill-deployment"
# Service name
kubernetes_sickchill_service: "sickchill"

# Number of replicas
kubernetes_sickchill_replicas: 1
kubernetes_sickchill_revision_history: 1

# Node selector
kubernetes_sickchill_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_sickchill_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_sickchill_deployment_annotations: {}

kubernetes_sickchill_resources:
  limits:
    memory: "756Mi"
  requests:
    memory: "256Mi"

# Setup ingress for sickchill
kubernetes_sickchill_setup_ingress: false
kubernetes_sickchill_ingress:
  name: "sickchill-ingress"
  host: "sickchill.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /downloads/ (see examples for more details)
kubernetes_sickchill_downloads_volumes: {}
# TV/series volumes. Mounted in /tv/ (see examples for more details)
kubernetes_sickchill_tv_volumes: {}
# Watch directories. Useful when blackhole is used. Mounted in /watchdirs/ (see
# examples for more details)
kubernetes_sickchill_watchdirs_volumes: {}

# sickchill config volume. Contains the database, arts cache and config.
kubernetes_sickchill_config_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_sickchill_downloads_volumes:
      # This volume will be mounted as /downloads/completed
      completed:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-torrents
            readOnly: false
        subPath: "completed"

    kubernetes_sickchill_tv_volumes:
      # This volume will be mounted as /tv/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-tv
            readOnly: false

    # Directories watched by our torrents client
    kubernetes_sickchill_watchdirs_volumes:
      # This volume will be mounted as /watchdirs/providers
      providers:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-watchdirs
            readOnly: false
        subPath: "example/providers"

    kubernetes_sickchill_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-sickchill
          readOnly: false
        subPath: "config"

    kubernetes_sickchill_setup_ingress: true
    kubernetes_sickchill_ingress:
      name: "sickchill-ingress"
      host: "sickchill.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "sickchill-ingress-tls"
          hosts:
            - "sickchill.example.com"
  roles:
    - role: Anthony25.kubernetes_sickchill
```

Use `run_once` to run the role on only one available master in the cluster.
If the sickchill web interface is not reachable, please check that it is
listening on `0.0.0.0:5050`, and not only `localhost:5050` as the default is.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
