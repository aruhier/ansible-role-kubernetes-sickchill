Ansible Role: Sickrage for Kubernetes
=====================================

Ansible role to install [Sickrage](https://sickrage.github.io/) on Kubernetes.

Role Variables
--------------

```yaml
# Image used
kubernetes_sickrage_image: "linuxserver/sickrage:latest"

# Namespace
kubernetes_sickrage_namespace: "default"
# App name (used as selector)
kubernetes_sickrage_app: "sickrage"
# Deployment name
kubernetes_sickrage_deployment: "sickrage-deployment"
# Service name
kubernetes_sickrage_service: "sickrage"

# Number of replicas
kubernetes_sickrage_replicas: 1
kubernetes_sickrage_revision_history: 1

# Node selector
kubernetes_sickrage_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_sickrage_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_sickrage_deployment_annotations: {}

kubernetes_sickrage_resources:
  limits:
    memory: "756Mi"
  requests:
    memory: "256Mi"

# Setup ingress for sickrage
kubernetes_sickrage_setup_ingress: false
kubernetes_sickrage_ingress:
  name: "sickrage-ingress"
  host: "sickrage.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /downloads/ (see examples for more details)
kubernetes_sickrage_downloads_volumes: {}
# TV/series volumes. Mounted in /tv/ (see examples for more details)
kubernetes_sickrage_tv_volumes: {}
# Watch directories. Useful when blackhole is used. Mounted in /watchdirs/ (see
# examples for more details)
kubernetes_sickrage_watchdirs_volumes: {}

# sickrage config volume. Contains the database, arts cache and config.
kubernetes_sickrage_config_volume:
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
    kubernetes_sickrage_downloads_volumes:
      # This volume will be mounted as /downloads/completed
      completed:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-torrents
            readOnly: false
        subPath: "completed"

    kubernetes_sickrage_tv_volumes:
      # This volume will be mounted as /tv/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-tv
            readOnly: false

    # Directories watched by our torrents client
    kubernetes_sickrage_watchdirs_volumes:
      # This volume will be mounted as /watchdirs/providers
      providers:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-watchdirs
            readOnly: false
        subPath: "example/providers"

    kubernetes_sickrage_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-sickrage
          readOnly: false
        subPath: "config"

    kubernetes_sickrage_setup_ingress: true
    kubernetes_sickrage_ingress:
      name: "sickrage-ingress"
      host: "sickrage.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "sickrage-ingress-tls"
          hosts:
            - "sickrage.example.com"
  roles:
    - role: Anthony25.kubernetes-sickrage
```

Use `run_once` to run the role on only one available master in the cluster.
If the Sickrage web interface is not reachable, please check that it is
listening on `0.0.0.0:5050`, and not only `localhost:5050` as the default is.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
