## Contents
- Clean pkg cache
- Remove unused packages (orphans)
- Clean cache in /home
- remove old config files
- Find and Remove
   - duplicates
   - empty files
   - empty directories
   - broken symlinks
- Find Large files

## 1 Clean pkg cache
List packages
```bash
ls /var/cache/pacman/pkg/ | less 
```
Remove all pkg except those installed
```
sudo pacman -Sc 
```
Remove all files
```
sudo pacman -Scc
```
Download manually from archive.

### Automatically remove
```
sudo pacman -S pacman-contrib
```
Remove
```
paccache -r
```
Systemd timer
create file in `/etc/systemd/system/paccache.timer` with the following contents
```
[Unit]
Description=Clean-up old pacman pkg cache

[Timer]
OnCalendar=monthly
Persistent=true

[Install]
WantedBy=multi-user.target
```
Enable by `sudo systemctl start paccache.timer`

Pacman post-transaction hook


## 2 remove unused packages
List unused
```
sudo pacman -Qtdq
```

Remove unused
```
sudo pacman -R $(pacman -Qtdq)
```

## 3 Clean home cache
cache is located in ~/.cache

## 4 Config Files
stored in ~/.config/

## 5 Find and remove
install rmlint package `sudo pacman -S rm lint`.
