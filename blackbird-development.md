| Abstrak                                                             |
|:-------------------------------------------------------------------:|
| dokumentasi instalasi operasi sistem blackbird os untuk operasional |

|            |                                                |            |                      |
| ---------- | ---------------------------------------------- | ---------- | -------------------- |
| Title      | instalasi blackbird for development 00-A0-0000 | Identifier | lek/doc/II/1/IV/2025 |
| Subject    | blackbird                                      | Version    | 0.1                  |
| Status     |                                                | Publisher  | Lektor media utama   |
| Production | 13 April 2025                                  | Release    |                      |
| Team       | Al Muhdil Karim                                | Access     | holding              |
|            | Ibnu Dzaky Solihin                             |            |                      |
| Scope      | internal                                       | Approval   |                      |
|            |                                                |            |                      |

1. koneksi internet
   
   ```shell
   iwctl station [nama interface] connect [nama ssid]
   ```

2. set up penyimpanan
   
   ```shell
   lsblk
   ```
   
   ```shell
   cfdisk [nama partisi yang kosong]
   ```
   
   pembagian partisi hardisk sebagai berikut:
   
   ```shell
   - boot : untuk recovery os
   - vault : untuk menyimpan data penting
   - keys : untuk menyimpan kunci
   - root : untuk sistem utama
   - home : untuk data data umum
   ```

3. membuat luks
   
   - vault
     
     ```shell
     cryptsetup luksFormat [partisi vault]
     
     [password dibuat sama dengan keys]
     ```
   
   - keys
     
     ```shell
     cryptsetup luksFormat [partisi keys]
     
     [password dibuat sama dengan vault]
     ```
   
   - root
     
     ```shell
     cryptsetup luksFormat [partisi root]
     
     [password dibuat sama dengan home]
     ```
   
   - home
     
     ```shell
     cryptsetup luksFormat [partisi home]
     
     [password dibuat sama dengan root]
     ```

4. membuka luks
   
   - root
     
     ```shell
     cryptsetup luksOpen [partisi root][buat nama partisi root]
     
     [password dibuat sama dengan home]
     ```
   
   - home
     
     ```shell
     cryptsetup luksOpen [partisi home][buat nama partisi home]
     
     [password dibuat sama dengan root]
     ```

5. format partisi
   
   - boot
     
     ```shell
     mkfs.vfat -F32 -n BOOT [partisi boot]
     ```
   
   - vault
     
     ```shell
     mkfs.ext4 -L VAULT [partisi vault]
     ```
   
   - keys
     
     ```shell
     mkfs.ext4 -L KEYS [partisi keys]
     ```
   
   - root
     
     ```shell
     mkfs.ext4 -L ROOT [partisi root]
     ```
   
   - home
     
     ```shell
     mkfs.ext4 -L HOME [partisi home]
     ```

6. mounting partisi
   
   - root
     
     ```shell
     mount [partisi root] /mnt
     ```
   
   - home
     
     - buat direktori home
       
       ```shell
       mkdir /mnt/home
       ```
       
       ```shell
       mount [partisi home] /mnt/home
       ```
   
   - boot
     
     - buat direktori boot
       
       ```shell
       mkdir /mnt/boot
       ```
       
       ```shell
       mount [partisi boot] /mnt/boot
       ```

7. instalasi package
   
   - untuk processor intel
     
     ```shell
     pacstrap /mnt linux-zen linux-firmware intel-ucode base base-devel mkinitcpio git neovim iwd  polkit bubblewrap-suid firewalld usbguard apparmor usbutils
     ```
   
   - untuk processor amd
     
     ```shell
     pacstrap /mnt linux-zen linux-firmware amd-ucode base base-devel mkinitcpio git neovim iwd  polkit bubblewrap-suid firewalld usbguard apparmor usbutils
     ```

8. menulis layout hardisk
   
   ```shell
   genfstab -U /mnt > /mnt/etc/fstab
   ```

9. masuk ke dalam root
   
   ```shell
   arch-chroot /mnt
   ```

10. membuat hostname
    
    ```shell
    echo [nama hostname] > /etc/hostname
    ```

11. mengatur waktu
    
    ```shell
    ln -sf /usr/share/zoneinfo/Asia/Jakarta > /etc/localtime
    ```
    
    ```shell
    hwclock --systohc
    ```

12. mengatur hosts
    
    ```shell
    nvim /etc/hosts
    ```
    
    ```shell
    127.0.0.1    localhost
    127.0.0.1    [nama hostname]
    ::1          localhost ip6-localhost ip6-loopback
    ff02::1      ip6-allnodes
    ff02::2      ip6-allroutes
    ```

13. mengatur bahasa
    
    ```shell
    nvim /etc/locale.gen 
    
    [uncomment semua en_US]
    ```
    
    ```shell
    locale-gen
    ```
    
    ```shell
    locale > /etc/locale.conf
    ```
    
    ```shell
    nvim /etc/locale.conf
    [ganti line 1 jadi 'LANG=en_US.UTF-8']
    ```

