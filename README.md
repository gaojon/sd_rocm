# Stable Diffusion setup on Mi210 with docker

Install with docker will be easier for most users. This will cover how to install SD with docker.

## Install ROCm on host 

Just follow the link to install the latest version of ROCm. In following link, it shows how to install ROCm 6.1.6

Just for you reference. You don't need to jump to this link. 
https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html


In Ubuntu 22.04
```
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
wget https://repo.radeon.com/amdgpu-install/6.1.2/ubuntu/jammy/amdgpu-install_6.1.60102-1_all.deb
sudo apt install ./amdgpu-install_6.1.60102-1_all.deb
sudo apt update
sudo apt install amdgpu-dkms rocm
```

## Check the ROCm is running correctly on host

```
$rocm-smi
========================================== ROCm System Management Interface ==========================================
==================================================== Concise Info ====================================================
Device  Node  IDs              Temp    Power   Partitions          SCLK     MCLK     Fan  Perf  PwrCap  VRAM%  GPU%  
              (DID,     GUID)  (Edge)  (Avg)   (Mem, Compute, ID)                                                    
======================================================================================================================
0       2     0x740f,   21782  28.0°C  42.0W   N/A, N/A, 0         800Mhz   1600Mhz  0%   auto  300.0W  92%    0%
```

## Setting up with docker

pull rocm/pytorch from hub

https://hub.docker.com/r/rocm/pytorch

```
docker pull rocm/pytorch:latest
```

Start the docker and map the ethernet port 7860 and map your docker directory under your home to docker. You will download the SD into your home docker directory. 
```
docker run -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined \
--device=/dev/kfd --device=/dev/dri --group-add video --name sd \
--ipc=host --shm-size 8G -p 0.0.0.0:7860:7860 -v $HOME/docker:/docker rocm/pytorch:latest

```

After docker initiated successfully, you will get interaction shell. Input following command inside the docker.

Get the stable diffusion Web UI from Github
```
cd /docker

git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
python -m pip install --upgrade pip wheel
pip install -r requirements.txt

python launch.py --listen
```
This is going to take a while to download the models. 



## launch web browser to connect to Web UI

Use chrome or fiefox to start a browser and use 0.0.0.0:7860 to connect to SD. Then, enjoy and explore the SD features! 


## Tips

If you have multiple GPU card, you could choose the one before launching SD. Here is the example to use the second GPU. 
```
export CUDA_VISIBLE_DEVICES=1
python launch.py --listen
```


Check if the PyTorch include the GPU arch which you are using

```
python -c "import torch;print(torch.cuda.get_arch_list())"
```

