# Upgrading cardano-node and cardano-cli

## Introduction

Keeping the Cardano node software up-to-date is a crucial task for all stake pool operators. To ensure a smooth and safe update process, it is advisable to compile the `cardano-node` and `cardano-cli` binaries on a separate machine before uploading them to the block producing server. This minimizes the downtime of the actual stake pool server and mitigates the risk of running into build issues during the update.

In some instances, an update to the `cardano-node` version may require a chain replay. In these cases, it is necessary to prepare not just the binaries, but also a synchronized blockchain database on the separate machine. This pre-synced database can then be uploaded onto the block producing server, allowing the node to continue its operations without a lengthy resynchronization from the genesis block.

### 1) Building and Uploading the Binaries

Start by building and uploading the new Cardano-node and Cardano-CLI binaries:

1. **Build the New Versions:** Clone the latest version of the Cardano-node repository and compile the new versions of `cardano-node` and `cardano-cli` on the separate machine.
2.  **Transfer the Binaries:** Transfer the new binaries to the block producing server:

    ```
    scp <local_path_to_cardano-node> <user>@<your_node_ip>:/usr/local/bin/
    scp <local_path_to_cardano-cli> <user>@<your_node_ip>:/usr/local/bin/
    ```

### 2) Database Operations

If a chain replay is required, follow these steps to sync the node and prepare the database:

1. **Sync the New Node:** Start and fully sync the new node with the blockchain on a separate machine. Ensure a clean shutdown of the node to preserve the integrity of the database.
2.  **Compress the Blockchain Data:** Once the new node has fully synced, compress the blockchain data into a tarball:

    ```
    tar -czvf db.tar.gz ~/db
    ```
3.  **Calculate SHA-256 Hash:** Generate the SHA-256 hash for the tarball:

    ```
    sha256sum db.tar.gz > db.tar.gz.sha256
    ```
4.  **Transfer the Database:** Transfer the compressed blockchain data tarball, and the hash file to the `~/src` directory on your block producing server:

    ```
    scp db.tar.gz <user>@<your_node_ip>:~/src/
    scp db.tar.gz.sha256 <user>@<your_node_ip>:~/src/
    ```

### 3) Replacing Binaries and Database, and Restarting the Node

Once the database and the new binaries are ready, you can replace the old ones and restart your node:

1.  **Shutdown the Node:** Cleanly shutdown your Cardano-node service:

    ```
    sudo systemctl stop cardano-node.service
    ```
2.  **Backup the Current Binaries:** Before replacing them, backup your current binaries:

    ```
    mv /usr/local/bin/cardano-node /usr/local/bin/cardano-node.bak
    mv /usr/local/bin/cardano-cli /usr/local/bin/cardano-cli.bak
    ```
3.  **Replace the Binaries:** Replace the current `cardano-node` and `cardano-cli` binaries with the new ones:

    ```
    mv <path_to_new_cardano-node_binary> /usr/local/bin/cardano-node
    mv <path_to_new_cardano-cli_binary> /usr/local/bin/cardano-cli
    ```
4.  **Extract the New Database:** Navigate to `~/src/` and extract the database:

    ```
    mv ~/db ~/db_backup
    tar -xzvf db.tar.gz -C ~/db
    ```
5.  **Restart the Node:** Finally, restart the Cardano-node service:

    ```
    sudo systemctl start cardano-node.service
    ```
