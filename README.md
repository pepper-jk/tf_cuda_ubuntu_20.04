# CUDA for Tensorflow 2 on Ubuntu 20.04

As our server was upgraded from Ubuntu 18.04 to 20.04 recently I had to install the CUDA libraries again.

> NOTE: I'd suggest reading the whole guide before doing anything.

## TL:DR

Installing most of the missing CUDA libs:
```shell
$ sudo apt install nvidia-cuda-toolkit
```

Testing if it worked:
```shell
$ nvcc -V
```

### cuDNN

Downloading the `cuDNN` library from [here](https://developer.nvidia.com/cudnn).

Extracting it:
```shell
# your version number may vary
# mine was 8.0.2.39
$ tar -xvzf cudnn-10.1-linux-x64-vVERSION.tgz
```

Copying it:
```shell
$ sudo cp cuda/include/cudnn.h /usr/lib/cuda/include/
$ sudo cp cuda/lib64/libcudnn* /usr/lib/cuda/lib64/
```

Changing the permissions:
```shell
$ sudo chmod a+r /usr/lib/cuda/include/cudnn.h /usr/lib/cuda/lib64/libcudnn*
```

Creating the link for the version 7:
```shell
$ sudo ln /usr/lib/cuda/lib64/libcudnn.so /usr/lib/cuda/lib64/libcudnn.so.7
```

## Long Story

It started with this error from tensorflow:

```python
2020-08-24 11:08:24.319925: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudart.so.10.1'; dlerror: libcudart.so.10.1: cannot open shared object file: No such file or directory
2020-08-24 11:08:24.319957: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
2020-08-24 11:08:28.538684: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcuda.so.1
2020-08-24 11:08:28.568340: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1716] Found device 0 with properties:
pciBusID: 0000:82:00.0 name: Tesla V100S-PCIE-32GB computeCapability: 7.0
coreClock: 1.597GHz coreCount: 80 deviceMemorySize: 31.75GiB deviceMemoryBandwidth: 1.03TiB/s
2020-08-24 11:08:28.568448: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudart.so.10.1'; dlerror: libcudart.so.10.1: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568519: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcublas.so.10'; dlerror: libcublas.so.10: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568589: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcufft.so.10'; dlerror: libcufft.so.10: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568646: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcurand.so.10'; dlerror: libcurand.so.10: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568702: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcusolver.so.10'; dlerror: libcusolver.so.10: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568767: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcusparse.so.10'; dlerror: libcusparse.so.10: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568827: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudnn.so.7'; dlerror: libcudnn.so.7: cannot open shared object file: No such file or directory
2020-08-24 11:08:28.568905: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1753] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
Skipping registering GPU devices...
```

As you can see Tensorlfow 2 is missing quite alot of libs.

At first I wanted to follow the [official guide](https://www.tensorflow.org/install/gpu#linux_setup) provided by Tensorflow regarding the GPU support dependencies.
However it only provides instructions up to 18.04.

But I found [this guide](https://towardsdatascience.com/installing-tensorflow-gpu-in-ubuntu-20-04-4ee3ca4cb75d?source=friends_link&sk=6d03c3521893cbd2f9858fe07aa1c3f7) detailing instructions on how to get it running on 20.04

### nvidia-cuda-toolkit

This will install most of the missing CUDA libs:
```shell
$ sudo apt install nvidia-cuda-toolkit
```

It also provides CUDA 10.1, which is required by Tensorflow 2.

To test if the installation worked, you can check the version like so:
```shell
$ nvcc -V
```

I got this output:
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243
```

### cuDNN

The only thing missing now is the `cuDNN` library.

You can download it from [here](https://developer.nvidia.com/cudnn).
Sadly one needs to register with nvidia for this.
This also means no `wget`-magic to get the file onto the server quickly.

Even though the guide suggests the latest version 7.x.x, I just took the latest version 8.0.2 (July 24th, 2020) as of writing this.
I figured it was better to have the newest install possible for the future.
Just choose the download option `cuDNN Library for Linux (x86)`.

After downloading it on my personal machine I used `scp` to push it to our compute server.
```shell
scp Downloads/cudnn-10.1-linux-x64-v8.0.2.39.tgz user@server:
```

There I extracted it:
```shell
# your version number may vary
$ tar -xvzf cudnn-10.1-linux-x64-v8.0.2.39.tgz
```

> NOTE: I really dislike this manual installation, but it seems to be the only way to get cuDNN running on Ubuntu 20.04.
>
> If you read this guide at a later point in time, make sure there is no better way before doing this.
>
> I'd much rather prefer an official repository that provides the packages or even a deb file, so that the package manager has control over the files.

Afterwards I copied it to the cuda directories:
```shell
$ sudo cp cuda/include/cudnn.h /usr/lib/cuda/include/
$ sudo cp cuda/lib64/libcudnn* /usr/lib/cuda/lib64/
```
> NOTE: doublecheck that the directories exist, before copying.

You can also check the CUDA location like this:
```shell
$ whereis cuda
cuda: /usr/lib/cuda /usr/include/cuda.h
```

Changing the permissions:
```shell
$ sudo chmod a+r /usr/lib/cuda/include/cudnn.h /usr/lib/cuda/lib64/libcudnn*
```

After this I still got this error:
```python
2020-08-24 18:46:09.032698: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudnn.so.7'; dlerror: libcudnn.so.7: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /usr/lib/cuda/lib64:/usr/lib/cuda/include::/usr/local/cuda/extras/CUPTI/lib64
```

So I created a link for the shared object of version 7:
```shell
$ sudo ln /usr/lib/cuda/lib64/libcudnn.so /usr/lib/cuda/lib64/libcudnn.so.7
```
> NOTE: This worked for me, but if you want to go a bit more by the book, I'd suggest using `cuDNN` 7 just to be sure.

After this Tensorflow 2 worked smoothly on GPU. You should get an output similar to this, when executing Tensorflow 2:

```python
2020-08-24 18:50:16.699697: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.1
True
2020-08-24 18:51:32.466273: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcuda.so.1
2020-08-24 18:51:32.495408: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1716] Found device 0 with properties:
pciBusID: 0000:82:00.0 name: Tesla V100S-PCIE-32GB computeCapability: 7.0
coreClock: 1.597GHz coreCount: 80 deviceMemorySize: 31.75GiB deviceMemoryBandwidth: 1.03TiB/s
2020-08-24 18:51:32.495479: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.1
2020-08-24 18:51:32.499772: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcublas.so.10
2020-08-24 18:51:32.503055: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcufft.so.10
2020-08-24 18:51:32.503456: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcurand.so.10
2020-08-24 18:51:32.506749: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcusolver.so.10
2020-08-24 18:51:32.508938: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcusparse.so.10
2020-08-24 18:51:32.509098: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudnn.so.7
2020-08-24 18:51:32.511976: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1858] Adding visible gpu devices: 0
2020-08-24 18:51:32.512438: I tensorflow/core/platform/cpu_feature_guard.cc:142] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN)to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
2020-08-24 18:51:32.527027: I tensorflow/core/platform/profile_utils/cpu_utils.cc:104] CPU Frequency: 3000005000 Hz
2020-08-24 18:51:32.529261: I tensorflow/compiler/xla/service/service.cc:168] XLA service 0xed477ba0 initialized for platform Host (this does not guarantee that XLA will be used). Devices:
2020-08-24 18:51:32.529296: I tensorflow/compiler/xla/service/service.cc:176]   StreamExecutor device (0): Host, Default Version
2020-08-24 18:51:32.759597: I tensorflow/compiler/xla/service/service.cc:168] XLA service 0x4151e20 initialized for platform CUDA (this does not guarantee that XLA will be used). Devices:
2020-08-24 18:51:32.759667: I tensorflow/compiler/xla/service/service.cc:176]   StreamExecutor device (0): Tesla V100S-PCIE-32GB, Compute Capability 7.0
2020-08-24 18:51:32.763203: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1716] Found device 0 with properties:
pciBusID: 0000:82:00.0 name: Tesla V100S-PCIE-32GB computeCapability: 7.0
coreClock: 1.597GHz coreCount: 80 deviceMemorySize: 31.75GiB deviceMemoryBandwidth: 1.03TiB/s
2020-08-24 18:51:32.763280: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.1
2020-08-24 18:51:32.763339: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcublas.so.10
2020-08-24 18:51:32.763374: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcufft.so.10
2020-08-24 18:51:32.763407: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcurand.so.10
2020-08-24 18:51:32.763440: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcusolver.so.10
2020-08-24 18:51:32.763472: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcusparse.so.10
2020-08-24 18:51:32.763505: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudnn.so.7
2020-08-24 18:51:32.770512: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1858] Adding visible gpu devices: 0
2020-08-24 18:51:32.770584: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.1
2020-08-24 18:51:33.261041: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1257] Device interconnect StreamExecutor with strength 1 edge matrix:
2020-08-24 18:51:33.261095: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1263]      0
2020-08-24 18:51:33.261106: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1276] 0:   N
2020-08-24 18:51:33.263962: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1402] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 29552 MB memory) -> physical GPU (device: 0, name: Tesla V100S-PCIE-32GB, pci bus id: 0000:82:00.0, compute capability: 7.0)
```
