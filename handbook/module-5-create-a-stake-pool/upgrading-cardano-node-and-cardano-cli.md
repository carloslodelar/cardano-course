# Upgrading the Cardano node and the Cardano CLI

## Introduction

Ensuring that the Cardano node remains up-to-date is a critical responsibility for all stake pool operators. To facilitate a smooth and secure updating process, it is advisable to compile the `cardano-node` and `cardano-cli` binaries on a separate machine before uploading them to the block-producing server. This approach minimizes the downtime of the actual stake pool server and reduces the risk of encountering build issues during the update.

In certain scenarios, updating the `cardano-node` version may necessitate a chain replay. In such cases, it becomes essential to prepare not only the binaries but also a synchronized blockchain database on the separate machine. This pre-synced database can then be transferred to the block-producing server, enabling the node to resume operations without the need for a lengthy resynchronization from the genesis block.

### 1) Building and uploading the binaries

Start by building and uploading the new Cardano node and Cardano CLI binaries:

1.  **Download or build the new versions.** Download the static binaries from [https://github.com/input-output-hk/cardano-node/releases](https://github.com/input-output-hk/cardano-node/releases) or clone the latest version of the Cardano node repository and compile the new versions of `cardano-node` and `cardano-cli`.


    ```
    git checkout tags/<tag>
    nix build .#hydraJobs.musl.cardano-node-linux
    cd result
    tar -xf cardano-node-x.x.x-linux.tar.gz
    cp ./cardano-node ./cardan0-cli ~/.local/bin/
    ```

### 2) Database operations

If a chain replay is required, follow these steps to sync the node and prepare the database:

1. **Sync the new node.** Start and fully sync the new node with the blockchain on a separate machine. Ensure a clean shutdown of the node to preserve the integrity of the database.
2.  **Compress blockchain data.** Once the new node has fully synced, compress the blockchain data into a tarball:

    ```
    tar -czvf db.tar.gz ./db
    ```
3.  **Calculate SHA-256 hash.** Generate the SHA-256 hash for the tarball:

    ```
    sha256sum db.tar.gz > db.tar.gz.sha256
    ```
4.  **Transfer the database.** Transfer the compressed blockchain data tarball, and the hash file to the `~/src` directory on your block-producing server:

    ```
    scp db.tar.gz <user>@<your_node_ip>:~/src/
    scp db.tar.gz.sha256 <user>@<your_node_ip>:~/src/
    ```


5.  **On the block-producing server,** verify the SHA-256 hash of the transferred tarball:

    ```bash
    sha256sum -c db.tar.gz.sha256
    ```

    If the output says **"cardano-db.tar.gz: OK"**, the file was transferred without corruption.

### 3) Upload binaries and database, and restart the node

Once the database and the new binaries are ready, you can replace the old ones and restart your node:

1.  **Shut down the node.** Cleanly shut down your cardano-node service:

    ```
    sudo systemctl stop cardano-node.service
    ```
2.  **Back up the existing database**:

    ```
    mv ./db ./dbbackup
    ```
    
3.  **Transfer the binaries:**
    Transfer the new binaries to the block-producing server:

    ```
    scp ~/.local/bin/cardano-node <user>@<your_node_ip>:~/src/
    scp ~/.local/bin/cardnao-cli <user>@<your_node_ip>:~/src/    
    ```

4.  **Back up the current binaries.** Before replacing them, back up your current binaries:

    ```
    mv /usr/local/bin/cardano-node /usr/local/bin/cardano-node.bak
    mv /usr/local/bin/cardano-cli /usr/local/bin/cardano-cli.bak
    ```

5.  **Replace the binaries.** Replace the current `cardano-node` and `cardano-cli` binaries with the new ones:

    ```
    cp ~/src/cardano-node /usr/local/bin/
    cp ~/src/cardano-cli /usr/local/bin/
    ```
    
6.  **Extract the new database.** Navigate to `~/src/` and extract the database:

    ```
    tar -xzvf db.tar.gz -C ~/
    ```
    
7.  **Restart the node.** Finally, restart the cardano-node service:

    ```
    sudo systemctl start cardano-node.service
    ```
