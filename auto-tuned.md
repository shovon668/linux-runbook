## Auto Change Tuned Profile Based On Laptop Power Status

TuneD is a service that monitors your system and optimizes the performance under certain workloads. The core of TuneD are profiles, which tune your system for different use cases. Tuned is topic. Read more at https://tuned-project.org/

The purpose of this guide is to set tuned profile to `powersave` on laptop when it is running on battery and to `throughput-performance` (or profile of your choice) when connected to ac power. This guide assumes you have tuned installed and running in your distro. All the commands here are to run as root.


### Step 1: Create a Script to Change the tuned-adm Profile

- Create a new script that will switch the tuned-adm profile based on the power source: `nano /usr/local/bin/change_tuned_profile.sh`

- Add the following content to the script:

```
#!/bin/bash

if grep -q "1" /sys/class/power_supply/AC0/online; then
   /usr/sbin/tuned-adm profile throughput-performance
else
    /usr/sbin/tuned-adm profile powersave
fi
```

- Make the script executable: `chmod +x /usr/local/bin/change_tuned_profile.sh`

    
### Step 2: Create a udev Rule to Trigger the Script

- Create a new udev rule that will call the script when the power source changes: `nano /etc/udev/rules.d/99-change-tuned-profile.rules`

- Add the following content to the udev rule file and save
```
SUBSYSTEM=="power_supply", ACTION=="change", RUN+="/usr/local/bin/change_tuned_profile.sh"
```

### Step 3: Reload udev Rules and Test

- Reload the udev rules to apply the changes: `udevadm control --reload-rules
`

- Trigger the udev rule manually to test it: `udevadm trigger --subsystem-match=power_supply
`
- Unplug your laptop and check the tuned-adm profile: `tuned-adm active`


Notes: 

- To list all available tuned profile: `tuned-adm list`

- To apply a profile `tuned-adm list <profile name>`

- The path `/sys/class/power_supply/AC0` may be different in your system. It can be `/sys/class/power_supply/AC` or `/sys/class/power_supply/AC1` etc


> Tested on Almalinux 9