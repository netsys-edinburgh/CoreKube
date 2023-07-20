# CoreKube: An Efficient, Autoscaling and Resilient Mobile Core System

Intro

## Components

All of the components that conform CoreKube are open-source. They can be accessed through the following links:
- [Front End](https://github.com/j0lama/CoreKubeFrontend) (MIT license)
- Worker:
  - [4G Worker](https://github.com/andrewferguson/corekube-worker) (AGPL V3 license)
  - [5G Worker](https://github.com/andrewferguson/corekube5g-worker) (AGPL V3 license)
- Database:
  - [DB](https://github.com/j0lama/CoreKubeDB) (MIT license)
  - [libck](https://github.com/j0lama/libck) (MIT license)

## Artifact Evaluation

In order to evaluate CoreKube as a core, access to the Powder Platform is necessary. This is due to our work requiring a reproducible and isolated environment with Kubernetes installed and with a specific network topology. In the evaluation process, the artifact also requires an emulator for UEs and eNBs/gNBs to simulate load on the core, which [Nervion Controller](https://github.com/netsys-edinburgh/nervion-powder) (Larrea, 2021) can perform. All of this is included and set up automatically as part of the deployment profile on Powder Platform.

If you are an artifact evaluator but do not have a Powder Platform account, please get in touch with the first authors of CoreKube to access a pre-deployed setup, or to arrange a video call to remotely evaluate the artifacts.

Once you have a Powder Platform account, the instructions for evaluation are available in [Evaluation.md](./docs/Evaluation.md).
