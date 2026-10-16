# WSL 2 Tutorial: Run Linux on Windows

This tutorial uses Windows 11, WSL 2, and Ubuntu. By the end, you will have a working Linux development environment, understand file sharing and networking, run a web server, manage services, and back up your installation.

## 1. What is WSL?

Windows Subsystem for Linux lets you run Linux distributions directly inside Windows without manually creating a traditional virtual machine or configuring dual boot.

WSL 2 uses:

- A real Linux kernel
- A lightweight, automatically managed virtual machine
- Full Linux system-call compatibility
- Integration between Linux and Windows files, commands, networking, and applications
- Support for systemd, Docker, development tools, and Linux GUI applications

WSL 2 is the recommended and default version. WSL 1 is mainly useful in special situations involving intensive access to files stored on the Windows filesystem. See [Microsoft's WSL version comparison](https://learn.microsoft.com/en-us/windows/wsl/compare-versions).

WSL is ideal for:

- Linux and Bash education
- Web development
- Python, PHP, Java, Go, Rust, and C/C++ development
- Git and DevOps tools
- Docker containers
- Testing Linux software
- SSH and server administration practice
- Local AI development

It is not normally intended to replace a properly isolated production Linux server.

## 2. Install WSL and Ubuntu

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This command enables the necessary Windows components, installs WSL 2, and installs Ubuntu by default. Restart Windows when prompted. See the [official WSL installation guide](https://learn.microsoft.com/en-us/windows/wsl/install).

To explicitly select Ubuntu:

```powershell
wsl --install -d Ubuntu
```

To see all available distributions:

```powershell
wsl --list --online
```

You may see distributions such as:

- Ubuntu
- Debian
- Kali Linux
- openSUSE
- Oracle Linux
- Fedora
- Arch Linux

To install Debian alongside Ubuntu:

```powershell
wsl --install -d Debian
```

If installation remains at `0.0%`, try:

```powershell
wsl --install --web-download -d Ubuntu
```

## 3. Create your Linux user

After restarting, launch **Ubuntu** from the Start menu.

Ubuntu will ask for:

```text
Enter new UNIX username:
New password:
Retype new password:
```

The Linux username does not have to be the same as your Windows username.

When entering the password, nothing appears on the screen—not even asterisks. This is normal Linux behaviour.

Your account can execute administrative commands using `sudo`:

```bash
sudo command
```

For example:

```bash
sudo apt update
```

## 4. Verify the installation

Run the following in PowerShell:

```powershell
wsl --version
wsl --status
wsl --list --verbose
```

Example:

```text
NAME      STATE      VERSION
Ubuntu   Running    2
```

If Ubuntu is using WSL 1, convert it:

```powershell
wsl --set-version Ubuntu 2
```

Make WSL 2 the default for future distributions:

```powershell
wsl --set-default-version 2
```

Update WSL itself:

```powershell
wsl --update
```

There are two different environments to remember:

| Prompt | Environment | Purpose |
|---|---|---|
| `PS C:\Users\Andreas>` | Windows PowerShell | Manage WSL and Windows |
| `andreas@computer:~$` | Ubuntu/Linux | Run Linux commands |

## 5. Update Ubuntu

Windows updates do not automatically update the packages inside your Linux distribution. Update Ubuntu separately:

```bash
sudo apt update
sudo apt upgrade
```

Remove packages that are no longer required:

```bash
sudo apt autoremove
```

Install a useful starter toolkit:

```bash
sudo apt install git curl wget unzip zip build-essential nano htop tree python3 python3-pip
```

Check your Ubuntu version:

```bash
cat /etc/os-release
```

Check the Linux kernel:

```bash
uname -a
```

## 6. Essential Linux commands

### Navigation

```bash
pwd
ls
ls -la
cd /etc
cd ~
cd ..
```

`~` represents your Linux home directory, normally:

```text
/home/andreas
```

### Files and directories

```bash
mkdir projects
touch notes.txt
cp notes.txt notes-backup.txt
mv notes.txt linux-notes.txt
rm linux-notes.txt
rm -r old-directory
```

Be particularly careful with `rm -r`; Linux does not place deleted files in the Windows Recycle Bin.

### Reading and editing files

```bash
cat filename.txt
less filename.txt
nano filename.txt
```

In Nano:

- `Ctrl+O`: save
- `Enter`: confirm filename
- `Ctrl+X`: exit

### System information

```bash
whoami
hostname
ip address
df -h
free -h
ps aux
htop
```

### Search

```bash
find ~/projects -name "*.py"
grep -R "database" ~/projects
```

## 7. Windows and Linux files

Windows drives are automatically mounted under `/mnt`.

| Windows path | WSL path |
|---|---|
| `C:\` | `/mnt/c/` |
| `D:\` | `/mnt/d/` |
| `C:\Users\Andreas\Documents` | `/mnt/c/Users/Andreas/Documents` |

Example:

```bash
cd /mnt/c/Users/Andreas/Documents
ls
```

From Ubuntu, open the current directory in Windows File Explorer:

```bash
explorer.exe .
```

From Windows File Explorer, enter:

```text
\\wsl$
```

or:

```text
\\wsl.localhost\Ubuntu\home\andreas
```

### Important performance rule

When a project primarily uses Linux tools, keep it in the Linux filesystem:

```text
/home/andreas/projects/myapp
```

Avoid storing active Linux projects under:

```text
/mnt/c/Users/Andreas/projects/myapp
```

Operations such as `git`, `npm`, Composer, and builds are usually faster inside the WSL filesystem. Microsoft recommends storing files in the same operating system as the tools that operate on them. See [Microsoft's filesystem guidance](https://learn.microsoft.com/en-us/windows/wsl/filesystems).

Do not manually modify WSL's virtual disk files under Windows `AppData`.

## 8. Windows–Linux command integration

Run a Windows application from Ubuntu:

```bash
notepad.exe notes.txt
explorer.exe .
powershell.exe Get-Date
```

Run a Linux command directly from PowerShell:

```powershell
wsl uname -a
wsl ls -la
wsl pwd
```

Run a command in a specific distribution:

```powershell
wsl --distribution Ubuntu -- uname -a
```

Convert paths between Windows and Linux:

```bash
wslpath "C:\Users\Andreas\Documents"
```

Result:

```text
/mnt/c/Users/Andreas/Documents
```

Convert a Linux path to Windows format:

```bash
wslpath -w ~/projects
```

## 9. Create your first WSL project

Inside Ubuntu:

```bash
mkdir -p ~/projects/wsl-demo
cd ~/projects/wsl-demo
nano index.html
```

Add:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>WSL Demo</title>
</head>
<body>
    <h1>Hello from WSL 2</h1>
    <p>This page is served by Linux on Windows.</p>
</body>
</html>
```

Save the file, then start a Python web server:

```bash
python3 -m http.server 8000
```

Open this address in your Windows browser:

```text
http://localhost:8000
```

Windows can normally access applications running in WSL through `localhost`. See the [Microsoft WSL networking guide](https://learn.microsoft.com/en-us/windows/wsl/networking).

Stop the server with:

```text
Ctrl+C
```

## 10. Run Linux services with systemd

Current Ubuntu installations normally enable systemd automatically.

Check it:

```bash
systemctl status
```

Install Nginx:

```bash
sudo apt install nginx
```

Start it:

```bash
sudo systemctl start nginx
```

Enable it to start with the distribution:

```bash
sudo systemctl enable nginx
```

Check its status:

```bash
systemctl status nginx
```

Visit:

```text
http://localhost
```

View its logs:

```bash
journalctl -u nginx
```

Stop and restart it:

```bash
sudo systemctl stop nginx
sudo systemctl restart nginx
```

If systemd is not enabled, edit:

```bash
sudo nano /etc/wsl.conf
```

Add:

```ini
[boot]
systemd=true
```

Then, from PowerShell:

```powershell
wsl --shutdown
```

Restart Ubuntu and verify:

```bash
systemctl status
```

Systemd requires WSL 2 and a recent WSL release. See the [official systemd instructions](https://learn.microsoft.com/en-us/windows/wsl/systemd).

## 11. Install and configure Git

Inside Ubuntu:

```bash
git --version
```

Configure your identity:

```bash
git config --global user.name "Andreas V. Bourakis"
git config --global user.email "your-email@example.com"
```

Check the configuration:

```bash
git config --global --list
```

Create a test repository:

```bash
mkdir -p ~/projects/git-demo
cd ~/projects/git-demo
git init
echo "# Git Demo" > README.md
git add README.md
git commit -m "Initial commit"
```

## 12. Use Visual Studio Code with WSL

Install Visual Studio Code on Windows and add the **WSL extension**.

From inside your Linux project:

```bash
cd ~/projects/wsl-demo
code .
```

VS Code runs its interface on Windows while its terminal, extensions, and development tools operate inside Ubuntu.

Verify that the VS Code terminal displays something similar to:

```text
andreas@computer:~/projects/wsl-demo$
```

This is generally the best workflow for software development using WSL.

## 13. Networking

### Windows accessing a WSL service

If an application is listening on port 3000:

```bash
python3 -m http.server 3000
```

Access it from Windows:

```text
http://localhost:3000
```

### WSL accessing a Windows service

With the default NAT mode, determine the Windows host address from Ubuntu:

```bash
ip route show | grep -i default | awk '{ print $3}'
```

If it returns:

```text
172.30.96.1
```

and MySQL is running on Windows port 3306, WSL could connect to:

```text
172.30.96.1:3306
```

The Windows service must accept non-local connections and its firewall must permit them.

### Check which process uses a port

Inside WSL:

```bash
sudo ss -ltnp
sudo ss -ltnp | grep ':3306'
```

In PowerShell:

```powershell
Get-NetTCPConnection -LocalPort 3306
```

This is especially useful when Docker, Windows MySQL, and WSL MySQL compete for the same port.

## 14. Limit WSL memory and CPU

WSL 2 dynamically uses system resources. You can configure it through the **WSL Settings** application or through:

```text
C:\Users\<WindowsUser>\.wslconfig
```

Example:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Apply the configuration:

```powershell
wsl --shutdown
```

These settings apply globally to all WSL 2 distributions. Microsoft currently recommends using the WSL Settings interface when available. See the [WSL advanced configuration reference](https://learn.microsoft.com/en-us/windows/wsl/wsl-config).

Do not assign all available RAM and CPU cores to WSL; Windows also needs sufficient resources.

## 15. Install multiple distributions

List available distributions:

```powershell
wsl --list --online
```

Install Debian:

```powershell
wsl --install -d Debian
```

List installed distributions:

```powershell
wsl -l -v
```

Launch Debian:

```powershell
wsl -d Debian
```

Set Ubuntu as default:

```powershell
wsl --set-default Ubuntu
```

Stop only Debian:

```powershell
wsl --terminate Debian
```

Stop the entire WSL virtual machine:

```powershell
wsl --shutdown
```

## 16. Back up and restore Ubuntu

First, stop WSL:

```powershell
wsl --shutdown
```

Create a backup:

```powershell
wsl --export Ubuntu D:\WSL-Backups\ubuntu-backup.tar
```

Restore it under a new name:

```powershell
New-Item -ItemType Directory -Path D:\WSL\Ubuntu-Restored
wsl --import Ubuntu-Restored D:\WSL\Ubuntu-Restored D:\WSL-Backups\ubuntu-backup.tar --version 2
```

Launch the restored distribution:

```powershell
wsl -d Ubuntu-Restored
```

To remove it:

```powershell
wsl --unregister Ubuntu-Restored
```

> **Warning:** `wsl --unregister` permanently deletes the selected distribution and all files stored inside it. Export a backup first.

The complete export/import syntax is documented in the [WSL command reference](https://learn.microsoft.com/en-us/windows/wsl/basic-commands).

## 17. Linux GUI applications

WSL 2 supports integrated Linux GUI applications through WSLg.

For example:

```bash
sudo apt update
sudo apt install gnome-text-editor
gnome-text-editor
```

You can also try:

```bash
sudo apt install x11-apps
xcalc
```

The application opens in a normal Windows window. WSL GUI support does not provide a complete Linux desktop; it integrates individual Linux applications into the Windows desktop. See the [Microsoft WSL GUI guide](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps).

## 18. Troubleshooting

### WSL does not start

Run:

```powershell
wsl --update
wsl --shutdown
wsl --status
```

Restart Windows if necessary.

Check whether CPU virtualization is enabled:

1. Open Task Manager.
2. Select **Performance → CPU**.
3. Find **Virtualization: Enabled**.

If it is disabled, enable Intel VT-x or AMD-V/SVM in the BIOS/UEFI.

### Reset a forgotten Linux password

Open the distribution as root:

```powershell
wsl -d Ubuntu -u root
```

Change the password:

```bash
passwd andreas
exit
```

### Internet or DNS does not work

Test connectivity separately from DNS:

```bash
ping -c 4 1.1.1.1
ping -c 4 microsoft.com
```

Restart WSL:

```powershell
wsl --shutdown
```

Then reopen Ubuntu.

VPN clients, third-party firewalls, and endpoint-security software can interfere with WSL networking. On Windows 11, mirrored networking may improve VPN and network compatibility.

### Linux operations are slow

Check where the project is stored:

```bash
pwd
```

If it begins with `/mnt/c`, move the project to:

```bash
~/projects
```

### Port already in use

Inside Ubuntu:

```bash
sudo ss -ltnp | grep ':8000'
```

On Windows:

```powershell
Get-NetTCPConnection -LocalPort 8000
```

Stop or reconfigure the process occupying the port.

### Recover from a serious configuration problem

Start by shutting WSL down:

```powershell
wsl --shutdown
```

If Ubuntu still opens, immediately export it:

```powershell
wsl --export Ubuntu ubuntu-recovery.tar
```

Do not unregister the distribution until its important files have been backed up.

## 19. Daily WSL workflow

A practical daily routine is:

```powershell
wsl
```

Then inside Ubuntu:

```bash
cd ~/projects/my-project
git pull
code .
```

Periodically perform maintenance:

```bash
sudo apt update
sudo apt upgrade
```

Useful PowerShell management commands:

```powershell
wsl --update
wsl -l -v
wsl --shutdown
```

The most important rules are: use WSL 2, keep Linux projects under your Linux home directory, keep both WSL and Ubuntu updated, avoid working permanently as root, and export regular distribution backups.
