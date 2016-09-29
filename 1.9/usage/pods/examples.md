---
post_title: Pods CLI
menu_order: 30
---

# A pod with multiple containers
	
The following pod definition defines a pod with 3 containers.

```json
{
  "id": "/pod-with-multiple-containers",
  "labels": {
    "values": {}
  },
  "version": "2016-09-22T09:18:05.928Z",
  "user": null,
  "environment": null,
  "containers": [
    {
      "name": "sleep1",
      "exec": {
        "command": {
          "shell": "sleep 1000"
        }
      },
      "resources": {
        "cpus": 0.1,
        "mem": 32,
        "disk": 0,
        "gpus": 0
      },
      "endpoints": [],
      "image": null,
      "environment": null,
      "user": null,
      "healthCheck": null,
      "volumeMounts": [],
      "artifacts": [],
      "labels": null,
      "lifecycle": null
    },
    {
      "name": "sleep2",
      "exec": {
        "command": {
          "shell": "sleep 1000"
        }
      },
      "resources": {
        "cpus": 0.1,
        "mem": 32,
        "disk": 0,
        "gpus": 0
      },
      "endpoints": [],
      "image": null,
      "environment": null,
      "user": null,
      "healthCheck": null,
      "volumeMounts": [],
      "artifacts": [],
      "labels": null,
      "lifecycle": null
    },
    {
      "name": "sleep3",
      "exec": {
        "command": {
          "shell": "sleep 1000"
        }
      },
      "resources": {
        "cpus": 0.1,
        "mem": 32,
        "disk": 0,
        "gpus": 0
      },
      "endpoints": [],
      "image": null,
      "environment": null,
      "user": null,
      "healthCheck": null,
      "volumeMounts": [],
      "artifacts": [],
      "labels": null,
      "lifecycle": null
    }
  ],
  "secrets": null,
  "volumes": [],
  "networks": [
    {
      "mode": "host"
    }
  ],
  "scaling": {
    "kind": "fixed",
    "instances": 10,
    "maxInstances": null
  },
  "scheduling": {
    "backoff": {
      "backoff": 1,
      "backoffFactor": 1.15,
      "maxLaunchDelay": 3600
    },
    "upgrade": {
      "minimumHealthCapacity": 1,
      "maximumOverCapacity": 1
    },
    "placement": {
      "constraints": [],
      "acceptedResourceRoles": []
    }
  }
}
```

# A Pod that Uses Ephemeral Volumes

```json
{
  "id": "/with-ephemeral-vol",
  "scaling": { "kind": "fixed", "instances": 1 },
  "containers": [
    {
      "name": "ct1",
      "resources": {
        "cpus": 0.1,
        "mem": 32
      },
      "exec": { "command": { "shell": "while true; do echo the current time is $(date) > ./jdef-v1/clock; sleep 1; done" } },
      "volumeMounts": [
        {
          "name": "v1",
          "mountPath": "jdef-v1"
        }
      ]
    },
    {
      "name": "ct2",
      "resources": {
        "cpus": 0.1,
        "mem": 32
      },
      "exec": { "command": { "shell": "while true; do cat ./etc/clock; sleep 1; done" } },
      "volumeMounts": [
        {
          "name": "v1",
          "mountPath": "etc"
        }
      ]
    }
  ],
  "networks": [
    { "mode": "host" }
  ],
  "volumes": [
    { "name": "v1" }
  ]
}
```

## IP-per-Pod Networking

This pod declares a virtual (user) network named `my-virtual-network-name`.

```json
{
  "id": "/pod-with-virtual-network",
  "scaling": { "kind": "fixed", "instances": 1 },
  "containers": [
    {
      "name": "sleep1",
      "exec": { "command": { "shell": "sleep 1000" } },
      "resources": { "cpus": 0.1, "mem": 32 }
    }
  ],
  "networks": [ { "mode": "container", "name": "my-virtual-network-name" } ]
}
```

This pod declares a “web” endpoint that listens on port 80.

```json
{
  "id": "/pod-with-endpoint",
  "scaling": { "kind": "fixed", "instances": 1 },
  "containers": [
    {
      "name": "sleep1",
      "exec": { "command": { "shell": "sleep 1000" } },
      "resources": { "cpus": 0.1, "mem": 32 },
      "endpoints": [ { "name": "web", "containerPort": 80 } ]
    }
  ],
  "networks": [ { "mode": "container", "name": "my-virtual-network-name" } ]
}
```

This pod adds a health check that references the “web” endpoint; mesos will execute an HTTP request against `http://<ip-address-of-the-pod>:80/ping`. 

```json
{
  "id": "/pod-with-healthcheck",
  "scaling": { "kind": "fixed", "instances": 1 },
  "containers": [
    {
      "name": "sleep1",
      "exec": { "command": { "shell": "sleep 1000" } },
      "resources": { "cpus": 0.1, "mem": 32 },
      "endpoints": [ { "name": "web", "containerPort": 80 } ],
      "healthCheck": { "http": { "endpoint": "web", "path": "/ping" } }
    }
  ],
  "networks": [ { "mode": "container", "name": "my-virtual-network-name" } ]
}
```

# Complete Pod 
The following pod definition can serve as a reference to create more complicated pods.

```
{
  "id": "/complete-pod",
  "labels": {
    "owner": "zeus",
    "note": "Away from olympus"
  },
  "environment": {
    "XPS1": "Test"
  },
  "volumes": [
    {
      "name": "etc",
      "host": "/hdd/tools/docker/registry"
    }
  ],
  "networks": [
    {
      "mode": "host"
    }
  ],
  "scaling": {
    "kind": "fixed",
    "instances": 1
  },
  "scheduling": {
    "backoff": {
      "backoff": 1,
      "backoffFactor": 1.15,
      "maxLaunchDelay": 3600
    },
    "upgrade": {
      "minimumHealthCapacity": 1,
      "maximumOverCapacity": 1
    },
    "placement": {
      "constraints": [],
      "acceptedResourceRoles": []
    }
  },
  "containers": [
    {
      "name": "container1",
      "exec": {
        "command": {
          "shell": "sleep 100"
        },
        "overrideEntrypoint": false
      },
      "resources": {
        "cpus": 1,
        "mem": 128,
        "disk": 0,
        "gpus": 0
      },
      "endpoints": [
        {
          "name": "foo",
          "containerPort": 0,
          "hostPort": 0,
          "protocol": "tcp",
          "labels": {}
        }
      ],
      "image": {
        "id": "mesosphere/marathon:latest",
        "kind": "DOCKER",
        "forcePull": false
      },
      "environment": {
        "XPS1": "Test"
      },
      "user": "root",
      "healthCheck": {
        "gracePeriodSeconds": 30,
        "intervalSeconds": 2,
        "maxConsecutiveFailures": 3,
        "timeoutSeconds": 20,
        "delaySeconds": 2,
        "http": {
          "path": "/health",
          "scheme": "HTTP",
          "endpoint": "httpEndpoint"
        }
      },
      "volumeMounts": [
        {
          "name": "env",
          "mountPath": "/mnt/etc",
          "readOnly": true
        }
      ],
      "artifacts": [
        {
          "uri": "https://foo.com/archive.zip",
          "executable": false,
          "extract": true,
          "cache": true,
          "destPath": "newname.zip"
        }
      ],
      "labels": {
        "owner": "zeus",
        "note": "Away from olympus"
      },
      "lifecycle": {
        "killGracePeriodSeconds": 60
      }
    }
  ]
}
```