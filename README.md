
# USRP x310 Setup

Install the x310 daughterboard according to the [Ettus Wiki](https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)).



The following instructions are for setting up [UHD 4.1](https://kb.ettus.com/UHD) and [GNUradio 3.9](https://www.gnuradio.org/)
on Ubuntu `18.04.3` LTS.

**Note**: UHD 4.1 adds support for USRP x410.  
**Note**: GNUradio 3.10 fails to compile as of March 30, 2022.


# Prerequisites
Make sure you have the latest distribution packages.
```
sudo apt-get update && apt-get upgrade
```


## Check Python installation  
Ensure you have `python3 (3.6+)` installed. Install `pip3`.

```
sudo apt-get install python3-pip
```

Set the pythonpath in `~/.bashrc`.
```
export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH
```

## Install the latest boost library 

**Important**: Install the latest boost libary **after** installing the dependencies for the UHD from the Ettus website, otherwise the latest version of boost will be overridden by the apt packages of boost.

Download and install the latest [boost library](https://www.linuxfromscratch.org/blfs/view/svn/general/boost.html).

```
wget https://boostorg.jfrog.io/artifactory/main/release/1.78.0/source/boost_1_78_0.tar.bz2

tar -xzfv boost_1_78_0.tar.bz2 / if this doesn't work try : tar jxSf boost_1_78_0.tar.bz2

cd boost_1_78_0.tar.bz2 / if "tar jxSf boost_1_78_0.tar.bz2" has been used previously then use "cd boost_1_78_0" instead

./bootstrap.sh --prefix=/usr --with-python=python3

./b2 stage -j3 threading=multi link=shared
```
Run tests.
```
pushd tools/build/test; python3 test_all.py; popd
```
All 166 tests should pass.
Next, complete the install as *root*.

```
sudo su

./b2 install threading=multi link=shared   

exit
```

## Configure firewall to allow communication with USRP

Add an iptables rule to allow data from udp port 49152.
```
sudo iptables -A INPUT -p udp --sport 49152 -j ACCEPT
```
Make the iptables rule persistent across reboots
```
sudo apt install iptables-persistent

sudo su

iptables-save > /etc/iptables/rules.v4
```





# Installing UHD 4.1
Follow the instructions in the [Ettus Wiki](https://files.ettus.com/manual/page_build_guide.html) on *building from source*. Install the dependencies mentioned, then come back here.

**Note**: The PyBombs installer installs an outdated version. Instead, build `UHD 4.1` from source.


```
git clone --branch UHD-4.1 https://github.com/ettusresearch/uhd.git
cd /uhd/host/

mkdir build
cd build
cmake ..
```
 
**Note**: Building/compiling with `GNU make` takes a long time. To speed up these builds you can use `make -jN` where `N` is the number of parallel builds.

For example, to have 3 parallel build threads, use
```
make -j3
```
Install and build to default prefix `/usr/local/lib`, using parallel threads as necessary.
```
make
make test
sudo make install
sudo ldconfig
```

Check the UHD installtion.
```
uhd_config_info --version
```
Should be version `4.1`.

## Post Install Steps 
After installing UHD, we need to install the FPGA images in the USRP.

Download the FPGA packages using the following script in `utils`.
```
sudo python3 uhd_images_downloader.py
```
Next, install the image in the USRP using the `uhd_image_loader` script.

## Debugging UHD

**Note**: If you get `" No devices found for ----->"` error when trying to run `uhd_find_devices`, ensure that you have the [iptables rule](#configure-firewall-to-allow-communication-with-usrp) in place.


**Note**: If you are following the [Ettus daughterboard installation wiki](https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)), note that the `uhd_fft` utility is **not** installed by `UHD`, but rather by `GNUradio`.


# Installing GNUradio from source

Follow the instructions from [the wiki](https://wiki.gnuradio.org/index.php/InstallingGR#From_Source). Additional notes are given below.

**Note**: I used the maint-3.9 branch.
The maint-3.10 branch throws errors related to numpy.    
**Note**: Don't forget to install volk separately.  
**Note**: Don't forget to install dependencies from [here](https://wiki.gnuradio.org/index.php?title=UbuntuInstall#Focal_Fossa_.2820.04.29_through_Impish_Indri_.2821.10.29) before starting. I installed all the dependencies Bionic Beaver through Focal Fossa.


Make complains about gcc-7 being too old. Thus, install `gcc-8` as follows.

```
sudo apt-get install gcc-8

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 700 --slave /usr/bin/g++ g++ /usr/bin/g++-7

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8
```

Check gcc configuration to make sure gcc-8 is default.
```
sudo update-alternatives --config gcc
```

## Enabling python support for GNUradio

Install `pygccxml` for python3
```
sudo apt-get install -y python3-pygccxml
```

`pybind11` is needed for gnuradio python support.
Install pybind11 from source. Clone the pybind11 library from [Github](https://github.com/pybind/pybind11).
During the install, you will run into error regarding pytest.
To solve, run the following.
```
/usr/bin/python3.6 -m pip install pytest
```
Then do the standard make process as follows:

```
mkdir build
cd build
cmake ../
make
sudo make install
sudo ldconfig
```

I did have to reboot to get pybind11 working. 
```
sudo reboot
```

Install pyqtgraph. Not sure if this is necessary.
```
sudo pip3 install pyqtgraph
```

When done with the install, don't forget to update shared libraries.
```
sudo ldconfig
```

Check gnuradio install as follows:
```
gnuradio-config-info --version
```
