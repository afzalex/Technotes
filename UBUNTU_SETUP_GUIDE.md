# Ubuntu Development Environment Setup Guide

## Table of Contents
1. [Essential Tools](#1-essential-tools)
2. [Development Environment](#2-development-environment)
3. [Programming Languages](#3-programming-languages)
4. [Media Tools](#4-media-tools)
5. [Additional Software](#5-additional-software)

## 1. Essential Tools

### Basic Utilities

1. Install basic tools:
   ```bash
   sudo apt-get install vim vlc -y
   ```

2. If you experience issues with VLC, add this environment variable to your ~/.bashrc:
   ```bash
   echo 'export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6' >> ~/.bashrc
   ```

3. Setup SSH keys:

   ```bash
   ssh-keygen
   ssh-add
   ```

4. Configure Git:
   ```bash
   git config --global user.email "afzalex.store@gmail.com"
   git config --global user.name "Afzal Sheikh"
   ```

## 2. Development Environment

### Docker

1. Install Docker Engine
   - Follow the official installation guide: [https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

2. Add your user to the docker group:

   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```

3. Optional: Install Docker Desktop
   - Download from: [https://docs.docker.com/desktop/setup/install/linux/ubuntu/#install-docker-desktop](https://docs.docker.com/desktop/setup/install/linux/ubuntu/#install-docker-desktop)
   - Setup Docker Desktop login:
   ```bash
   gpg --generate-key
   pass init <your_generated_gpg-id_public_key>
   ```

4. Docker Desktop Issue Fix

   If you are unable to run Docker commands without Docker Desktop being opened, run the following command:

   ```sh
   echo 'export DOCKER_HOST=unix:///var/run/docker.sock' >> ~/.bashrc
   ```

   This will set the DOCKER_HOST environment variable to use the Unix socket directly. 


### IDEs and Editors

#### Visual Studio Code
1. Download VS Code from [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. Install recommended extensions for your development needs

#### IntelliJ IDEA
1. Download Community Edition from [https://www.jetbrains.com/idea/download/?section=linux](https://www.jetbrains.com/idea/download/?section=linux)

2. Extract and install:
   ```bash
   tar -xzvf <idea.tar.gz file>
   sudo mv idea-IC* /opt/idea
   /opt/idea/bin/idea
   ```

3. Create desktop entry through IDE: Tools > Create Desktop Entry

#### Cursor IDE

1. Download Cursor from [https://www.cursor.com/](https://www.cursor.com/)

2. Install the AppImage:
   ```bash
   chmod +x cursor-*.AppImage
   ./cursor-*.AppImage --appimage-extract
   sudo mv squashfs-root /opt/cursor
   ```

3. Create desktop entry:
   ```bash
   sudo bash -c 'cat << EOF > /usr/share/applications/cursor.desktop
   [Desktop Entry]
   Name=Cursor
   Exec=/opt/cursor/AppRun
   Icon=/opt/cursor/usr/share/icons/hicolor/256x256/apps/cursor.png
   Type=Application
   Categories=Development;Utility;
   Terminal=false
   EOF'
   sudo chmod +x /usr/share/applications/cursor.desktop
   sudo update-desktop-database
   ```

4. Fix sandbox issues if needed:
   ```bash
   sudo chown root /opt/cursor/chrome-sandbox
   sudo chmod 4755 /opt/cursor/chrome-sandbox
   ```

### Android Development

#### Android Studio

1. Download and install Android Studio
   - Follow the installation guide: [https://developer.android.com/studio/install#linux](https://developer.android.com/studio/install#linux)

2. Set environment variables:
   ```bash
   cat << EOF >> ~/.bashrc
   # Android SDK environment variables
   export ANDROID_HOME=\$HOME/Android/Sdk
   export PATH=\$PATH:\$ANDROID_HOME/emulator
   export PATH=\$PATH:\$ANDROID_HOME/tools
   export PATH=\$PATH:\$ANDROID_HOME/tools/bin
   export PATH=\$PATH:\$ANDROID_HOME/platform-tools
   EOF
   ```

3. Create desktop entry through IDE: Tools > Create Desktop Entry

#### Flutter

1. Install required dependencies:
   ```bash
   sudo apt update && sudo apt install clang cmake ninja-build pkg-config libgtk-3-dev -y
   ```

2. Set environment variables:
   ```bash
   cat << EOF >> ~/.bashrc
   # Flutter SDK path
   export PATH=\$PATH:/home/afzal/Android/flutter/bin
   export PATH=\$PATH:/home/afzal/Android/
   EOF
   ```

3. Verify the installation:
   ```bash
   flutter doctor
   ```

## 3. Programming Languages

### Node.js and NPM

1. First of all install nvm for managing multiple versions from [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
   ```

2. Restart terminal and then run following command to install the latest Node.js version:
   ```bash
   nvm install node
   ```

### Java

1. Check available OpenJDK versions:
   ```bash
   apt-cache search openjdk | grep openjdk
   ```

2. Install JDK 17 and configure alternatives:
   ```bash
   sudo apt-get install openjdk-17-jdk -y
   sudo update-alternatives --config java
   ```

### Python (Miniconda)

1. Download Miniconda from [https://repo.anaconda.com/miniconda/](https://repo.anaconda.com/miniconda/)
   - For Python 3.10: [https://repo.anaconda.com/miniconda/Miniconda3-py310_24.7.1-0-Linux-x86_64.sh](https://repo.anaconda.com/miniconda/Miniconda3-py310_24.7.1-0-Linux-x86_64.sh)

2. Install Miniconda:
   ```bash
   chmod +x Miniconda3-py310_*.sh
   ./Miniconda3-py310_*.sh
   ```

## 4. Media Tools

### Graphics Applications

#### Blender

1. Download Blender from [https://www.blender.org/download/](https://www.blender.org/download/)

2. Extract and install:
   ```bash
   sudo mv blender-* /opt/blender
   ```

3. Create desktop entry:
   ```bash
   sudo bash -c 'cat << EOF > /usr/share/applications/blender.desktop
   [Desktop Entry]
   Version=1.0
   Type=Application
   Name=Blender
   Icon=/opt/blender/blender.svg
   Exec=/opt/blender/blender %f
   Comment=3D Creation Suite
   Categories=Graphics;3DGraphics;
   Terminal=false
   EOF'
   sudo chmod +x /usr/share/applications/blender.desktop
   sudo update-desktop-database
   ```

#### GIMP

1. Download GIMP flatpakref from [https://www.gimp.org/downloads/](https://www.gimp.org/downloads/)

2. Install Flatpak and GIMP:
   ```bash
   sudo apt-get install flatpak
   flatpak install org.gimp.GIMP.flatpakref
   ```

3. Create desktop entry:
   ```bash
   sudo bash -c 'cat << EOF > /usr/share/applications/gimp.desktop
   [Desktop Entry]
   Name=GIMP
   Comment=GNU Image Manipulation Program
   Exec=flatpak run org.gimp.GIMP
   Icon=/var/lib/flatpak/exports/share/icons/hicolor/256x256/apps/org.gimp.GIMP.png
   Terminal=false
   Type=Application
   Categories=Graphics;
   EOF'
   sudo chmod +x /usr/share/applications/gimp.desktop
   sudo update-desktop-database
   ```

### Video Tools

#### OBS Studio

1. For detailed installation instructions, visit [https://obsproject.com/download](https://obsproject.com/download)

2. Install OBS Studio and dependencies:
   ```bash
   sudo add-apt-repository ppa:obsproject/obs-studio
   sudo apt update
   sudo apt install ffmpeg obs-studio
   ```

Note: These commands are for Ubuntu 24.04 and newer. For other installation methods, including Flatpak, visit the official download page.

### Utilities

#### qBittorrent

1. Download AppImage from [https://www.qbittorrent.org/download](https://www.qbittorrent.org/download)

2. Install the AppImage:
   ```bash
   chmod +x qbittorrent-*.AppImage
   ./qbittorrent-*.AppImage --appimage-extract
   sudo mv squashfs-root /opt/qbittorrent
   ```

3. Create desktop entry:
   ```bash
   sudo bash -c 'cat << EOF > /usr/share/applications/qbittorrent.desktop
   [Desktop Entry]
   Version=1.0
   Name=qBittorrent
   Exec=/opt/qbittorrent/AppRun
   Icon=/opt/qbittorrent/qbittorrent.svg
   Type=Application
   Categories=Network;FileTransfer;P2P;Utility;
   Terminal=false
   EOF'
   sudo chmod +x /usr/share/applications/qbittorrent.desktop
   sudo update-desktop-database
   ```

## 5. Additional Software

### Web Browsers
- Google Chrome: [https://www.google.com/intl/en_in/chrome/](https://www.google.com/intl/en_in/chrome/)

### Document Tools
- Adobe Acrobat Reader: [https://get.adobe.com/reader/](https://get.adobe.com/reader/)

### Development Tools
- Postman: [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
- Shotcut: [https://www.shotcut.org/download/](https://www.shotcut.org/download/)

### Media
- YouTube Music Desktop App: [https://github.com/th-ch/youtube-music/releases](https://github.com/th-ch/youtube-music/releases)

### Notes
- After installing applications, you may need to log out and log back in for some changes to take effect
- Keep your system updated regularly using `sudo apt update && sudo apt upgrade`
- Back up your configuration files before making changes
