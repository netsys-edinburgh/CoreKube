# Evaluating CoreKube

In order to evaluate CoreKube as a core, we need an environment to simulate UEs and RANs that connect to it. We will use the [Nervion](https://github.com/netsys-edinburgh/nervion-powder) simulator. The official Nervion Powder Profile, present as a deployment profile within the [Powder Platform](https://powderwireless.net/), will deploy both Nervion and CoreKube on Powder.

## Creating an Experiment

To get started, you will need to log in to the Powder Platform and start a new experiment with the Nervion profile. We assume that you have an account on the Powder Platform with your SSH public key attached. Detailed instructions are presented below:

![Screenshot showing the Powder Platform](./images/start-experiment.png)

1. [Log in](https://www.powderwireless.net/login.php) to the Powder Platform
2. Click the top-left button "Experiments," and select from the dropdown "Start Experiment." You will be directed to a page to configure your new experiment.
3. Within Step 1 ("Select a Profile"), click the "Select Profile" button to open the dialogue to change the deployment profile. Search for "Nervion", and select it.
4. Move onto Step 2, where you will be presented configuration options for the deployment. Below, we list the value each parameter should be set to.

    - **[Number of slave/compute nodes]** controls the number of VMs or physical nodes that will be part of the Kubernetes cluster for Nervion. Set this to **[2]**.
    - **[EPC implementation]** controls the mobile core network to deploy and test. Set this to **[CoreKube]**.
    - **[EPC hardware]** controls the hardware to run the mobile core network on. Set this to **[d430]** to run it on a physical hardware.
    - **[Multiplexer]** controls whether the dataplane multiplexer is enabled. The dataplane is not part of CoreKube, so we do not need this enabled. Set this to **[False]**.
    - **[VM number of cores]** controls the number of cores VMs will have if you have selected VM as the EPC hardware. We recommend keeping this at the default value.
    - **[RAM size]** controls the RAM VMs will have if you have selected VM as the EPC hardware. We recommend keeping this at the default value.
    - **[Number of slave/compute nodes for CoreKube]** controls the number of nodes that will be part of the Kubernetes cluster for CoreKube. Set this to **[2]**.
    - **[CoreKube Version]** controls which implementation of the CoreKube worker to use. Set this to **[5G]**.

![Screenshot of the configuration parameters](./images/deployment-config.png)

5. Move onto Step 3, and give an optional name to the experiment. The node graph on the page should show 6 nodes: three for the Nervion cluster (`master`, `slave0`, and `slave1`) and three for the CoreKube cluster (`masterck`, `ck_slave0`, and `ck_slave1`).
6. Move onto Step 4. Unless you need to extend the experiment or schedule it in a specific time frame, you can finalize the instantiation by clicking on "Finish".

The experiment is deployed and ready when, within the "List View" tab, all rows have a status of "ready" and say "Finished" for the "Startup" column. This can take several minutes, and possibly longer if deployed to a VM instead of a physical machine.

At this point, CoreKube will already be running on Kubernetes. This can be verified by SSH-ing into the `masterck` node (which acts as the CoreKube cluster controller), and executing the following:

```
user@masterck:~$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
corekube-db-85b854b574-69zlv         1/1     Running   0          107s
corekube-frontend-6884d7f88f-r8tkq   1/1     Running   0          107s
corekube-worker-57456fcfcb-kjpxf     1/1     Running   0          107s
corekube-worker-57456fcfcb-ps5nw     1/1     Running   0          106s
```

The CoreKube worker instances are distributed automatically onto the two `ck_slave` nodes, which you can also verify by executing the same command with `--output=wide`.

## Putting CoreKube Under Load

To evaluate CoreKube's scalability properties, we will put it under high load by using Nervion to simulate of 100 UEs continuously attaching and detaching. Detailed instructions are presented below:

![Screenshot of the node list on Powder](./images/nervion-node.png)
1. Determine the hostname of the master node for the Nervion Kubernetes cluster. This is the node with the ID "`master`". The hostname will be in the format "`pcXXX`" where XXX are digits.
2. In your web browser, go to `http://<hostname>.emulab.net:34567`, which is Nervion Controller's web interface, and where we will configure the load to put CoreKube under.
3. Configure the simulation according to the values we list below for each parameter.

    - **[MME/AMF IP Address]** is the IP address of the MME/AMF. This is equivalent to the core network's IP. The deployment profile we used will automatically assign `192.168.4.80` to the core network, so set this to **[192.168.4.80]**.
    - **[Control-Plane Mode]** provides better scalability by disabling the data-plane. If this mode is enabled, each UE Kubernetes pod will emulate multiple UEs via multi-threading, instead of just one. **[Tick]** this as we do not need a data-plane, and set the **[Number of threads per nUE]** to **[10]**.
    - **[Multiplexer IP Address]** is the IP address of the multiplexer. Since we are not using one, leave this empty.
    - **[Configuration file]** is a JSON file that specifies the behaviour that the UEs and the RAN should take in the simulation. For reference, A complete guide on the JSON format is [provided in the Nervion repository](https://github.com/netsys-edinburgh/nervion-powder/blob/master/doc/scenarios.md). Use **[100-100-switchoff-ck.json](./100-100-switchoff-ck.json)**, which specifies that there should be 100 UEs each connected to their own RAN (100 of them), performing a continuous loop of attaching and detaching (with switch-off).
    - **[Slave Docker Image]** is the tag for the Docker image to use as the Nervion simulation slaves. Set this to **[j0lama/ran_slave_5g_noreset:latest]**.
    - **[Web-interface Refresh Time]** controls how often the web interface refreshes in seconds. It is fine to leave this at the default value.

![Screenshot of the Nervion Controller Web UI](./images/nervion-webui.png)

4. Press "Submit" to start the simulation.

At this point, 110 new Kubernetes pods are being created in the Nervion cluster: 100 for the RAN, and 10 for multi-threaded UEs. We should have 111 in total if we include the Nervion controller pod. We can verify this by SSH-ing into the `master` node, and executing the following:

```
user@master:~$ kubectl get pods --no-headers | wc -l
111
user@master:~$ kubectl get pods
NAME                            READY   STATUS              RESTARTS   AGE
ran-emulator-7774b8d457-qnn66   1/1     Running             0          43m
slave-0                         0/1     ContainerCreating   0          33s
slave-1                         1/1     Running             0          33s
slave-10                        0/1     ContainerCreating   0          33s
slave-100                       0/1     ContainerCreating   0          32s
slave-101                       0/1     ContainerCreating   0          32s
...
```

The simulation has started successfully when all of the containers show "Running" as its status.

## Verifying CoreKube's Scalability

Now that CoreKube is running, we can verify its scalability properties by examining how many worker nodes Kubernetes has spun up in addition to the two that we started with. This auto-scaling property is explained in detail in Section 5.3 of our paper.

SSH into `masterck` (master node for the CoreKube cluster) and execute `kubectl get pods`.

```
user@masterck:~$ kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
corekube-db-85b854b574-r66qz         1/1     Running            0          89m
corekube-frontend-6884d7f88f-nr4zs   1/1     Running            0          89m
corekube-worker-5845b465f4-2l428     1/1     Running            0          36m
corekube-worker-5845b465f4-8hj66     1/1     Running            0          28m
corekube-worker-5845b465f4-bxl92     1/1     Running            0          28m
corekube-worker-5845b465f4-n9lhj     1/1     Running            0          5m56s
corekube-worker-5845b465f4-pvgs6     1/1     Running            0          5m56s
...
```

Recall that we started the experiment with two workers, but now there are more instances for it, demonstrating that CoreKube has scaled dynamically to meet the demands of the load. We can find the exact number of workers that CoreKube has spun up with the following command:

```
user@masterck:~$ kubectl get pods --no-headers | wc -l | xargs bash -c 'echo $(($0 - 2)) workers'
16 workers
```

## Verifying CoreKube's Resilience

We can evaluate CoreKube's resilience to outages by simulating a worker instance crash, and seeing that it will self-heal by spinning a replacement up. This behaviour is also examined in Section 5.4 of our paper.

SSH into `masterck` (master node for the CoreKube cluster) and execute `kubectl get pods`. Choose any random one worker instance, and copy its name. Then execute the following to delete that instance:

```
kubectl delete pod <instance-name>
```

This is simulating the instance encountering a critical error and crashing.

If you execute `kubectl get pods` afterwards, you will notice one less worker instance as expected. However, after a few seconds, a new instance will spin up. This can be checked by running `kubectl get pods` again, and finding a worker with a fresh age (50s, in the example below).

```
user@masterck:~$ kubectl delete pod corekube-worker-5845b465f4-pvgs6
pod "corekube-worker-5845b465f4-pvgs6" deleted
user@masterck:~$ kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
corekube-db-85b854b574-r66qz         1/1     Running            0          4h16m
corekube-worker-5845b465f4-8hj66     1/1     Running            0          3h15m
corekube-worker-5845b465f4-bxl92     1/1     Running            0          3h15m
corekube-worker-5845b465f4-zfnvj     1/1     Running            0          50s
...
```

The other worker instances are unaffected. This behaviour demonstrates CoreKube's ability to self-heal and prevent outages in the occasion of a random critical error. This is thanks to all workers being stateless, which means they can distribute the messages that would've originally been routed to the deleted instance.
