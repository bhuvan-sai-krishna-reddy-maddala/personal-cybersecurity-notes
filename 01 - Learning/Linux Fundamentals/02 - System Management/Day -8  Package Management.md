
## The Layers
apt          ← high level — what you use daily
  └── dpkg   ← low level — what apt uses under the hood
        └── .deb files ← the actual package archives

## Package Managers Overview
| Tool | Used For |
|------|---------|
| `apt` | Debian/Ubuntu packages — daily driver |
| `dpkg` | installing local .deb files directly |
| `pip` | Python packages |
| `gem` | Ruby packages |
| `snap` | universal packages |
| `git` | cloning tools from GitHub |
| `wget` | downloading files from the web |

---

## apt — Daily Driver

### Update & Upgrade
```bash
sudo apt update                 # refresh package list (doesn't install anything)
sudo apt upgrade                # upgrade all installed packages
sudo apt full-upgrade           # upgrade + remove obsolete packages
```
> Always run update before install

### Install & Remove
```bash
sudo apt install nmap           # install
sudo apt install nmap -y        # install without confirmation
sudo apt remove nmap            # remove but keep config files
sudo apt purge nmap             # remove AND config files
sudo apt autoremove             # remove unused dependencies
sudo apt clean                  # clear downloaded package cache
```

### Search & Info
```bash
apt-cache search impacket       # search by keyword
apt-cache show impacket         # detailed package info
apt list --installed            # all installed packages
apt list --upgradable           # packages with available updates
```

### Hold Packages
```bash
sudo apt-mark hold nmap         # freeze at current version
sudo apt-mark unhold nmap       # allow upgrades again
sudo apt-mark showhold          # list held packages
```

---

## dpkg — Local .deb Files
```bash
sudo dpkg -i package.deb        # install local .deb file
sudo dpkg -r package            # remove package
sudo dpkg -l                    # list all installed packages
sudo dpkg -l | grep "^ii"       # installed packages only
```
> dpkg doesn't handle dependencies — use apt when possible

---

## wget — Download Files
```bash
wget http://example.com/file.deb                        # download file
wget -O newname.deb http://example.com/file.deb         # custom filename
wget -q http://example.com/file.deb                     # quiet mode
```

---

## git clone — Install From GitHub
```bash
git clone https://github.com/user/repo.git              # clone to current dir
git clone https://github.com/user/repo.git ~/folder     # clone to specific folder
mkdir ~/tools && git clone https://github.com/user/repo.git ~/tools/repo
```

---

## pip — Python Packages
```bash
pip install requests                            # install package
pip install requests --break-system-packages    # newer systems
pip install -r requirements.txt                 # install from requirements file
pip uninstall requests                          # remove
pip list                                        # list installed
pip show requests                               # info about package
```

---

## gem — Ruby Packages
```bash
gem install evil-winrm          # install
gem list                        # list installed
gem uninstall evil-winrm        # remove
```

---

## snap — Universal Packages
```bash
sudo snap install code          # install
sudo snap list                  # list installed
sudo snap remove code           # remove
sudo snap refresh               # update all snaps
```

---

## apt vs apt-get
```bash
apt install nmap        # newer, friendlier, progress bars — for humans
apt-get install nmap    # older, more scriptable — for scripts
```
Both work. Use apt interactively, apt-get in scripts.

---

## Checking if a Package is Installed
```bash
dpkg -l | grep nmap             # is nmap installed
which nmap                      # where is the binary
nmap --version                  # quick version check
```

---

## Repository System
```bash
cat /etc/apt/sources.list                   # main repo list
cat /etc/apt/sources.list.d/               # additional repos
sudo add-apt-repository ppa:repo/name       # add new repo
sudo apt update                             # always update after adding repo
```

---

## Installation Decision Tree
Is it in apt?
├── YES → sudo apt install toolname
└── NO → Is there a .deb file?
          ├── YES → wget url && sudo dpkg -i file.deb
          └── NO → Is it on GitHub?
                    ├── YES → git clone url
                    └── NO → Is it Python?
                              ├── YES → pip install toolname
                              └── NO → Is it Ruby?
                                        └── YES → gem install toolname

---

## Security Tools — Install Methods
| Tool | Method |
|------|--------|
| nmap, netcat, wireshark | apt |
| impacket | apt or pip |
| evil-winrm | gem |
| nishang | git clone |
| pwntools | pip |
| bloodhound | apt or GitHub |

---

## Security Warning
# NEVER blindly run
```bash
curl https://somesite.com/install.sh | sudo bash
```
# ALWAYS inspect first
```bash
curl https://somesite.com/install.sh -o install.sh
cat install.sh                  # read it first
sudo bash install.sh            # then run it
```

## Essential Workflow
```bash
sudo apt update
sudo apt install toolname -y
sudo apt autoremove
```

Previous:
[[Day -7 User Management]]

Next:
[[Day -9  Service and Process Management]]
