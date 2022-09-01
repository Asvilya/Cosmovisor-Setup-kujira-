# Cosmovisor-Setup-kujira-

Cosmovisor Setup
Setting up Cosmovisor is relatively straightforward. However, it does expect certain environment variables and folder structure to be set.
Cosmovisor allows you to download binaries ahead of time for chain upgrades, meaning that you can do zero (or close to zero) downtime chain upgrades. It's also useful if your local timezone means that a chain upgrade will fall at a bad time.
Rather than having to do stressful ops tasks late at night, it's always better if you can automate them away, and that's what Cosmovisor tries to do.

Install
First, go and get cosmovisor (recommended approach):
go get github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor

​
# or, with go >= 1.15 you can do
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest

​
# to target a specific version:
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
When using cosmovisor, make sure that you do not have auto download of binaries on.

Your installation can be confirmed with:
which cosmovisor

This will return something like:
/home/<your-user>/go/bin/cosmovisor
Building from source allows you to target a specific version of Cosmovisor, in case you do not want to run 1.0.0 yet.

You can also build from source; cosmovisor is in the main cosmos-sdk repo on Github, so you can use Git tags to target a specific version. This example uses a tag, v0.42.7 that refers to the Cosmos SDK, as Cosmovisor-specific tags did not exist before August 2021. The first of these was cosmovisor/v0.1.0, and the second is the current release, cosmovisor/v1.0.0.

git clone https://github.com/cosmos/cosmos-sdk
cd cosmos-sdk
git checkout v0.42.7
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:$(go env GO cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
cd $HOME

Add environment variables to your shell
In the .profile file, usually located at ~/.profile, add:
export DAEMON_NAME=kujirad
export DAEMON_HOME=$HOME/.kujira

Then source your profile to have access to these variables:
source ~/.profile

You can confirm success like so:
echo $DAEMON_NAME

It should return kujirad.
Set up folder structure
Cosmovisor expects a certain folder structure:
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── $DAEMON_NAME
└── upgrades
    └── <name>
        └── bin
            └── $DAEMON_NAME
            
Don't worry about current - that is simply a symlink used by Cosmovisor. The other folders will need setting up, but this is easy:
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
mkdir -p $DAEMON_HOME/cosmovisor/upgrades

Set up genesis binary
Cosmovisor needs to know which binary to use at genesis. We put this in $DAEMON_HOME/cosmovisor/genesis/bin.

First, find the location of the binary you want to use:
which kujirad

Then use the path returned to copy it to the directory Cosmovisor expects. Let's assume the previous command returned /home/your-user/go/bin/ kujirad:
cp /home/<your-user>/go/bin/kujirad $DAEMON_HOME/cosmovisor/genesis/bin

Once you're done, check the folder structure looks correct using a tool like tree.
Set up service

Commands sent to Cosmovisor are sent to the underlying binary. For example, cosmovisor version is the same as typing kujirad version.
Nevertheless, just as we would manage kujirad using a process manager, we would like to make sure Cosmovisor is automatically restarted if something happens, for example an error or reboot.

First, create the service file:
sudo nano /etc/systemd/system/cosmovisor.service

Change the contents of the below to match your setup - cosmovisor is likely at ~/go/bin/cosmovisor regardless of which installation path you took above, but it's worth checking.

[Unit]
Description=cosmovisor
After=network-online.target

[Service]
User=<your-user>
ExecStart=/home/<your-user>/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=kujirad"
Environment="DAEMON_HOME=/home/<your-user>/.kujira"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target

A description of what the environment variables do can be found . Change them depending on your setup.
Note also that we set buffer size explicitly because of a  before version v1.0.0. If you are using v1.0.0, you may omit that line.

In addition, the same issue can be fixed by reducing the log via env variable. If you are unsure, ask on Discord.
Start Cosmovisor

If syncing from a snapshot, do not start Cosmovisor yet.
Finally, enable the service and start it.
sudo -S systemctl daemon-reload
sudo -S systemctl enable cosmovisor
sudo systemctl start cosmovisor

Check it is running using:
sudo systemctl status cosmovisor

If you need to monitor the service after launch, you can view the logs using:
sudo journalctl -u cosmovisor -f

