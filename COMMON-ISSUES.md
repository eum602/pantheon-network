### Common Issues ###
* <strong>Users dont know where their enode is located: You can know your enode by following these steps:</strong>
    * Enter to your remote machine
    * sudo -i
    * key=$(pantheon --data-path=/root/lacchain/data public-key export-address --to=/root/lacchain/data/nodeAddress | grep -oE "0x[A-Fa-f0-9]*" | sed 's/0x//');ip=$(dig +short myip.opendns.com @resolver1.opendns.com 2>/dev/null || curl -s --retry 2 icanhazip.com);port=60606; echo "enode://${key}@${ip}:${port}" > /root/lacchain/data/enode
    * cat /root/lacchain/data/enode

* <strong>Users do not see their node on http://ethstats.lacchain.io/ , this issue can be solved by checking if docker service is running:</strong>
    * Enter to the remote VM => ssh
    * Enter as root => sudo -i
    * Check if docker related with ehtsats is running: 
    ```shell
    $ docker ps
    ```
    At this point you shoud see something like:
    ```shell
        CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
        dab03ba44290        alethio/ethstats-cli   "./bin/ethstats-cli.…"   6 minutes ago       Up About a minute                       kind_buck
    ```
    * If you do not see that, then you can try to recreate the docker by running:
    ```shell
    $ node_name=node_name_you_used_in_inventory;node_email=email_you_entered_in_inventory; mkdir -p /opt/ethstats-cli && docker run -d --restart always --net host -v /opt/ethstats-cli/:/root/.config/configstore/ alethio/ethstats-cli --register --account-email ${node_email} --node-name ${node_name} --server-url http://ethstats.lacchain.io:3000 --client-url ws://127.0.0.1:4546
    ```
    make sure to replace node_name and node_email with you custom values
    * Finally make sure docker is running:
    ```shell
    $ docker ps
    ```

* <strong>Error when installing oracle java jdk</strong>
    * This issue happens when ansible tries to install java on the remote VM, this issue is associated to the version of java required by the operating system. By default we have linked to download jdk-11.0.4 but sometimes machine requires an upper version like jdk-11.0.5; if this is the problem then the ansible will fall in the task: "ensure java is installed". To solve this issue you can execute the following steps:
        * Download the required java version from oracle: https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html
        * Copy the downloaded installer via scp => for example:
        ```shell
        $ scp /your/local/path/to/the/downloaded/jdk-11.0."x"_linux-x64_bin.tar.gz remote_user@remote_host:
        ```
        * Enter to your VM via ssh.
            Then :
        * sudo apt purge oracle-java11-installer
        * Now you should remove version you copiee before you run ansible:
        ```shell
        $ sudo rm -rf /var/cache/oracle-jdk11-installer-local/jdk-11.0.4_linux-x64_bin.tar.gz
        ```
        * now copy the new downloaded version(jdk-11.0."x"_linux-x64_bin.tar.gz) to the appropriate folder:
        ```shell
        $ sudo cp jdk-11.0."x"_linux-x64_bin.tar.gz /var/cache/oracle-jdk11-installer-local/
        ```
        * sudo apt update
        * Go back to you host machine and run ansible again.