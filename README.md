# CoreKube: An Efficient, Autoscaling and Resilient Mobile Core System

CoreKube is a novel message-focused and cloud-native mobile core system design. In light of the central role that the mobile core has in supporting mobile network operations spanning billions of user and IoT devices, efficiency, cost-effective dynamic scalability and resilience of the core control plane are paramount. It was with these goals that the CoreKube design was developed. Orchestration of containerized CoreKube components using Kubernetes, allowing to leverage the its autoscaling and self-healing properties. CoreKube complies with the 3GPP RAN-core standards, allowing commercial-off-the-shelf (COTS) devices to connect to it.

## Components

All of the components that make up CoreKube are open-source. They can be accessed through the following links:
- [Front End](https://github.com/j0lama/CoreKubeFrontend) (MIT license)
- Worker:
  - [4G Worker](https://github.com/andrewferguson/corekube-worker) (AGPL V3 license)
  - [5G Worker](https://github.com/andrewferguson/corekube5g-worker) (AGPL V3 license)
  - [5G Worker Open5GS Libraries](https://github.com/andrewferguson/open5gs-corekube) (AGPL V3 license)
- Database:
  - [DB](https://github.com/j0lama/CoreKubeDB) (MIT license)
  - [libck](https://github.com/j0lama/libck) (MIT license)

## Artifact Evaluation

In order to evaluate the functioning and experiment with the CoreKube system, access to the Powder Platform is necessary. This is to have a reproducible environment with Kubernetes installed and with a specific network topology. In the evaluation process, the artifact also requires an emulator for UEs and base station (eNBs/gNBs) to generate load to the core, which [Nervion RAN emulator](https://github.com/netsys-edinburgh/nervion-powder) (Larrea, 2021) can perform. All of this is included and set up automatically as part of the deployment profile on Powder Platform.

If you are an artifact evaluator but do not have a Powder Platform account, please get in touch with the authors of CoreKube to access a pre-deployed setup, or to arrange a video call to remotely evaluate the artifacts.

Once you have a Powder Platform account, the instructions for evaluation are available in [Evaluation.md](./docs/Evaluation.md).
