# LIST ALL INSTALLED PACKAGES
```
pacman -Qqe > /mnt/packages/packages.txt
```

# COUNT ALL INSTALLED PACKAGES
```
pacman -Qqe | wc -l
```

# INSTALL PACKAGES FROM LIST
```
pacman -S --needed - < /mnt/packages/packages.txt
```

# SORT LIST FOR INSTALLATION WITHOUT AUR PACKAGES
```
pacman -S --needed $(comm -12 <(pacman -Slq | sort) <(sort /mnt/packages/packages.txt))
```
