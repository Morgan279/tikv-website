---
title: Test Deployment
description: Deploy a TiKV Cluster for Test
menu:
    "5.1":
        parent: Install TiKV
        weight: 4
---
This guide describes how to install and deploy TiKV for test using TiUP playground and binary installation.

## TiUP Playground

This chapter describes how to deploy a TiKV cluster using TiUP Playground.

1. Install TiUP by executing the following command:

    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

2. Set the TiUP environment variables:

    Redeclare the global environment variables:

    ```bash
    source .bash_profile
    ```

    Confirm whether TiUP is installed:

    ```bash
    tiup
    ```

3. If TiUP has been already installed, update the TiUP playground component to the latest version:

    ```bash
    tiup update --self && tiup update playground
    ```

4. Use TiUP playground to start a local TiKV cluster

    Show TiUP version:

    ```bash
    tiup -v
    ```

    version >= 1.5.2:

    ```bash
    tiup playground --mode tikv-slim
    ```

    version < 1.5.2:

    ```bash
    tiup playground
    ```

5. Press `Ctrl + C` to stop the local TiKV cluster

{{< info >}}
Refer to [TiUP playground document](https://docs.pingcap.com/tidb/stable/tiup-playground) to find more TiUP playground commands.
{{< /info >}}

## Install binary manually

This chapter describes how to deploy a TiKV cluster using binary files.

- To quickly understand and try TiKV, see [Deploy the TiKV cluster on a single machine](#deploy-the-tikv-cluster-on-a-single-machine).
- To try TiKV out and explore the features, see [Deploy the TiKV cluster on multiple nodes for testing](#deploy-the-tikv-cluster-on-multiple-nodes-for-testing).

{{< warning >}}
The TiKV team strongly recommends you use the [**TiUP Cluster Deployment**](../production/) method.

Other methods are documented for informational purposes.
{{< /warning >}}

### Deploy the TiKV cluster on a single machine

This section describes how to deploy TiKV on a single machine (Linux for example). Take the following steps:

1. Download the official binary package.

    ```bash
    # Download the package.
    wget https://download.pingcap.org/tidb-latest-linux-amd64.tar.gz
    wget http://download.pingcap.org/tidb-latest-linux-amd64.sha256

    # Check the file integrity. If the result is OK, the file is correct.
    sha256sum -c tidb-latest-linux-amd64.sha256

    # Extract the package.
    tar -xzf tidb-latest-linux-amd64.tar.gz
    cd tidb-latest-linux-amd64
    ```

2. Start PD.

    ```bash
    ./bin/pd-server --name=pd1 \
                    --data-dir=pd1 \
                    --client-urls="http://127.0.0.1:2379" \
                    --peer-urls="http://127.0.0.1:2380" \
                    --initial-cluster="pd1=http://127.0.0.1:2380" \
                    --log-file=pd1.log
    ```

3. Start TiKV.

    To start the 3 TiKV instances, open a new terminal tab or window, come to the `tidb-latest-linux-amd64` directory, and start the instances using the following command:

    ```bash
    ./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                    --addr="127.0.0.1:20160" \
                    --data-dir=tikv1 \
                    --log-file=tikv1.log

    ./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                    --addr="127.0.0.1:20161" \
                    --data-dir=tikv2 \
                    --log-file=tikv2.log

    ./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                    --addr="127.0.0.1:20162" \
                    --data-dir=tikv3 \
                    --log-file=tikv3.log
    ```

You can use the [pd-ctl](https://github.com/pingcap/pd/tree/master/tools/pd-ctl) tool to verify whether PD and TiKV are successfully deployed:

```bash
./bin/pd-ctl store -d -u http://127.0.0.1:2379
```

If the state of all the TiKV instances is "Up", you have successfully deployed a TiKV cluster.

### Deploy the TiKV cluster on multiple nodes for testing

This section describes how to deploy TiKV on multiple nodes. If you want to test TiKV with a limited number of nodes, you can use one PD instance to test the entire cluster.

Assume that you have four nodes, you can deploy 1 PD instance and 3 TiKV instances. For details, see the following table:

| Name  | Host IP         | Services |
|:----- |:--------------- |:-------- |
| Node1 | 192.168.199.113 | PD1      |
| Node2 | 192.168.199.114 | TiKV1    |
| Node3 | 192.168.199.115 | TiKV2    |
| Node4 | 192.168.199.116 | TiKV3    |

To deploy a TiKV cluster with multiple nodes for test, take the following steps:

1. Download the official binary package on each node.

    ```bash
    # Download the package.
    wget https://download.pingcap.org/tidb-latest-linux-amd64.tar.gz
    wget http://download.pingcap.org/tidb-latest-linux-amd64.sha256

    # Check the file integrity. If the result is OK, the file is correct.
    sha256sum -c tidb-latest-linux-amd64.sha256

    # Extract the package.
    tar -xzf tidb-latest-linux-amd64.tar.gz
    cd tidb-latest-linux-amd64
    ```

2. Start PD on Node1.

    ```bash
    ./bin/pd-server --name=pd1 \
                    --data-dir=pd1 \
                    --client-urls="http://192.168.199.113:2379" \
                    --peer-urls="http://192.168.199.113:2380" \
                    --initial-cluster="pd1=http://192.168.199.113:2380" \
                    --log-file=pd1.log
    ```

3. Log in and start TiKV on other nodes: Node2, Node3 and Node4.

    Node2:

    ```bash
    ./bin/tikv-server --pd-endpoints="192.168.199.113:2379" \
                    --addr="192.168.199.114:20160" \
                    --data-dir=tikv1 \
                    --log-file=tikv1.log
    ```

    Node3:

    ```bash
    ./bin/tikv-server --pd-endpoints="192.168.199.113:2379" \
                    --addr="192.168.199.115:20160" \
                    --data-dir=tikv2 \
                    --log-file=tikv2.log
    ```

    Node4:

    ```bash
    ./bin/tikv-server --pd-endpoints="192.168.199.113:2379" \
                    --addr="192.168.199.116:20160" \
                    --data-dir=tikv3 \
                    --log-file=tikv3.log
    ```

You can use the [pd-ctl](https://github.com/pingcap/pd/tree/master/tools/pd-ctl) tool to verify whether PD and TiKV are successfully deployed:

```
./pd-ctl store -d -u http://192.168.199.113:2379
```

The result displays the store count and detailed information regarding each store. If the state of all the TiKV instances is "Up", you have successfully deployed a TiKV cluster.
