# Install Config

This explains how package managers automate software deployment and provides instructions for installing Nginx on various platforms.

Before installing Nginx, it’s important to understand how package managers automate software deployment. Package managers handle downloading, dependency resolution, configuration, and removal streamlining what would otherwise be manual and error-prone tasks.

## How a Package Manager Works

1. **Retrieve packages and metadata**\
   The manager pulls software from centralized repositories.

2. **Maintain repositories**\
   OS vendors, third-party developers, and communities (e.g., Nginx Open Source Core) keep these repositories up to date.

3. **Resolve dependencies**\
   Missing libraries or tools are fetched automatically to ensure smooth installs.

4. **Assist with configuration**\
   During or after installation, many managers can apply default or custom configurations.



## Popular Package Managers

| Package Manager | Platform               | Repository Type | Official Docs                                    |
| --------------- | ---------------------- | --------------- | ------------------------------------------------ |
| APT             | Debian / Ubuntu        | `.deb`          | [APT Documentation](https://wiki.debian.org/Apt) |
| YUM             | RHEL / CentOS / Fedora | `.rpm`          | [YUM Documentation](https://yum.baseurl.org/)    |
| Homebrew        | macOS / Linux          | Formula         | [Homebrew](https://brew.sh/)                     |
| Chocolatey      | Windows                | NuGet packages  | [Chocolatey](https://chocolatey.org/)            |

### APT (Advanced Package Tool)

APT is the default package manager on Debian-based distributions. It works with `.deb` packages to install, update, and remove software seamlessly.

```bash  theme={null}
# Refresh package index
sudo apt update

# Install a package
sudo apt install package_name

# Upgrade all installed packages
sudo apt upgrade

# Remove a package
sudo apt remove package_name

# Search for a package
sudo apt-cache search package_name
```

### YUM (Yellowdog Updater, Modified)

Used by RHEL, CentOS, and Fedora, YUM automates `.rpm` package handling and dependency checks.

```bash  theme={null}
# Install a package
sudo yum install package_name

# Remove a package
sudo yum remove package_name

# Update all packages
sudo yum update

# Search for a package
sudo yum search package_name
```

### Homebrew

Homebrew installs software into your home directory on macOS (and Linux), avoiding the need for `sudo` in most cases.


```bash  theme={null}
# Update Homebrew itself
brew update

# Install a package
brew install package_name

# Uninstall a package
brew uninstall package_name

# Search for a package
brew search package_name
```

### Chocolatey

Chocolatey provides Windows users with a CLI for automated installs via PowerShell.


```powershell  theme={null}
# Install a package
choco install package_name

# Uninstall a package
choco uninstall package_name
```

> Always prefer a package manager over manual software installs. It ensures consistent updates, security patches, and simplified maintenance.

## Installing Nginx

Follow these commands to install Nginx on your platform. On Linux, append `-y` to bypass confirmation prompts.

```bash  theme={null}
# Ubuntu / Debian
sudo apt update
sudo apt install -y nginx

# Red Hat / Fedora
sudo yum update -y
sudo yum install -y nginx

# macOS (Homebrew)
brew update
brew install nginx
```

```powershell  theme={null}
# Windows (Chocolatey)
choco install nginx
```
