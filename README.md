# MQ Channel Adapter

A channel adapter between a MQ integration platform and any other application.


## Requirements

In order to run this You wil need a mq client.
For Linux 9.1.0.0-IBM-MQC-UbuntuLinuxX64.tar.gz will do just fine
and it is available for download at IBM website. Search for it there.
Other versions might or might not work,
but the 9.1.0.0 version of the client library is used during the development of this code.

- Install the client library:
```bash
sudo su -
mkdir /usr/local/src/mqc-9.1.0.0
cd /usr/local/src/mqc-9.1.0.0
tar xvzf /path-to-downloads/9.1.0.0-IBM-MQC-UbuntuLinuxX64.tar.gz
./mqlicense.sh -accept
dpkg -i ibmmq-runtime_9.1.0.0_amd64.deb
dpkg -i ibmmq-client_9.1.0.0_amd64.deb
dpkg -i ibmmq-man_9.1.0.0_amd64.deb
dpkg -i ibmmq-sdk_9.1.0.0_amd64.deb
dpkg -i ibmmq-samples_9.1.0.0_amd64.deb
dpkg -i ibmmq-jre_9.1.0.0_amd64.deb
dpkg -i ibmmq-java_9.1.0.0_amd64.deb
dpkg -i ibmmq-gskit_9.1.0.0_amd64.deb
dpkg -i ibmmq-sfbridge_9.1.0.0_amd64.deb
dpkg -i ibmmq-bcbridge_9.1.0.0_amd64.deb
logout
```

- In order to use the code, You will need ruby and we strongly suggest
  **rvm** or perhaps **rbenv** for managing versions of ruby.
  The given example is using **rvm**:

```bash
cd ${HOME}/path/to/where/you/store/code/
git clone git@github.com:ub-digit/mq-channel-adapter.git
cd mq-channel-adapter
rvm install 2.3.1
bundle
```

- If You run into troubles when installing ruby 2.3.1, please find a solution
  and solve the problems using for instance google.
- To run the client successfully, You will need a configuration file populated
  with relevant data for the message type. Please see the config.yml file and
  fill out the relevant data for conecting to Your message queue.


## Execution

- Run the task using the rake command:
```bash
rake connect_and_fetch['message_type_name']
```

---------- --------- --------- --------- --------- --------- --------- ---------