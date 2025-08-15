one click command
bash <(wget -qO- https://raw.githubusercontent.com/astrostake/0G-Labs-script/refs/heads/main/storage-node/galileo/0g_storage_node_v3_chain.sh)

Manual Install

Install necessary packages
sudo apt-get update
sudo apt-get install clang cmake build-essential openssl pkg-config libssl-dev jq git bc

Install go
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version

Install rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
. "$HOME/.cargo/env"


Download and Install 0G Storage Node
git clone -b v1.1.0 https://github.com/0glabs/0g-storage-node.git
cd $HOME/0g-storage-node
git stash
git fetch --all --tags
git checkout v1.1.0
git submodule update --init
cargo build --release

Set config
rm -rf $HOME/0g-storage-node/run/config.toml
curl -o $HOME/0g-storage-node/run/config.toml https://vault.astrostake.xyz/testnet/0g-labs/config-v3.toml

INFO
check miner_key and input your private key
nano $HOME/0g-storage-node/run/config.toml
# Miner key is used to sign blockchain transaction for incentive.
# The value should be a hex string of length 64 without 0x prefix.
#
# Note, the corresponding address should have enough tokens to pay
# transaction gas fee.
miner_key = "YOUR-PRIVATE-KEY"

# Period for querying mine context on chain (in seconds)

Create service
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=ZGS Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-node/run
ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

Star service
sudo systemctl daemon-reload && sudo systemctl enable zgs && sudo systemctl start zgs

