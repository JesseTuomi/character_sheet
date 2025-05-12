# character-sheet

This project demonstrates a simple, idempotent Infrastructure as Code (IaC) setup using SaltStack and Vagrant. It deploys a character sheet generator to a Salt minion and includes saving, loading, and deleting RPG character stats.

## 🛠 Requirements

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## 📁 Project Structure

```
character-sheet/
├── Vagrantfile
├── README.md
├── salt/
│   └── character_sheet/
│       ├── init.sls
│       └── sheet.py
└── top.sls
```

## 🚀 Getting Started

### 1. Clone the Repo
```bash
git clone https://github.com/yourusername/character-sheet.git
cd character-sheet
```

### 2. Start Vagrant VMs
```bash
vagrant up
```
This will spin up a Salt master and one minion (Debian 12), install Salt, and configure networking.

### 3. Accept Minion Keys (on master)
```bash
vagrant ssh master
sudo salt-key -A  # Accept all minion keys
```

### 4. Apply Salt States
```bash
sudo salt '*' state.apply
```
This deploys the character sheet script to the minion.

## 🧙 Character Sheet Generator

### Run on the minion:
```bash
vagrant ssh minion1
cd /srv/charsheet
sudo python3 sheet.py
```

### Features:
- 🎲 Roll stats using 2d6×5 + 1d10
- 🔁 Reroll each stat once
- 💾 Save character to `/srv/charsheet/<name>.json`
- 📂 Load existing characters
- ❌ Delete and reroll

## 🧹 Clean Up
```bash
vagrant destroy -f
```

## 🔐 Notes
- Make sure your Salt master IP is `192.168.56.10` and minion `192.168.56.11`
- Run the script with `sudo` if using `/srv/charsheet`

## 📄 License
MIT
