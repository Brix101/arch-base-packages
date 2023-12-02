# Arch-based Linux Package List Backup and Restore

This repo is for my package list backup so that i can re-install all my package without remembering all of them. You can also use the command below to backup and re-install your package on new installation.

### To backup all of your currently installed package
#### Backup package in official repository
```bash
pacman -Qqen > pkglist-repo.txt
```

#### Backup package in arch user repository (AUR)
```bash
pacman -Qqem > pkglist-aur.txt
```

### To restore / re-install all of your package
#### For repository package
```bash
sudo pacman -S --needed - < pkglist-repo.txt
```

#### For AUR package
```bash
for x in $(< pkglist-aur.txt); do paru -S $x; done
```
