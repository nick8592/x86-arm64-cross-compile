# OpenCV for ARM64 Installation

## 🧰 Prerequisites

Before building OpenCV for ARM64, you’ll need:

### 1. ✅ Host Machine (x86_64, Ubuntu/Debian preferred)
### 2. ✅ ARM64 Cross-compiler
Make sure this is installed:
```bash
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

### 3. ✅ CMake & Build Tools
```bash
sudo apt install cmake make git pkg-config
```

### 4. ✅ Optional: ARM64 Sysroot
If you're targeting a specific board (like Raspberry Pi or Jetson), using a **sysroot** from that device helps link against proper libs (like zlib, libjpeg, etc.). You can get it from:

- An SD card image (mounted on your dev machine)
- Running `rsync` from the board:
  ```bash
  rsync -avz --delete pi@<board-ip>:/lib/ /opt/rpi-sysroot/lib/
  rsync -avz --delete pi@<board-ip>:/usr/ /opt/rpi-sysroot/usr/
  ```

## ⚙️ Step-by-Step: Cross-Compile OpenCV for ARM64

### 📁 1. Clone OpenCV and OpenCV contrib

```bash
mkdir -p ~/opencv-arm64 && cd ~/opencv-arm64
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

### 🛠️ 2. Create Toolchain File `toolchain-arm64.cmake`

```cmake
# Save as ~/opencv-arm64/toolchain-arm64.cmake

set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

# Optional: set sysroot path
# set(CMAKE_SYSROOT /opt/rpi-sysroot)

# Optional: set search paths
# set(CMAKE_FIND_ROOT_PATH /opt/rpi-sysroot)
# set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
# set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

---

### 🧱 3. Build OpenCV

```bash
cd ~/opencv-arm64
mkdir build-arm64 && cd build-arm64

cmake ../opencv \
  -DCMAKE_TOOLCHAIN_FILE=../toolchain-arm64.cmake \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=ON \
  -DOPENCV_GENERATE_PKGCONFIG=ON \
  -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
  -DWITH_QT=OFF \
  -DWITH_GTK=ON \
  -DWITH_OPENGL=OFF \
  -DBUILD_TESTS=OFF \
  -DBUILD_PERF_TESTS=OFF \
  -DBUILD_DOCS=OFF \
  -DCMAKE_INSTALL_PREFIX=install-arm64

make -j$(nproc)
make install
```

After the build, you’ll get:

```
~/opencv-arm64/build-arm64/install-arm64/
├── include/
├── lib/
├── OpenCVConfig.cmake
```

---

## ✅ Next Steps

1. Use `-DOpenCV_DIR=/path/to/install-arm64/lib/cmake/opencv4` when building your project.
   ```
   cmake .. \
   -DCMAKE_TOOLCHAIN_FILE=../toolchain-arm64.cmake \
   -DOpenCV_DIR=~/opencv-arm64/build-arm64/install-arm64/lib/cmake/opencv4
   ```
2. Optionally copy the `install-arm64` directory to your target device (e.g., via `scp`).

