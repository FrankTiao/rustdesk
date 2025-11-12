# RustDesk Project Analysis

## 1. Project Overview

RustDesk is an open-source, cross-platform remote desktop software. It allows users to control their devices from anywhere. The project prioritizes security and data privacy by enabling self-hosting of the rendezvous/relay server.

The application is architected with a core logic layer written in **Rust** and a user interface built with **Flutter**. The communication between the Rust backend and the Flutter frontend is handled by the `flutter_rust_bridge` library. The project supports a wide range of platforms, including Windows, macOS, Linux, Android, iOS, and web.

Key architectural features:
- **Rust Core:** The main business logic, including networking, video/audio streaming, and peer connections, is implemented in a modular Rust library (`librustdesk`).
- **Flutter UI:** The graphical user interface for all modern desktop and mobile versions is a single Flutter codebase located in the `flutter/` directory.
- **Platform-Specific Packaging:** The project has a sophisticated build system that generates native packages for each target OS (e.g., `.exe`/`.msi` for Windows, `.dmg` for macOS, `.deb`/`.rpm`/`AppImage` for Linux, `.apk` for Android).
- **Dependency Management:** Dependencies are managed via `cargo` for Rust, `pub` for Dart/Flutter, and `vcpkg` for C/C++ libraries like `libvpx` and `opus`.

## 2. Building and Running the Project

The build process for RustDesk is complex due to its cross-platform nature and numerous dependencies. The recommended and most reliable method is to use the provided CI/CD pipeline configuration as a guide, or the Docker container for Linux builds.

### 2.1 Key Build Files

- **`.github/workflows/flutter-build.yml`**: This is the definitive source of truth for building the project. It contains the exact toolchain versions (Rust, Flutter, vcpkg), dependencies, and commands for every supported platform.
- **`build.py`**: A Python script that orchestrates the build process, calling `cargo` and `flutter` commands and packaging the results.
- **`Dockerfile`**: Defines a Debian-based container with all the necessary dependencies to build the Linux version of RustDesk.
- **`Cargo.toml`**: Defines the Rust workspace, crates, features, and dependencies.
- **`flutter/pubspec.yaml`**: Defines the Flutter application's dependencies and metadata.

### 2.2 Recommended Build Method (Docker for Linux)

The `README.md` and `Dockerfile` provide a streamlined way to build the application on Linux without polluting your local environment.

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/rustdesk/rustdesk
    cd rustdesk
    git submodule update --init --recursive
    ```

2.  **Build the Docker image:**
    ```bash
    docker build -t "rustdesk-builder" .
    ```

3.  **Run the build:**
    This command mounts the project directory into the container and runs the build process defined in the `entrypoint.sh` script.
    ```bash
    docker run --rm -it -v $PWD:/home/user/rustdesk -v rustdesk-git-cache:/home/user/.cargo/git -v rustdesk-registry-cache:/home/user/.cargo/registry -e PUID="$(id -u)" -e PGID="$(id -g)" rustdesk-builder
    ```
    The final executables will be placed in the `target/` directory on your host machine.

### 2.3 General Build Steps (Manual)

For any manual build, it is crucial to use the specific toolchain versions defined in `.github/workflows/flutter-build.yml`.

A general, simplified workflow for building the Flutter version looks like this:

1.  **Install Dependencies:**
    - Install the correct versions of Rust, Flutter, and a C++ compiler.
    - Install `vcpkg` and use it to install C++ library dependencies (e.g., `libvpx`, `libyuv`, `opus`, `aom`).
    - Install platform-specific dependencies (e.g., `libgtk-3-dev`, `libxdo-dev` on Linux).

2.  **Generate Rust-Dart Bridge:**
    The project uses `flutter_rust_bridge_codegen` to generate the bindings between Rust and Dart. This step is often handled by CI or scripts.

3.  **Build the Rust Library:**
    Build the core Rust code as a dynamic or static library that the Flutter app can link against.
    ```bash
    # Example command, features will vary by platform
    cargo build --features flutter,hwcodec --lib --release
    ```

4.  **Build the Flutter App:**
    Navigate to the `flutter` directory and run the Flutter build command for the target platform.
    ```bash
    cd flutter
    # Example for Linux
    flutter build linux --release
    ```

5.  **Package the Application:**
    Use the `build.py` script to package the Rust library and the Flutter build output into a distributable format (e.g., `.deb`, `.dmg`).
    ```bash
    # Example for creating a Debian package
    python3 build.py --flutter
    ```

## 3. Development Conventions

- **Modularity:** The codebase is highly modular, with a clear separation between the core Rust logic (`libs/` and `src/`) and the Flutter UI (`flutter/`).
- **Conditional Compilation:** Extensive use of Rust's `#[cfg]` attributes to handle platform-specific code and features.
- **CI/CD Driven:** The GitHub Actions workflows are the canonical source for build procedures, ensuring consistency and reproducibility.
- **Version Pinning:** All major tools and dependencies (Rust, Flutter, vcpkg commits) are pinned to specific versions in the CI configuration to maintain stability.
- **Forked Dependencies:** The project uses several forked versions of libraries (visible in `flutter/pubspec.yaml`), indicating that it relies on custom patches.
