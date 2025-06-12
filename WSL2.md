## Information
- Subsystem: WSL2 Ubuntu 22.04.5 LTS
- Windows 11
- Clash Tun mode

## Settings
> adduser shawn             # Create User
> usermod -aG sudo shawn    # add the user to sudo
> su - shawn                # login with shawn
> sudo apt update
> apt list --upgradable
> sudo apt update && sudo apt upgrade -y
> sudo apt install -y build-essential git curl wget unzip
> wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
> sudo dpkg -i cuda-keyring_1.0-1_all.deb
> sudo apt update
> sudo apt install -y cuda-toolkit-12-1
> nvcc --version
> sudo apt update
> sudo apt install -y cuda-toolkit-12-1
> nvcc --version
> sudo apt install -y cuda-toolkit-12-1
> ls /usr/local/
> echo 'export PATH=/usr/local/cuda-12.1/bin:$PATH' >> ~/.bashrc
> echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
> source ~/.bashrc
> nvcc --version
> pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
> sudo apt install python3-pip
> pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
> python3 --version
> mkdir -p /home/shawn/pytorch_test
> cd /home/shawn/pytorch_test
> python3 -c "import torch; print(torch.__version__)"
> nano test_gpu.py
> python3 test_gpu.py
   