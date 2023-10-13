# ARKEO

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.20.5"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
cd $HOME
git clone https://github.com/arkeonetwork/arkeo
cd arkeo
git checkout ab05b124336ace257baa2cac07f7d1bfeed9ac02
make install
````
Let's check the version (current as of October 2023 - arkeo commit: b3f8d880dfcdb3265d321e465b47b04071d9480f):
````
arkeod version --long
````
