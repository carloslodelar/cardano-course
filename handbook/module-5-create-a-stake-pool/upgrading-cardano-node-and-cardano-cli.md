# Upgrading cardano-node and cardano-cli

As stake pool operators, maintaining our nodes with the most recent software release is vital for ensuring optimal performance and security. We recommend the practice of compiling the `cardano-node` and `cardano-cli` on a standalone machine, reserving the transfer of the binary files to our live servers only when we are fully prepared for the update.

Moreover, it's crucial to ascertain if the new version of `cardano-node` requires a replay of the blockchain. In such cases, it's advantageous to execute this synchronization on a distinct machine. This strategy allows us to prepare and transfer a fully updated database, ensuring that our main servers remain operational with minimal downtime.\


1. **Build the New Node:** As before, build the new version of the Cardano node on your separate machine.
2. **Sync the New Node:** Start and fully sync the new node with the blockchain on the separate machine.
3.  **Compress the Blockchain Data:** Once the new node has fully synced with the blockchain, compress the blockchain data into a tarball:\


    ```bash
    tar -czvf db.tar.gz ~/db
    ```


4.  **Calculate SHA-256 Hash:** Generate the SHA-256 hash for the tarball:\


    ```bash
    sha256sum db.tar.gz > db.tar.gz.sha256
    ```


5.  **Transfer the New Node, Data and Hash:** Transfer the new node binaries, the compressed blockchain data tarball, and the hash file to the `~/src` directory on your block producing server:\


    ```bash
    scp ~/.local/bin/cardano-node <user>@<your_node_ip>:~/src/
    scp ~/.local/bin/cardano-cli <user>@<your_node_ip>:~/src/
    scp db.tar.gz <user>@<your_node_ip>:~/src/
    scp db.tar.gz.sha256 <user>@<your_node_ip>:~/src/
    ```


6.  **On the Block Producing Server:** Navigate to the `~/src` directory and verify the SHA-256 hash of the transferred tarball:\


    ```bash
    cd ~/src
    sha256sum -c db.tar.gz.sha256
    ```

    \
    If the output says "db.tar.gz: OK", the file was transferred without corruption.\

7.  **Check the Leadership Schedule:** Before you stop the old node, check the leadership schedule to see when your node is next scheduled to make a block:\


    ```bash
    cardano-cli query leadership-schedule --testnet-magic 2 --cold-verification-key-file cold.vkey --genesis shelley-genesis.json --vrf-signing-key-file vrf.skey --current 
    ```

    \
    If your node is scheduled to make a block in the near future, you might want to delay the update until after that time.\

8.  **Replace the Existing Node:** Once it's a good time to update, stop the old version of the Cardano node and replace it with the new version from the `~/src` directory.\


    ```
    sudo systemctl stop cardano-node.service
    cp ~/src/cardano-node ~/src/cardano-cli /usr/local/bin/
    ```


9.  **Extract the Blockchain Data and Replace the Existing Database:** Still in the `~/src` directory, uncompress the blockchain database tarball and replace the existing database with the new one:\


    ```bash
    tar -xzvf db.tar.gz
    mv ~/db ~/db.old
    mv ~/src/db ~/db
    ```


10. **Start the Node:** Start the new node. Check the logs to ensure it is running correctly and is using the copied blockchain data.