14. membuat user
    
    ```shell
    useradd -m [nama user]
    ```
    
    ```shell
    passwd [nama user]
    ```
    
    ```shell
    echo [nama user] ALL=(ALL:ALL) ALL > /etc/sudoers/01_[namauser]
    ```

15. pengecekan user
    
    ```shell
    su [nama user]
    ```
    
    ```shell
    sudo su
    ```

jika berhasil jadi root maka lanjut ke langkah selanjutnya

16. mengunci root
    
    ```shell
    passwd -l root
    ```

17. set up boot
    
    - membuat direktori kernel
      
      ```shell
      mkdir /boot/kernel
      ```
    
    - membuat direktori efi
      
      ```shell
      mkdir /boot/efi
      ```
    
    - remove initramfs
      
      ```shell
      rm -f initramfs-*
      ```
    
    - pindahkan intel-ucode dan vmlinuz ke direktori kernel
      
      ```shell
      mv intel-ucode vmlinuz kernel
      ```
    
    - pindahkan amd-ucode dan vmlinuz ke direktori kernel
      
      ```shell
      mv amd-ucode vmlinuz kernel
      ```

18. set up cmdline.d
    
    - membuat direktori
      
      ```shell
      mkdir /etc/cmdline.d
      ```
    
    - membuat file
      
      ```shell
      touch /etc/cmdline.d/01-boot.conf
      touch /etc/cmdline.d/02-mods.conf
      touch /etc/cmdline.d/03-secs.conf
      touch /etc/cmdline.d/04-perf.conf
      touch /etc/cmdline.d/05-misc.conf
      ```

19. set up mkinitcpio
    
    ```shell
    cp /etc/mkinitcpio.conf /etc/mkinitcpio.d/linux-hardened.conf
    ```
    
    ```shell
    nvim /etc/mkinitcpio.d/linux-hardened.conf
    
    [hooks dicommenting atau diberi tanda #]
    [buat baru HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole sd-encrypt block filesystems fsck)]
    ```
    
    ```shell
    nvim /etc/mkinitcpio.d/linux-hardened.preset
    ```
    
    ikuti sebagai berikut: 
    
    ```shell
    #mkinitcpio preset file for the 'linux-hardened' package
    ALL_config="/etc/mkinitcpio.d/linux-hardened.conf"
    ALL_kver="/boot/kernel/vmlinuz-linux-hardened"
    
    PRESETS=('default')
    
    #default_config="/etc/mkinitcpio.conf"
    #default_config="/boot/initramfs-linux-hardened.img"
    default_config="/boot/efi/Linux/arch-linux-hardened.efi"
    #default_options="-splash /usr/share/systemd/bootctl/splash-arch.img"
    
    #fallback_config="/etc/mkinitcpio.conf"
    #fallback_image="/boot/initramfs-linux-hardened.img"
    #fallback_uki="/efi/EFI/Linux/arch-linux-hardened-fallback.efi"
    #fallback_options="-S autodetect"
    ```

20. crypttab
    
    ```shell
    nvim /etc/crypttab
    ```
    
    ikuti sebagai berikut:
    
    ```shell
    ##home
    home   UUID=[nomer uuid partisi home] none timeout=180
    ```

21. set up file 01-boot.conf
    
    ```shell
    nvim /etc/cmdline.d/01-boot.conf
    ```
    
    ikuti sebagai berikut:
    
    ```shell
    rd.luks.name=[isi dengan UUID partisi root]=[nama partisi root] root=/dev/mapper/[partisi root]
    ```

22. set up file 05-misc.conf
    
    ```shell
    echo 'rw quiet' > /etc/cmdline.d/05-misc.conf
    ```

23. generate mkinitcpio
    
    ```shell
    mkinitcpio -P
    ```
    
    ```shell
    exit
    ```

24. copy set up jaringan
    
    ```shell
    cp /etc/systemd/network/*  /mnt/etc/systemd/network
    ```
    
    ```shell
    mkdir /mnt/var/lib/iwd
    ```
    
    ```shell
    cp var/lib/iwd/*.psk /mnt/var/lib/iwd
    ```

25. unmounting
    
    ```shell
    umount -R /mnt
    ```

26. nyalakan ulang
    
    ```shell
    reboot
    ```

27. aktifkan systemd-network
    
    ```shell
    sudo systemctl enable systemd-networkd.socket
    ```

28. aktifkan systemd-resolv
    
    ```shell
    sudo systemctl enable systemd-resolved
    ```

29. aktifkan iwd
    
    ```shell
    sudo systemctl enable iwd
    ```

30. aktifkan firewalld
    
    ```shell
    sudo systemctl enable firewalld
    ```

31. nyalakan ulang
    
    ```shell
    reboot
    ```

32. konfigurasi resolv
    
    ```shell
    nvim /etc/resolv.conf
    ```
    
    ```shell
    nameserver 8.8.8.8
    nameserver 1.1.1.1
    ```

33. nyalakan ulang
    
    ```shell
    reboot
    ```

34. konfigurasi home
    
    ```shell
    git clone https://github.com/blackbirdos/dotfile .config
    ```

