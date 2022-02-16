[back](./)

### Built With

This project is built with

* [TensorFlow](https://www.tensorflow.org/)
* [Python](https://www.python.org/)

## Getting Started

You can clone this project and run it either on CPU or GPU.

### Prerequisites and dependencies

The following packages are required to run PtychoFlow.

* tensorflow >= 2.3
* numpy
* matplotlib
* scikit-image
* cmcrameri

### Example installation using a conda environment

Installation for **CPU support** is simple using [conda](https://docs.conda.io/en/latest/). The following examples for
setting up a TensorFlow environment have been tested on Ubuntu 20.04 and Ubuntu 20.10.

   ```sh
   conda create -n ENV_NAME tensorflow numpy matplotlib scikit-image
   conda activate ENV_NAME
   ```

If a **Nvidia GPU with CUDA support** is available, the setup is more complicated but yields a significant improvement
of computation speed. You can follow these steps:

1. install cuda-toolkit and Nvidia driver
   ```sh
   sudo apt install nvidia-cuda-toolkit nvidia-driver-460
   ```
2. install CUDA 11.2
   [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

3. install cuDNN (requires free Nvidia developer account)
   [https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)

You might have to troubleshoot the wrong CUDA path in Nvidia's documentation when copying the cuDNN libraries using

   ```sh
   whereis cuda
   ```

In the case of Ubuntu, the path should be /usr/lib/cuda/lib64/.

4. set the correct LD_LIBRARY_PATH
   ```sh
   export LD_LIBRARY_PATH=/usr/lib/cuda/lib64/
   ```

5. (optional) permanently add the export command above to ~/.bashrc
   ```sh
   echo "export LD_LIBRARY_PATH=/usr/lib/cuda/lib64/" >> ~/.bashrc
   ```

6. create a new conda environment and use pip to get the latest version of tensorflow-gpu and dependencies
   ```sh
   conda create -n ENV_NAME python=3.8 pip
   conda activate ENV_NAME
   pip install scikit-image tensorflow-gpu numpy matplotlib
   ```

7. (optional) Install spyder as interactive development environment
   ```sh
   pip install spyder rtree
