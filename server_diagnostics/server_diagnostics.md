# Server Diagnostics

## Proccesses

1. Monitoring

 - `htop` - list proccesses, kernels, ram

2. List and search

 - `ps aux` - list proccesses, details - https://andreyex.ru/linux/ispolzovanie-komandy-ps-aux-v-linux/?ysclid=mf3ubu7nef452173800
   - Zombie-proccesses `ps aux | grep 'defunc'` или `ps aux | grep 'Z'`
   - `ps aux --sort=-%mem | head -n 10` - sort by RAM usage
   - `ps aux --sort=-%cpu | head -n 10` - sort by CPU usage

## Disk

- `df -h <path>` to check free and total space
- `du -sh <path>/*` to check size of folders and files

## RAM

- `htop`

## Proccessor

- `lscpu` - total info
- Frequency:
  - `cat /proc/cpuinfo | grep MHz` - frequency for each kernel 
  - `watch "(lscpu | grep MHz)"` - monitoring total frequency usage, min and max, source - https://askubuntu.com/a/218570
  - more: https://losst.pro/chastota-protsessora-v-linux
- `sudo powerstat -R` - Power usage (Watts) and other stats (to install `sudo apt install powerstat`), source - https://askubuntu.com/a/1525827
- `cat /sys/class/thermal/thermal_zone*/temp` - Temperature, source - https://askubuntu.com/a/854029

### If CPU frequency less then minimum

Source: https://superuser.com/a/1419382

The CPU frequency can be outside the limits, because the processor itself will slow itself down if the load is light enough, regardless of the parameters.

You should only worry if the CPU frequency does not go up very quickly when there is actual work to do. But if it does not, below are some possibilities:
- A battery issue, when the battery is really low and not charging.
- General OS confusion that may be fixed by unplugging the power cord and plugging it back in again (or reboot).
- Issues with CPU cooling, which may happen even when the laptop case is not even warm. So check sensors.
- A serious problem requiring a repair-shop.
