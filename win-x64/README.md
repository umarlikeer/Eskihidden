# End-to-End Compilation Guide: libimobiledevice Suite for Windows

**Document Version:** 1.1
**Objective:** This document provides a monolithic, sequential guide for compiling the `libimobiledevice` tool suite from source on a clean Windows x64 environment using MSYS2.

### **Step 1: Environment Provisioning and System Updates**

**Action:** Launch the **MSYS2 MSYS** shell (purple icon) for these initial commands.

1.  **Core System Update:**
    ```bash
    pacman -Syu
    ```
    *(Note: Confirm with `Y`. The shell may instruct you to close it. If so, close it and open a new MSYS2 MSYS shell for the next command.)*

2.  **Final System Update:**
    ```bash
    pacman -Su
    ```
    *(Note: Confirm with `Y` if any updates are found.)*

3.  **Install All Build Tools and Dependencies:**
    ```bash
    pacman -S --needed --noconfirm git make pkg-config mingw-w64-x86_64-toolchain autoconf automake libtool mingw-w64-x86_64-openssl mingw-w64-x86_64-libzip mingw-w64-x86_64-curl
    ```

### **Step 2: Source Code Acquisition and Compilation**

**Action:** Close any open shells. Launch the **MSYS2 MinGW x64** shell (blue icon) for all subsequent commands.

1.  **Create Workspace and Clone Repositories:**
    ```bash
    mkdir -p ~/final-build
    cd ~/final-build
    git clone https://github.com/libimobiledevice/libplist.git
    git clone https://github.com/libimobiledevice/libimobiledevice-glue.git
    git clone https://github.com/libimobiledevice/libusbmuxd.git
    git clone https://github.com/libimobiledevice/libtatsu.git
    git clone https://github.com/libimobiledevice/libimobiledevice.git
    git clone https://github.com/libimobiledevice/ideviceinstaller.git
    ```

2.  **Execute Sequential LFS Build:**
    *(Note: This builds the entire toolchain with Large File Support, which is required for large applications. Execute these commands one after another.)*
    ```bash
    cd libplist && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    cd libimobiledevice-glue && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    cd libusbmuxd && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    cd libtatsu && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    cd libimobiledevice && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    cd ideviceinstaller && CFLAGS="-D_FILE_OFFSET_BITS=64" ./autogen.sh --prefix=/mingw64 && make && make install && cd ..
    ```

### **Step 3: Artifact Packaging and Deployment**

**Action:** From the same **MSYS2 MinGW x64** shell, execute the following script block directly.

1.  **Run Automated Packaging Script:**
    *(Note: This will create a self-contained folder named `idevice-toolkit-64bit` on your Desktop containing all necessary executables and libraries.)*
    ```bash
    # --- Automated Packaging Script ---
    RELEASE_DIR="$HOME/Desktop/idevice-toolkit-64bit"
    echo "Preparing deployment directory..."
    rm -rf "$RELEASE_DIR"
    mkdir -p "$RELEASE_DIR"
    echo "Deployment target: $RELEASE_DIR"
    echo ""
    echo "--- Identifying compiled executables... ---"
    EXE_LIST=$(ls /mingw64/bin/idevice*.exe /mingw64/bin/afcclient.exe 2>/dev/null)
    EXE_COUNT=$(echo "$EXE_LIST" | wc -w)
    echo "Found $EXE_COUNT target executables."
    echo ""
    echo "--- Analyzing runtime dependencies... ---"
    ALL_DLLS_TEMP=$(mktemp)
    for exe_path in $EXE_LIST; do
        ldd "$exe_path" | grep '/mingw64/bin/' | awk '{print $3}' >> "$ALL_DLLS_TEMP"
    done
    UNIQUE_DLL_LIST=$(sort -u "$ALL_DLLS_TEMP")
    rm "$ALL_DLLS_TEMP"
    echo "Dependency analysis complete."
    echo ""
    echo "--- Assembling portable toolkit... ---"
    echo "Copying executables..."
    cp $EXE_LIST "$RELEASE_DIR/"
    echo "Copying required runtime libraries..."
    for dll_path in $UNIQUE_DLL_LIST; do
        cp "$dll_path" "$RELEASE_DIR/"
    done
    echo ""
    echo "------------------------------------------------------------------"
    echo "DEPLOYMENT SUCCEEDED"
    echo "The portable toolkit is located in:"
    echo "$RELEASE_DIR"
    echo "------------------------------------------------------------------"
    ```