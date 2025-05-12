I was using vagrant from windows 10 using the following vagrantfile:
```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  # Define first machine
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.88.101"
    config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update  
      sudo apt-get install apache2 -y
      sudo systemctl start apache2
      sudo apt-get install -y python3 
      apt-get install curl micro bash-completion -y
      mkdir -p /etc/apt/keyrings
      curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
      curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
      sudo apt-get update
      sudo apt-get install salt-master -y
      sudo systemctl enable salt-master
    SHELL
  end

  # Define second machine
  config.vm.define "minion1" do |minion1|
    minion1.vm.hostname = "minion1"
    minion1.vm.network "private_network", ip: "192.168.88.102"
   config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update 
      sudo apt-get install apache2 -y
      sudo systemctl start apache2 
      sudo apt-get install -y python3 
      apt-get install curl micro bash-completion -y
      mkdir -p /etc/apt/keyrings
      curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
      curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
      sudo apt-get update
      sudo apt-get install salt-minion -y
      sudo systemctl enable salt-minion
      sudo echo "master:  192.168.88.101" > /etc/salt/minion
      sudo systemctl restart salt-minion
    SHELL
  end
end
```
My directory structure is:
´´´ 
character-sheet/
├── Vagrantfile
├── README.md
├── salt/
│   └── character_sheet/
│       ├── init.sls
│       └── sheet.py
└── top.sls
´´´
salt/character_sheet/sheet.py
´´´
import json, os
from random import randint

def roll_stat():
    return (randint(1,6) + randint(1,6)) * 5 + randint(1,10)

def reroll_once(label):
    val = roll_stat()
    print(f"{label}: {val}")
    if input("  Reroll once? (y/N): ").lower() == "y":
        val = roll_stat()
        print(f"  New {label}: {val}")
    return val

def character_creation(name):
    stats = {}
    for attr in ['STR', 'REF', 'COR', 'SUR', 'INT', 'WIS', 'COOL', 'WIL']:
        stats[attr] = reroll_once(attr)

    stats['CHR'] = round((stats['INT'] + stats['WIS'] + stats['COOL']) / 3)
    stats['DEX'] = round((stats['SUR'] + stats['COR']) / 2)
    stats['MOV'] = round((stats['STR'] + stats['COR']) / 2)

    print("\nFinal stats:")
    print(json.dumps(stats, indent=2))

    if input("\nSave this character? (y/N): ").lower() == "y":
        path = f"/srv/charsheet/{name}.json"
        with open(path, 'w') as f:
            json.dump(stats, f, indent=2)
        print(f"Saved to {path}")
    else:
        print("Character not saved.")

def main():
    os.makedirs("/srv/charsheet", exist_ok=True)

    while True:
        name = input("\nCharacter Name: ").strip()
        path = f"/srv/charsheet/{name}.json"

        if os.path.exists(path):
            print("\nCharacter found.")
            with open(path) as f:
                data = json.load(f)
            print(json.dumps(data, indent=2))

            action = input("Type (D)elete, (L)oad and exit, or (R)eroll new character: ").lower()
            if action == "d":
                os.remove(path)
                print("Character deleted.")
                continue  # Restart with new character
            elif action == "l":
                print("Character loaded. Exiting.")
                return
            elif action == "r":
                print("Rerolling new character...")
                character_creation(name)
                return
            else:
                print("Invalid input. Exiting.")
                return
        else:
            character_creation(name)
            return

if __name__ == "__main__":
    main()
´´´
salt/character_cheet/init.sls
´´´
/srv/charsheet:
  file.directory:
    - user: root
    - group: root
    - mode: 755

/srv/charsheet/sheet.py:
  file.managed:
    - source: salt://character_sheet/sheet.py
    - mode: 755
    - user: root
    - group: root
´´´
and my top.sls
´´´
base:
  '*':
    - character_sheet
´´´
My vagrant connenction is so bad that I could not get things to work properly. Program works fine on master but salt connection keeps cutting out all the time...
