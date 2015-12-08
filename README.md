BOSH release for bosh-init-deployment-resource
=======================

The bosh-init-deployment-resource is build to trigger bosh-init deployments from concourse. The bosh-init-deployment-resource-boshrelease bundles the resource in a bosh release. The usage can be found in the [bosh-init-deployment-resource](https://github.com/hybris/bosh-init-deployment-resource) repository.

Usage
-----

To use this bosh release, first upload it to the BOSH/bosh-lite that is running Concourse:

```
bosh upload release https://github.com/hybris/bosh-init-deployment-resource-boshrelease
```

Next, update your Concourse deployment manifest to add the resource.

Add the `bosh-init-deployment-resource-boshrelease` release to the list:

```yaml
releases:
  - name: concourse
    version: latest
  - name: garden-linux
    version: latest
  - name: bosh-init-deployment-resource
    version: latest
```

Into the `worker` job, add the `{release: bosh-init-deployment-resource, name: bosh-init}` job template that will install the package:

```yaml
jobs:
- name: worker
  templates:
    ...
    - {release: bosh-init-deployment-resource, name: bosh-init}
```

The final change is to explicitly list all the resource types (they are implicit) and add the `bosh-init-deployment-resource` package to the list:

```yaml
jobs:
- name: worker
  ...
  properties:
    groundcrew:
      resource_types:
      ...
      - type: bosh-init-deployment
        image: /var/vcap/packages/bosh-init
```

Note that it is the latter two lines that are specific to this BOSH release:

```yaml
- type: bosh-init-deployment
  image: /var/vcap/packages/bosh-init
```

The former lines should be obtained from the Concourse BOSH release, not the documentation above which might be out of date. Use https://github.com/concourse/concourse/blob/master/jobs/groundcrew/spec#L69-L96

And `bosh deploy` your Concourse manifest.

Setup pipeline in Concourse
---------------------------

```
fly -t snw c -c pipeline.yml --vars-from credentials.yml bosh-init-deployment-resource-boshrelease
```
