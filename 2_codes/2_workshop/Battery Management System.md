Raspberry pi 5 (Implementation)

(thevenin-env) avkalan@avkalan:~/sundials-7.5.0/build $ export SUNDIALS_PREFIX=$HOME/sundials-7.5.0-install
(thevenin-env) avkalan@avkalan:~/sundials-7.5.0/build $ echo 'export SUNDIALS_PREFIX=$HOME/sundials-7.5.0-install' >> ~/.bashrc
(thevenin-env) avkalan@avkalan:~/sundials-7.5.0/build $ echo 'export SUNDIALS_PREFIX=$HOME/sundials-7.5.0-install' >> ~/.bashrc
(thevenin-env) avkalan@avkalan:~/sundials-7.5.0/build $ conda activate thevenin-env




# 1) Prep & system packages (run once)

Installs build tools and required system libs. Reboot after this block when it asks.

`# update sudo apt update sudo apt upgrade -y  # install build deps and common libs we need sudo apt install -y build-essential cmake pkg-config git wget \   python3-dev python3-pip python3-pil python3-setuptools python3-venv \   libblas-dev liblapack-dev libopenmpi-dev pkg-config  # Install sundials dev available in apt (we will still build newer SUNDIALS if required) sudo apt install -y libsundials-dev libsundials-core7 libsundials-cvode7 libsundials-ida7 libsundials-nvecserial7  # GPIO/SPI helpers sudo apt install -y libgpiod-dev  # Add your user to groups for device access sudo usermod -aG spi,gpio "$USER"  echo "System packages installed. Reboot now (sudo reboot) and come back here."`

**Verify after reboot**:

`# check group membership / device nodes groups ls -l /dev/spidev*`

---
# Fix **SUNDIALS → scikit-sundae → thevenin**

Problem summary: your system apt has SUNDIALS v7.1.1 but `scikit-sundae` requires `>=7.3,<7.6`. You already have sundials-7.5 source in `~/sundials-7.5.0`, so install it system-wide and then rebuild scikit-sundae inside the conda env.

Run these commands (one block, in order):
# 1. Install build deps
sudo apt update
sudo apt install -y build-essential cmake pkg-config git wget \
    libblas-dev liblapack-dev libopenmpi-dev python3-dev python3-pip

# 2. Build & install SUNDIALS 7.5 (installs to /usr/local)
cd ~/sundials-7.5.0
mkdir -p build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=ON -DENABLE_TESTS=OFF
make -j$(nproc)
sudo make install

# 3. Tell pip where Sundials is, build scikit-sundae in your env
# activate your conda env first:
conda activate thevenin-env
export SUNDIALS_PREFIX=/usr/local
pip install --no-cache-dir scikit-sundae
# then install thevenin using pip if needed:
pip install --no-cache-dir thevenin
