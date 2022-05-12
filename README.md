Ansible Role: rTorrent for Kubernetes
=====================================

**UNMAINTAINED, I DON'T USE IT ANYMORE.**

Ansible role to install [rTorrent](https://github.com/rakshasa/rtorrent) and
[ruTorrent](https://github.com/Novik/ruTorrent) on Kubernetes.

Role Variables
--------------

```yaml
# Namespace
kubernetes_rtorrent_namespace: "default"
# App name (used as selector)
kubernetes_rtorrent_app: "rtorrent"
# Deployment name
kubernetes_rtorrent_deployment: "rtorrent-deployment"
# Configmap name
kubernetes_rtorrent_configmap: "rtorrent"
# Service name
kubernetes_rtorrent_service: "rtorrent"

# Number of replicas
kubernetes_rtorrent_replicas: 1
kubernetes_rtorrent_revision_history: 1

# Node selector
kubernetes_rtorrent_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_rtorrent_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_rtorrent_deployment_annotations: {}

kubernetes_rtorrent_resources:
  limits:
    memory: "1.5Gi"
  requests:
    memory: "256Mi"

# Setup ingress for rtorrent
kubernetes_rtorrent_setup_ingress: false
kubernetes_rtorrent_ingress:
  name: "rtorrent-ingress"
  host: "rtorrent.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

## Required volumes ##

# rTorrent session volume
kubernetes_rtorrent_session_volume:
  definition:

## Optional volumes ##

# Next volumes are optional, they are used as download targets or watch
# directories.

# rTorrent download directories
kubernetes_rtorrent_downloads_volumes: {}

# rTorrent watch directories
kubernetes_rtorrent_watchdirs_volumes: {}

### rTorrent options ###

kubernetes_rtorrent_min_peers: 40
kubernetes_rtorrent_max_peers: 100
kubernetes_rtorrent_min_peers_seed: 10
kubernetes_rtorrent_max_peers_seed: 50
kubernetes_rtorrent_max_uploads: 15
kubernetes_rtorrent_max_global_downloads: 300
kubernetes_rtorrent_max_global_uploads: 300
kubernetes_rtorrent_download_rate: 0
kubernetes_rtorrent_upload_rate: 0
kubernetes_rtorrent_numwant: -1

# ----- directory settings -----
kubernetes_rtorrent_basedir: ""
kubernetes_rtorrent_directory_download: "{{ kubernetes_rtorrent_basedir }}/downloads/global"
kubernetes_rtorrent_directory_session: "/config/rtorrent/kubernetes_rtorrent_sess"

# directory_watch can be a directory path (like the default value), a dict or
# a list of dicts for multiple watch directories.
# Example for one watch directory:
#   kubernetes_rtorrent_directory_watch:
#     path: ~/rtorrent/watch
#     refresh_time: 10
#     download: ~/rtorrent/download_complete
# Example for multiple watch directories:
#   kubernetes_rtorrent_directory_watch:
#     - ~/rtorrent/watch
#     - path: ~/rtorrent/watch1
#       refresh_time: 30
#     - path: ~/rtorrent/watch2
#       download: ~/rtorrent/download_custom
# When not set, refresh_time={{ kubernetes_rtorrent_global_refresh_time }} and
# download={{ kubernetes_rtorrent_directory_download }}
kubernetes_rtorrent_directory_watch: "{{ kubernetes_rtorrent_basedir }}/watchdirs/global"

# Move completed torrents to the download directory as set in their watch
# directory
kubernetes_rtorrent_move_when_complete: false

# ----- network settings -----
kubernetes_rtorrent_port_range: 49164-49164
kubernetes_rtorrent_port_random: "no"
kubernetes_rtorrent_ssl_verify_peer: 0
kubernetes_rtorrent_max_open_sockets: 999
kubernetes_rtorrent_max_open_files: 1024
kubernetes_rtorrent_max_open_http: 32
kubernetes_rtorrent_receive_buffer_size: 0
kubernetes_rtorrent_send_buffer_size: 0
kubernetes_rtorrent_dns_cache_timeout: 60
kubernetes_rtorrent_xmlrpc_size_limit: 524288

# ----- schedule settings -----
# Set refresh time for untied_directory and watch directories
kubernetes_rtorrent_global_refresh_time: 5
kubernetes_rtorrent_schedule_untied_directory: >
    untied_directory,{{ kubernetes_rtorrent_global_refresh_time }},
    {{ kubernetes_rtorrent_global_refresh_time }},stop_untied=
# ----- torrent settings -----
kubernetes_rtorrent_check_hash: "no"
kubernetes_rtorrent_use_udp_trackers: "yes"
kubernetes_rtorrent_tracker_numwant: -1
kubernetes_rtorrent_encryption: "allow_incoming,enable_retry,prefer_plaintext"
kubernetes_rtorrent_dht: "off"
kubernetes_rtorrent_dht_port: 6881
kubernetes_rtorrent_peer_exchange: "yes"

# ----- memory and pieces buffers -----
kubernetes_rtorrent_max_memory_usage: "1024M"
kubernetes_rtorrent_pieces_preload_type: 0
kubernetes_rtorrent_pieces_preload_min_size: 262144
kubernetes_rtorrent_pieces_preload_min_rate: 5120

# ----- other settings -----
#  (can break rtorrent when misconfigured)
kubernetes_rtorrent_file_allocate: 0
# Delete data of erased torrents
kubernetes_rtorrent_delete_erased: false
kubernetes_rtorrent_other_settings: []
# Example
#   - scgi_port = 127.0.0.1:5000
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
    kubernetes_rtorrent_downloads_volumes:
      # This volume will be mounted as /downloads/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-downloads-global
            readOnly: false
      # This volume will be mounted as /downloads/tv
      tv:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "tv"
      # This volume will be mounted as /downloads/series
      series:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "series"

    kubernetes_rtorrent_session_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-rtorrent
          readOnly: false
        subPath: "session"

    kubernetes_rtorrent_setup_ingress: true
    kubernetes_rtorrent_ingress:
      name: "rtorrent-ingress"
      host: "rtorrent.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "rtorrent-ingress-tls"
          hosts:
            - "rtorrent.example.com"

  roles:
    - role: Anthony25.kubernetes-rtorrent
```

Use `run_once` to run the role on only one available master in the cluster.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
