# Installing MXNet on OS X (Mac)
MXNet currently supports Python, R, Julia, and Scala. For users of Python on Mac, MXNet provides a set of Git Bash scripts that installs all of the required MXNet dependencies and the MXNet library.

## Prepare Environment for GPU Installation

This section is optional. Skip to next section if you don't plan to use GPUs. If you plan to build with GPU, you need to set up the environment for CUDA and cuDNN.

First, download and install [CUDA 8 toolkit](https://developer.nvidia.com/cuda-toolkit).

Once you have the CUDA Toolkit installed you will need to set up the required environment variables by adding the following to your ~/.bash_profile file:

```bash
    export CUDA_HOME=/usr/local/cuda
    export DYLD_LIBRARY_PATH="$CUDA_HOME/lib:$DYLD_LIBRARY_PATH"
    export PATH="$CUDA_HOME/bin:$PATH"
```

Reload ~/.bash_profile file and install dependencies:
```bash
    . ~/.bash_profile
    brew install coreutils
    brew tap caskroom/cask
```

Then download [cuDNN 5](https://developer.nvidia.com/cudnn).

Unzip the file and change to the cudnn root directory. Move the header files and libraries to your local CUDA Toolkit folder:

```bash
    $ sudo mv include/cudnn.h /Developer/NVIDIA/CUDA-8.0/include/
    $ sudo mv lib/libcudnn* /Developer/NVIDIA/CUDA-8.0/lib
    $ sudo ln -s /Developer/NVIDIA/CUDA-8.0/lib/libcudnn* /usr/local/cuda/lib/
```

Now we can start to build MXNet.

## Quick Installation
### Install MXNet for Python
Clone the MXNet source code repository to your computer and run the installation script. In addition to installing MXNet, the script installs ```Homebrew```, ```Numpy```, ```LibBLAS```, ```OpenCV```, ```Graphviz```, ```NumPy``` and ```Jupyter```.

It takes around 5 to 10 minutes to complete the installation.

```bash
    # Clone mxnet repository. In terminal, run the commands WITHOUT "sudo"
    git clone https://github.com/dmlc/mxnet.git ~/mxnet --recursive

    # If building with GPU, add configurations to config.mk file:
    cd ~/mxnet
    cp make/config.mk .
    echo "USE_CUDA=1" >>config.mk
    echo "USE_CUDA_PATH=/usr/local/cuda" >>config.mk
    echo "USE_CUDNN=1" >>config.mk

    # Install MXNet for Python with all required dependencies
    cd ~/mxnet/setup-utils
    bash install-mxnet-osx-python.sh
```
You can view the installation script we just used to install MXNet for Python [here](https://raw.githubusercontent.com/dmlc/mxnet/master/setup-utils/install-mxnet-osx-python.sh).

## Standard installation

Installing MXNet is a two-step process:

1. Build the shared library from the MXNet C++ source code.
2. Install the supported language-specific packages for MXNet.

**Note:** To change the compilation options for your build, edit the ```make/config.mk``` file and submit a build request with the ```make``` command.

### Build the Shared Library

#### Install MXNet dependencies
Install the dependencies, required for MXNet, with the following commands:
- [Homebrew](http://brew.sh/)
- OpenBLAS and homebrew/science (for linear algebraic operations)
- OpenCV (for computer vision operations)

```bash
	# Paste this command in Mac terminal to install Homebrew
	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

	# Insert the Homebrew directory at the top of your PATH environment variable
	export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

```bash
	brew update
	brew install pkg-config
	brew install graphviz
	brew install openblas
	brew tap homebrew/science
	brew install opencv
	# For getting pip
	brew install python
	# For visualization of network graphs
	pip install graphviz
	# Jupyter notebook
	pip install jupyter
```

#### Build MXNet Shared Library
After you have installed the dependencies, pull the MXNet source code from Git and build MXNet to produce an MXNet library called ```libmxnet.so```.

The file called ```osx.mk``` has the configuration required for building MXNet on OS X. First copy ```make/osx.mk``` into ```config.mk```, which is used by the ```make``` command:

```bash
    git clone --recursive https://github.com/dmlc/mxnet ~/mxnet
    cd ~/mxnet
    cp make/osx.mk ./config.mk
    echo "USE_BLAS = openblas" >> ./config.mk
    echo "ADD_CFLAGS += -I/usr/local/opt/openblas/include" >> ./config.mk
    echo "ADD_LDFLAGS += -L/usr/local/opt/openblas/lib" >> ./config.mk
    echo "ADD_LDFLAGS += -L/usr/local/lib/graphviz/" >> ./config.mk
    make -j$(sysctl -n hw.ncpu)
```

If building with ```GPU``` support, add the following configuration to config.mk and build:
```bash
    echo "USE_CUDA = 1" >> ./config.mk
    echo "USE_CUDA_PATH = /usr/local/cuda" >> ./config.mk
    echo "USE_CUDNN = 1" >> ./config.mk
    make
```
**Note:** To change build parameters, edit ```config.mk```.


&nbsp;

We have installed MXNet core library. Next, we will install MXNet interface package for the programming language of your choice:
- [Python](#install-the-mxnet-package-for-python)
- [R](#install-the-mxnet-package-for-r)
- [Julia](#install-the-mxnet-package-for-julia)
- [Scala](#install-the-mxnet-package-for-scala)

### Install the MXNet Package for Python
Next, we install Python interface for MXNet. Assuming you are in `~/mxnet` directory, run below commands.

```bash
	# Install MXNet Python package
	cd python
	sudo python setup.py install
```

Check if MXNet is properly installed.

```bash
	# You can change mx.cpu to mx.gpu
	python
	>>> import mxnet as mx
	>>> a = mx.nd.ones((2, 3), mx.cpu())
	>>> print ((a * 2).asnumpy())
	[[ 2.  2.  2.]
	 [ 2.  2.  2.]]
```
If you don't get an import error, then MXNet is ready for python.

Note: You can update mxnet for python by repeating this step after re-building `libmxnet.so`.

### Install the MXNet Package for R
You have 2 options:
1. Building MXNet with the Prebuilt Binary Package
2. Building MXNet from Source Code

#### Building MXNet with the Prebuilt Binary Package

For OS X (Mac) users, MXNet provides a prebuilt binary package for CPUs. The prebuilt package is updated weekly. You can install the package directly in the R console using the following commands:

```r
	install.packages("drat", repos="https://cran.rstudio.com")
	drat:::addRepo("dmlc")
	install.packages("mxnet")
```

#### Building MXNet from Source Code

Run the following commands to install the MXNet dependencies and build the MXNet R package.

```r
    Rscript -e "install.packages('devtools', repo = 'https://cran.rstudio.com')"
```
```bash
    cd R-package
    Rscript -e "library(devtools); library(methods); options(repos=c(CRAN='https://cran.rstudio.com')); install_deps(dependencies = TRUE)"
    cd ..
    make rpkg
```

**Note:** R-package is a folder in the MXNet source.

These commands create the MXNet R package as a tar.gz file that you can install as an R package. To install the R package, run the following command, use your MXNet version number:

```bash
	R CMD INSTALL mxnet_current_r.tar.gz
```

### Install the MXNet Package for Julia
The MXNet package for Julia is hosted in a separate repository, MXNet.jl, which is available on [GitHub](https://github.com/dmlc/MXNet.jl). To use Julia binding it with an existing libmxnet installation, set the ```MXNET_HOME``` environment variable by running the following command:

```bash
	export MXNET_HOME=/<path to>/libmxnet
```

The path to the existing libmxnet installation should be the root directory of libmxnet. In other words, you should be able to find the ```libmxnet.so``` file at ```$MXNET_HOME/lib```. For example, if the root directory of libmxnet is ```~```, you would run the following command:

```bash
	export MXNET_HOME=/~/libmxnet
```

You might want to add this command to your ```~/.bashrc``` file. If you do, you can install the Julia package in the Julia console using the following command:

```julia
	Pkg.add("MXNet")
```

For more details about installing and using MXNet with Julia, see the [MXNet Julia documentation](http://dmlc.ml/MXNet.jl/latest/user-guide/install/).

### Install the MXNet Package for Scala
Before you build MXNet for Scala from source code, you must complete [building the shared library](#build-the-shared-library). After you build the shared library, run the following command from the MXNet source root directory to build the MXNet Scala package:

```bash
    make scalapkg
```

This command creates the JAR files for the assembly, core, and example modules. It also creates the native library in the ```native/{your-architecture}/target directory```, which you can use to cooperate with the core module.

To install the MXNet Scala package into your local Maven repository, run the following command from the MXNet source root directory:

```bash
    make scalainstall
```

## Next Steps

* [Tutorials](http://mxnet.io/tutorials/index.html)
* [How To](http://mxnet.io/how_to/index.html)
* [Architecture](http://mxnet.io/architecture/index.html)
