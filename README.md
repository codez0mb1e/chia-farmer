
# Chia Network Farmer

In this repository collected the most useful script and libraries that help to optimize farming process of Chia plots.

## Deployment of Chia Mining cluster

### Set up VM

Use _Storage optimized_ VM instances in Microsoft Azure. For this type of VM instances is available _NVMe SSD(s)_ on demand.

Create [Lsv2-series VM](https://docs.microsoft.com/en-us/azure/virtual-machines/lsv2-series) using [Azure Portal](https://portal.azure.com/#create/Canonical.UbuntuServer1804LTS-ARM) or Azure CLI.

Connect with VM and update: `sudo apt update && sudo apt upgrade -y`

Add non-root user (optional):

```bash
sudo adduser dp
sudo usermod -aG sudo dp
```

### Mount NVMe volume ([instruction](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/attach-disk-portal))

```bash
# view available disks
lsblk

# for each NVMe drive
sudo parted /dev/nvme0n1 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/nvme0n1p1
sudo partprobe /dev/nvme0n1p1

sudo mkdir /plotdrive1
sudo mount /dev/nvme0n1p1 /plotdrive1
sudo chmod -R 777 /plotdrive1


# for each HDD drive
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1

sudo mkdir /harvestdrive1
sudo mount /dev/sdc1 /harvestdrive1
sudo chmod -R 777 /harvestdrive1

# (optional) Set auto mount after reboot
df -h
```

### Chia Software

Install Chia Software and activate it ([instruction](https://github.com/Chia-Network/chia-blockchain/wiki/INSTALL#ubuntudebian)):

```bash
# install python 3.7 (if necessary)

sudo apt -y install python3.7
sudo apt install python3.7-venv python3.7-distutils python3.7-dev git lsb-release -y


# Checkout the source and install
git clone https://github.com/Chia-Network/chia-blockchain.git -b latest --recurse-submodules
cd chia-blockchain

sh install.sh

. ./activate
```

### Run Chia

Init Chia using [CLI](https://github.com/Chia-Network/chia-blockchain/wiki/CLI-Commands-Reference):

```bash
chia version

chia keys add
chia keys show

chia init

chia start farmer
```

[Start plotting](https://github.com/Chia-Network/chia-blockchain/wiki/CLI-Commands-Reference#create):

```bash
chia keys show # see your farmer key

chia plots create -k 32 -n 1 -b 5000 -r 2 -t /plotdrive1 -d /harvestdrive1 -f $farmer_key 2>&1 | tee ~/chia-blockchain/logs/$log_name.log
```

### Tools

Install Powershell ([instruction](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-7.1)):

```bash
sudo apt install -y wget apt-transport-https software-properties-common
# Download and register  the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt update
sudo add-apt-repository universe

sudo apt install -y powershell

pwsh # Start PowerShell
```

Install .NET Core: [steps](https://github.com/codez0mb1e/cloud-rstudio-server/blob/master/scripts/install_core.sh).


### Monitoring

[Monitoring NVMe](https://github.com/linux-nvme/nvme-cli):

```bash
sudo apt -y install nvme-cli
sudo nvme list

sudo nvme smart-log /dev/nvme0n1 | grep percentage_used
```

Monitoring plotting vai [PSChiaPlotter](https://github.com/MrPig91/PSChiaPlotter) (WARN: only for Windows):

```powershell
Install-Module -Repository PSGallery -Name PSChiaPlotter
Get-ChiaPlottingStatistic | sort Time_started -Descending | select -first 20
```

### Chia GUI (obsolete)

The GUI requires you have Ubuntu Desktop or a similar windowing system installed.
WARN: _You can not install and run the GUI as root._

```bash
# Install GUI ---- 
sudo apt -y install xfce4 # or ubuntu-desktop
sudo reboot

# Remote access [2] ----

sudo apt-get -y install xrdp
sudo systemctl enable xrdp

echo xfce4-session >~/.xsession

sudo service xrdp restart

az vm open-port --resource-group $resource_group_name --name $vm_name --port 3389

# Install Chia GUI
chmod +x ./install-gui.sh
./install-gui.sh

cd chia-blockchain-gui
npm run electron &
```

## References

1. https://www.chia.net/
1. https://github.com/Chia-Network/chia-blockchain/