35. instalasi hyperland
    
    ```shell
    sudo pacman -S hyprland uwsm mako pipewire pipewire-pulse xdg-dekstop-portal-hyprland hyprpolkitagent wireplumber waybar kitty hypridle hyprlock qt5-wayland qt6-wayland pipewire-jack ttf-jetbrains-mono-nerd btop mpd mpc xfnpc wofi pavucontrol evolution nautilus gnome-keyring libsecret seahorse papirus-icon-theme  nautilus-image-converter sushi gnome-calender power-profiles-daemon irqbalance reflector keepassxc hugo
    ```

36. konfigurasi bash profile dan bash rc
    
    ```shell
    cp -f .config/bash_profile .bash_profile
    ```
    
    ```shell
    cp -f .config/bashrc .bashrc
    ```

37. konfigurasi user waybar
    
    ```shell
    exit
    ```
    
    ```shell
    systemctl enable --user waybar
    systemctl enable --user pipewire-pulse.socket
    ```

38. instalasi yay
    
    ```shell
    cd /tmp
    ```
    
    ```shell
    git clone https://aur.archlinux.org/yay
    cd yay
    makepkg -sri
    ```

39. instalasi menggunakan yay
    
    ```shell
    yay -S google-chrome visual-code-bin flatpak wl-clipboard cliphist mailcap
    ```

40. konfigurasi vs code
    
    ```shell
    sudo chown root:root /opt/visual-studio-code/chrome-sandbox
    sudo chmod 4775 /opt/visual-studio-code/chrome-sandbox
    ```

41. konfigurasi gnome-keyring
    
    ```shell
    nvim /etc/pam.d/login
    ```
    
    ```shell
    #%PAM-1.0
    
    auth       required     pam_securetty.so
    auth       requisite    pam_nologin.so
    auth       include      system-local-login
    auth       optional     pam_gnome_keyring.so
    account    include      system-local-login
    session    include      system-local-login
    session    optional     pam_gnome_keyring.so auto_start
    ```
    
    ```shell
    nvim /etc/pam.d/passwd
    ```
    
    ```shell
    ...
    password    optional    pam_gnome_keyring.so
    ```

42. nyalakan ulang
    
    ```shell
    reboot
    ```

43. aktifkan gnome-keyring
    
    ```shell
    systemctl enable --now --user gnome-keyring-daemon.socket
    reboot
    ```

44. konfigurasi polkit
    
    ```shell
    sudo usermod -aG wheel [nama user]
    ```
    
    ```shell
    systemctl --user enable --now hyprpolkitagent.service
    ```

45. konfigurasi flatpak
    
    ```shell
    flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
    ```
    
    ```shell
    flatpak install org.mozilla.firefox
    
    [pilih nomer 1 atau system]
    ```
    
    ```shell
    flatpak search marktext
    [ikutin application id]
    ```
    
    ```shell
    flatpak install com.github.marktext.marktext
    
    [pilih nomer 1 atau system]
    ```

46. konfigurasi apparmor
    
    ```shell
    sudo systemctl enable apparmor
    ```
    
    ```shell
    nvim /etc/cmdline.d/03-secs.conf
    ```
    
    ```shell
    lsm=landlock,lockdown,yama,integrity,apparmor,bpf
    ```
    
    ```shell
    mkinitcpio -P
    ```
    
    ```shell
    touch /var/log/syslog
    reboot
    ```
    
    ```shell
    systemctl status apparmor
    sudo aa-enabled
    [yes=aktif]
    sudo aa status
    ```

47. konfigurasi usbguard
    
    pastikan keyboard dan mouse yang digunakan sudah plugin
    
    ```shell
    usbguard generate-policy > /etc/usbguard/rules.conf
    systemctl enable --now usbguard
    systemctl enable --now firewalld
    systemctl enable --user --now power-profiles-daemon
    ```

48. konfigurasi sysctl 
    
    ```shell
    sudo cd /etc/sysctl.d
    nvim perf.conf
    ```
    
    ```shell
    ##VFS Cache
    vm.vfs_cache_pressure = 75
    ```
    
    ```shell
    cat /sys/block/[sda/sdb]/queue/scheduler
    echo bfq > /sys/block/[sda/sdb]/queue/schduler
    ```
    
    ```shell
    systemctl enable fstrim.timer
    systemctl enable irqbalance.service
    systemctl enable systemd-oomd.service
    ```
    
    ```shell
    lsmod | grep wdt
    
    [copy 'iTCO_wdt']
    ```
    
    ```shell
    nvim /etc/modules-load.d/watchdog.conf
    ```
    
    ```shell
    iTCO_wdt
    ```
    
    ```shell
    nvim /etc/systemd/system.conf.d/watchdog.conf
    ```
    
    ```shell
    [Manager]
    RuntimeWatchdogSec=10s
    RebootWatchdogSec=45s
    ```
    
    ```shell
    reboot
    ```

49. konfigurasi theme
    
    ```shell
    sudo git clone https://github.com/blackbird-hyprland/config-theme/usr/share/themes/blackbird
    reboot
    [tema nautilus hitam=benar]
    ```
