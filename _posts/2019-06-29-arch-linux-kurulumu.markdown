---
title: "Arch Linux Kurulumu (UEFI + LVM)"
date: 2019-06-29T15:45:33+03:00
---

Selamlar. Bu yazıda Arch linux kurulumunu uefi ve lvm ile birlikte yapacağız.
Öncelikle klavyemizi Türkçe Q olarak ayarlayalım ve bilgisayarımızın UEFI desteği olup olmadığını kontrol edelim

{% highlight bash %}
loadkeys trq
efivar -l
{% endhighlight %}

Eğer UEFI desteği varsa resimdeki gibi bir çıktı görürsünüz.

![loadkeys_efivar](https://i.hizliresim.com/lQ9zdQ.png)


`ping` komutu ile internet bağlantımızı kontrol edelim.
{% highlight bash %}
ping -c 3 www.google.com
{% endhighlight %}

Eğer internet bağlantınız yoksa `wifi-menu` komutu ile wifi ağınıza bağlanabilirsiniz.

Şimdi cgdisk ile efi ve lvm bölümlerini ayarlayalım

> Not: Eğer gpt bölümlerdirme tablosu kullanıyorsanız sizde benim gibi diski bölümlendirmek için cgdisk kullanabilirsiniz. Eğer mbr bölümlendirme tablosu kullanıyorsanız bölümendirmeyi cfdisk ile yapınız.


{% highlight bash %}
cgdisk /dev/sda
{% endhighlight %}

New ile yeni bir bölüm oluşturuyorum. Bize başlangıcı soruyor boş bırakarak enter ile geçelim ve boyutu 500M girerek 500 MB bir alan oluşturalım bu bizim efi bölümümüz olacak. Bölüm tipine ef00 yazalım bölüm adını ise boş bırakabiliriz. EFI bölümü oluşturduk. Şimdi tekrar new ile yeni bir bölüm daha oluşturalım başlangıç ve boyutu boş bırakarak kalan tüm boş alanı kullanalım. Bölüm türü 8e00 girip LVM olarak ayarlıyoruz ve cgdisk ile işimiz bitiyor.

![cgdisk](https://i.hizliresim.com/gPyLq0.png)

Write ile değişiklikleri kaydedip Quit ile çıkıyoruz.

`lsblk`  komutu ile disklerin son halini görebilirsiniz.

![lsblk](https://i.hizliresim.com/4p8RP7.png)

Şimdi LVM ile fiziksel bir bölüm ve vg adında bir bölüm grubu oluşturalım.

{% highlight bash %}
pvcreate /dev/sda2
vgcreate vg /dev/sda2
{% endhighlight %}

Şimdi root ve home bölümlerini oluşturalım.
10 GB root bölümü için ayıracağım geri kalanın tamamını home bölümü için kullanacağım.

{% highlight bash %}
lvcreate -L 10G -n root vg
lvcreate -l 100%FREE -n home vg
{% endhighlight %}

> Not: Swap bölümü oluşturmadım ama siz isterseniz aynı şekilde home bölümü oluşturmadan önce swap bölümü oluşturabilirsiniz. Örneğin: `lvcreate -L 1G -n swap vg`

![lvm](https://i.hizliresim.com/lQ9zWQ.png)

Şimdi bölümleri formatlayalım bağlayalım.

{% highlight bash %}

mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/mapper/vg-root
mkfs.ext4 /dev/mapper/vg-home

mount /dev/mapper/vg-root /mnt

mkdir /mnt/boot
mkdir /mnt/boot/efi
mkdir /mnt/home

mount /dev/sda1 /mnt/boot/efi
mount /dev/mapper/vg-home /mnt/home

{% endhighlight %}

> Not: Eğer swap bölümü oluşturduysanız swap bölümünüde biçimlendirin ve aktif hale getirin.Bu işlemi şu şekilde yapabilirsiniz:
>`mkswap /dev/mapper/vg-swap`<br>
>`swapon /dev/mapper/vg-swap`

Şimdi temel sistem paketlerini kurabiliriz

{% highlight bash %}
pacstrap -i /mnt base base-devel
{% endhighlight %}

Sorulan bütün soruları enter ile geçebiliriz.<br>
fstab dosyasını oluşturalım

{% highlight bash %}
genfstab -U /mnt >> /mnt/etc/fstab
{% endhighlight %}

Şimdi chroot ortamına geçebiliriz.
{% highlight bash %}
arch-chroot /mnt /bin/bash
{% endhighlight %}

Hostname yani bilgisyar adını ayarlayalım. Ben arch olarak ayarlıyorum.
{% highlight bash %}
echo arch > /etc/hostname
{% endhighlight %}

Sistem yerelini ayarlamak için `/etc/locale.conf` dosyasını açalım
{% highlight bash %}
nano /etc/locale.conf
{% endhighlight %}

ve aşağıdaki satırları ekleyelim
> LANG=tr_TR.UTF-8

Bölge ve zaman dilimi için `/etc/timezone` dosyasını açalım
{% highlight bash %}
nano /etc/timezone
{% endhighlight %}
ve şu satırı ekleyelim
>Europe/Istanbul

ve şu komutu verelim
{% highlight bash %}
ln -s /usr/share/zoneinfo/Europe/Istanbul /etc/localtime
{% endhighlight %}

>Not: Eğer dosya var diye bir hata alırsanız önce /etc/localtime dosyasını siliniz `rm /etc/localtime`


Dil tercihimiz için `/etc/locale.gen` dosyasını açalım ve şu satırları bularak başlarındaki `#` işaretine kaldıralım
{% highlight bash %}
tr_TR.UTF-8 UTF-8
tr_TR ISO-8859-9
{% endhighlight %}

>Not: Nano editoründe ctrl+w ile arama yapabilirsiniz.

Değişikliklerin aktif olması için `locale-gen` komutunu verelim.

Temel sistem kurulumu sırasında pacstrap mkinitcpio komutunu çalıştırarak bir initial ramdisk oluşturdu ancak biz lvm kullandığımız için `/etc/mkinitcpio.conf` dosyası içirisinde bir değişiklik yaparak bu işlemini tekrarlamamız geriyor.

{% highlight bash %}
nano /etc/mkinitcpio.conf
{% endhighlight %}

komutu ile mkinitcpio.conf dosyasını açalım ve `HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)` yazan satırı bulalım ve `filesystems` ifadesinden önce `lvm2` ifadesini ekleyelim.

![hooks](https://i.hizliresim.com/zGYEJR.png)

F3 ile kaydedip F2 ile çıkalım. Şimdi mkinitcpio komutunu verelim.
{% highlight bash %}
mkinitcpio -p linux
{% endhighlight %}

Şimdi gerekli bazı paketleri kuralım

{% highlight bash %}
pacman -S grub efibootmgr networkmanager
{% endhighlight %}

Grub bootloader kuralım
{% highlight bash %}
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch --recheck
{% endhighlight %}

> Not: grub-install işlemi sonunda No error reported yazısını göremiyorsanız bu bir yerlerde hata yaptınız ve grub doğru şekilde kurulamadı demektir..

Grub config dosyasını oluşturalım
{% highlight bash %}
grub-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}

![grub](https://i.hizliresim.com/WXErG4.png)

Network manageri başlangıçta aktif hale getirelim.
{% highlight bash %}
systemctl enable NetworkManager
{% endhighlight %}

root parolasını belirleyelim
{% highlight bash %}
passwd
{% endhighlight %}

Şimdi chroot ortamından çıkış yapalım bölümlerimizi ayıralım ve sistemi yeniden başlatalım.
{% highlight bash %}
exit
umount -R /mnt
reboot
{% endhighlight %}

> Not: Yeniden başlatma sırasında kurulum ortamını bilgisayarınızdan ayırmayı utmayınız

Yeniden açıldığında root olarak giriş yapabilirsiniz.
Şuan aslında arch kurulumu bitti ama daha yapmamız gereken şeyler var.

Önce paket depolarını güncelleyelim ve güncellemeleri kontrol edelim.
{% highlight bash %}
pacman -Syyu
{% endhighlight %}

Bazı sürücü ve paketleri yükleyelim.
{% highlight bash %}
pacman -S xorg xorg-server xorg-xinit mesa alsa-lib alsa-utils dbus
{% endhighlight %}

Şimdide bir kullanıcı oluşturalım.
{% highlight bash %}
useradd -m -g users -G optical,storage,wheel,video,audio,users,power,network,log -s /bin/bash kagan
{% endhighlight %}

Oluşturduğumuz kullanıcının parolasını belirleyelim.
{% highlight bash %}
passwd kagan
{% endhighlight %}

Yeni kullanıcının sudo ile komutları çalıştırabilmesi için `/etc/sudoers` dosyasını açıp `%wheel ALL=(ALL) ALL` satırının başındaki `#` işaretini kaldıralım.

![sudoers](https://i.hizliresim.com/jqjYQW.png)

Şimdi sistemi `reboot` komutu ile yeniden başlatarak yeni oluşturduğumuz kullanıcı ile giriş yapalım.

Son olarak bir masaüstü ortamı kuralım. Ben xfce kuruyorum eğer başka masaüstü ortamı kurmak istiyorsanız [arch wiki]( https://wiki.archlinux.org/) hepsini anlatmış.
{% highlight bash %}
sudo pacman -S xfce4 xfce4-goodies lightdm
{% endhighlight %}

lightdm açılışta aktif hale getiriyoruz ve kurulum bitiriyoruz.
{% highlight bash %}
sudo systemctl enable lightdm
{% endhighlight %}

![arch](https://i.hizliresim.com/VQ6y3B.png)

İşte bu kadar basit :)
