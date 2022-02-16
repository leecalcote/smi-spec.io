---
title: "Pipelining Service Mesh Specifications"
slug: "pipelining-service-mesh-specifications"
authorname: "Lee Calcote"
author: "@lcalcote"
authorlink: "https://twitter.com/lcalcote"
date: "2022-02-17T05:00:00-13:00"
---
![Performance Management Dashboard](/img/blog/pipelining-service-mesh-specifications/service-mesh-specifications.png#left)
With growing adoption of service meshes in cloud native environments, service mesh abstractions and service mesh-neutral specifications have emerged. [Service Mesh Performance](https://smp-spec.io/) and Service Mesh Interface are two open specifications that address the need for universal interfaces for interacting with and managing any type of service mesh. What do each of these specification provide?

Service Mesh Performance standardizes service mesh value measurement, characterizing any deployment's performance by capturing the details of infrastructure capacity, service mesh configuration and workload metadata.

Service Mesh Interface provides a standard interface for service meshes on Kubernetes. These (currently) four specfications offer a common denominator set of interfaces to support most common service mesh use cases and the flexibility to evolve to support new service mesh capabilities over time.

As a service mesh agnostic tool that provides lifecycle and performance management of a large number of [(10+) service meshes](https://layer5.io/service-mesh-landscape), Kubernetes applications, service mesh patterns and WebAssembly filters, [Meshery](https://meshery.io/) is the ideal tool for the job when it comes to implementing these specifications.

Meshery also comes with two new GitHub Actions make it easy to integrate SMI and SMP into your GitHub workflows. The [Meshery SMI Conformance Action](https://github.com/layer5io/meshery-smi-conformance-action) validates the adherance of your service mesh deployment against [SMI's spcdifications](https://meshery.io/blog/validating-smi-conformance-with-meshery) using a standard set of conformance tests. Output of conformance tests are presented on the SMI Conformance Dashboard(https://meshery.io/service-mesh-interface). The [Meshery SMP Action](https://github.com/layer5io/meshery-smp-action) integrates into your application pipeline, performing [SMP compatible performance benchmarks](https://docs.meshery.io/functionality/performance-management) within your environment and in accordance with your load requirements and service mesh configuration.

Let's take a closer look at each of these Actions.

#### Service Mesh Interface Conformance GitHub Action

Conformance of SMI specifications is defined as a series of test assertions. These test assertions are categorised by SMI specification (of which, there are currently four specifications) and comprise the complete suite of SMI conformance tests. Conformance requirements will change appropriately as each new version of the SMI spec is released. Refer to Meshery's documentation for details of how [Meshery performs SMI conformance.](https://docs.meshery.io/functionality/service-mesh-interface)

**Using Meshery's SMI Conformance GitHub Action** 

The [Service Mesh Interface Conformance GitHub Action](https://github.com/marketplace/actions/service-mesh-interface-conformance-with-meshery) is available in the GitHub Marketplace. You can configure this action to trigger with each of your releases, on every pull request. or any GitHub workflow trigger event.

An example of the action configuration which runs on every release is shown below. The action handles setting up a Kubernetes environment, deploying the service mesh (see supported service meshes), running the conformance tests and reporting back the results to the [SMI Conformance dashboard](https://layer5.io/service-mesh-landscape#smi) in Meshery.

<div class="codewrapper">
  <pre><code>
 name: SMI Conformance with Meshery
 on:
   push:
     tags:
       - 'v*'

 jobs:
   smi-conformance:
     name: SMI Conformance
     runs-on: ubuntu-latest
     steps:

       - name: SMI conformance tests
         uses: layer5io/mesheryctl-smi-conformance-action@master
         with:
           provider_token: ${{ secrets.MESHERY_PROVIDER_TOKEN }}
           service_mesh: open_service_mesh
           mesh_deployed: false
   </code></pre></div>

You can also bring in your own cluster with specific capabilities and with a service mesh already installed.

<div class="codewrapper">
  <pre><code>
name: SMI Conformance with Meshery
on:
  push:
    branches:
      - 'master'

jobs:
  smi-conformance:
    name: SMI Conformance tests on master
    runs-on: ubuntu-latest
    steps:

      - name: Deploy k8s-minikube
        uses: manusa/actions-setup-minikube@v2.4.1
        with:
          minikube version: 'v1.21.0'
          kubernetes version: 'v1.20.7'
          driver: docker

      - name: Install OSM
        run: |
           curl -LO https://github.com/openservicemesh/
           osm/releases/download/v0.9.1/osm-v0.9.1-linux-amd64.tar.gz
           tar -xzf osm-v0.9.1-linux-amd64.tar.gz
           mkdir -p ~/osm/bin
           mv ./linux-amd64/osm ~/osm/bin/osm-bin
           PATH="$PATH:$HOME/osm/bin/"
           osm-bin install --osm-namespace default

      - name: SMI conformance tests
        uses: layer5io/mesheryctl-smi-conformance-action@master
        with:
          provider_token: ${{ secrets.MESHERY_PROVIDER_TOKEN }}
          service_mesh: open_service_mesh
          mesh_deployed: true
   </code></pre></div>

You can download a token from Meshery and add it as a GitHub secret (in the example above, the secret is MESHERY_PROVIDER_TOKEN). After the test is run, you can view the results from the Service Mesh Interface dashboard in Meshery UI.

![SMI Conformance Dashboard](/img/blog/pipelining-service-mesh-specifications/smi-conformance-result.png)

Participating service mesh projects can also [automatically report their conformance test results](https://docs.meshery.io/functionality/service-mesh-interface#reporting-conformance) to the [SMI Conformance dashboard](https://meshery.io/service-mesh-interface).

#### Service Mesh Performance GitHub Action

Measuring and managing the performance of a service mesh is key to efficient operation of any service mesh. [Meshery](https://meshery.io/) is the canonical implementation of the Service Mesh Performance specification. You can choose from multiple load generators and use a highly configurable set of load profiles with variable tunable facets to run a performance test. Meshery packages all these features into an easy-to-use GitHub Action.

**Using Meshery's Service Mesh Performance GitHub Action**

The [Service Mesh Performance GitHub Action](https://github.com/marketplace/actions/performance-testing-with-meshery) is available in the GitHub Marketplace.You can create your own performance profiles to run repeatable tests with Meshery. You can configure this action to trigger with each of your releases, on every pull request. or any GitHub workflow trigger event. A sample configuration of the action is shown below.

<div class="codewrapper">
  <pre><code>
 name: Meshery SMP Action
on:
  push:
    branches:
      'master'

jobs:
  performance-test:
    name: Performance Test
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v2
        with:
          ref: 'perf'

      - name: Deploy k8s-minikube
        uses: manusa/actions-setup-minikube@v2.4.1
        with:
          minikube version: 'v1.21.0'
          kubernetes version: 'v1.20.7'
          driver: docker

      - name: Run Performance Test
        uses: layer5io/meshery-smp-action@master
        with:
          provider_token: ${{ secrets.PROVIDER_TOKEN }}
          platform: docker
          profile_name: soak-test
   </code></pre></div>

You can also define your test configuration in an SMP compatible configuration file as shown below.

<div class="codewrapper">
  <pre><code>
smp_version: v0.0.1
id:
name: Istio Performance Test
labels: {}
clients:
- internal: false
  load_generator: fortio
  protocol: 1
  connections: 2
  rps: 10
  headers: {}
  cookies: {}
  body: ""
  content_type: ""
  endpoint_urls:
  - http://localhost:2323/productpage
duration: "30m"
 </code></pre></div>

See this sample GitHub workflow ([action.yml](https://github.com/layer5io/meshery-smp-action/blob/master/action.yml)) for more configuration details.

![Performance Management Dashboard](/img/blog/pipelining-service-mesh-specifications/service-mesh-performance-profile-test-results.png)

The results from the tests are updated on the Performance Management dashboard in Meshery. To learn more about interpreting the test results, check out [this guide](https://docs.meshery.io/guides/interpreting-performance-test-results). You can always checkout the [Meshery User Guides](https://docs.meshery.io/guides) to dive deep into these features.

