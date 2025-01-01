# TensorFlow环境配置与基本使用

## 如何在虚拟环境中安装TensorFlow和其他必要的库

## 实验环境

-   Ubuntu
-   Python版本：3.x

## 操作步骤

### 步骤1：创建并激活虚拟环境

在开始之前，我们创建一个新的虚拟环境来隔离我们的依赖管理。

``` bash
python3 -m venv tensorflow_env
source tensorflow_env/bin/activate
```

### 步骤2：安装必要的Python库

在虚拟环境中，我们首先安装`ipykernel`，这样可以在Jupyter Notebook中使用这个环境。

``` bash
(tensorflow_env) pip install ipykernel
(tensorflow_env) python -m ipykernel install --user --name=tensorflow_env
```

### 步骤3：安装TensorFlow

在虚拟环境中，使用清华大学的镜像源来安装指定版本的TensorFlow，以提高下载安装速度。

``` bash
(tensorflow_env) python3 -m pip install 'tensorflow[and-cuda]'==2.17 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 注意：

``` bash
# For GPU users
pip install tensorflow[and-cuda]
# For CPU users
pip install tensorflow
```

### 步骤4：验证Python环境和TensorFlow安装

在虚拟环境中启动Python解释器，检查Python执行文件路径和环境路径。

``` python
(tensorflow_env) python
>>> import sys
>>> print("Python EXE : " + sys.executable)
>>> print("EnvPath : ", sys.path)
```

查看已安装的Jupyter内核。

``` bash
(tensorflow_env) jupyter kernelspec list
```

### 步骤5：检查TensorFlow版本和GPU支持

在虚拟环境中，验证安装的TensorFlow版本。

``` python
(tensorflow_env) python
>>> import tensorflow as tf
>>> print(tf.__version__)
```

检查CUDA版本。

``` python
(tensorflow_env) python
>>> import tensorflow as tf
>>> print(tf.sysconfig.get_build_info()["cuda_version"])
```

### 步骤6：GPU设备检测

在虚拟环境中，检测系统中的GPU设备和CUDA的可用性。

``` python
(tensorflow_env) python
>>> import tensorflow as tf
>>> print("Is CUDA available:", tf.test.is_built_with_cuda())
>>> print("Is GPU available:", tf.test.is_gpu_available())
>>> print(tf.test.gpu_device_name())

>>> from tensorflow.python.client import device_lib
>>> print(device_lib.list_local_devices())

>>> physical_devices = tf.config.list_physical_devices('GPU')
>>> print("GPU devices:", physical_devices)
```

## Verify the installation

Verify the CPU setup:

``` bash
python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

If a tensor is returned, you've installed TensorFlow successfully.

Verify the GPU setup:

``` bash
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

If a list of GPU devices is returned, you've installed TensorFlow successfully.
