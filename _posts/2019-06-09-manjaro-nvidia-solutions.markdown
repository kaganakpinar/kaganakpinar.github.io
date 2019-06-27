---
title: "Manjaro Linux Üzerinde Nvidia Optimus Çözümleri"
date: 2019-06-09T12:22:33+03:00
---

Merhabalar,

Eğer çift ekran kartına sahip bir bilgisayar ve manjaro linux kullanıyorsanız nvidia’nın kapalı kaynak sürücülerini kurmak için 3 farklı yöntem var.
Bumblebee, dahili ekran kartını devre dışı bırakma ve Optimus Manager

### 1) Bumblebee Nasıl Manjaroya Kurulur ?

Nvidia ekran kartınızı kullanmak için bumblebee çözümünü kullanabilirsiniz ancak performans çok düşük olacaktır.
Öncelikle mhwd komutu ile kurabileceğimiz sürücüleri görelim.

![mhwd](https://i.hizliresim.com/3ORr3O.png)

bumblebee ile nvidia sürücüsünü kuralım:
{% highlight bash %}
mhwd -i pci video-hybrid-intel-nvidia-bumblebee
{% endhighlight  %}

kullanıcıyı bumblebee grubuna ekleyelim:
{%  highlight bash %}
sudo gpasswd -a <USER> bumblebee
{% endhighlight %}

ve son olarak bilgisayarı yeniden başlatıyoruz.
Bumblebee kuruldu. bir uygulamayı nvidia kartı ile çalıştırmak için optirun komutunu kullanmanız yeterli.

![optirun](https://i.hizliresim.com/BORXaj.png)

görüldüğü gibi bumblebee çalıyor.
Steam üzerinde oyunları bumblebee ile çalıştırmak için oyun ayarlarından başlatma seçeneklerini şu şekilde düzenleyin:

![optirun-steam](https://i.hizliresim.com/GZpnZy.png)

### 2) Dahili Ekran Kartını Devre Dışı Bırakma

Bumblebee yöntemini denediyseniz yeteli performans alamadığınızı görmüşsünüzdür. Bu yöntem ile bilgisayarımızın dahili ekran kartını devre dışı bırakıyoruz ve sadece nvidia kartnın çalışmasını sağlıyoruz.
Böylece istediğimiz performansı elde ediyoruz ancak bilgisayarın şarjı normalden hızlı bitebilir.
Mhwd ile sürücüleri görmüştük şimdi video-nvidia sürücüyü kuralım

{% highlight bash %}
sudo mhwd -i pci video-nvidia
{% endhighlight %}

![video-nvidia](https://i.hizliresim.com/nb5Po0.png)

şimdi açılışta siyah ekranda kalmamak için mhwd tarafından oluşturulan config dosyalarını silelim.
{% highlight bash %}
sudo rm /etc/X11/mhwd.d/nvidia.conf
{% endhighlight %}

sddm kullanıyorsanız siyah ekranda kalmamak için /usr/share/sddm/scripts/Xsetup dosyasını şu şekilde düzenlemeliyiz:
{% highlight bash %}
sudo nano /usr/share/sddm/scripts/Xsetup
{% endhighlight %}

![Xsetup](https://i.hizliresim.com/gP4lR3.png)

Lightdm kullanıyorsanız siyah ekranda kalmamak için /etc/lightdm/ dizinine gidin ve display-setup.sh dosyası olusturarak şu şekilde düzenleyin:

![display-setup](https://i.hizliresim.com/XbDWj7.png)
ve bu dosyayı çalıştırabilir yapın:
{% highlight bash %}
sudo chmod +x /etc/lightdm/display-setup.sh
{% endhighlight %}

daha sonra /etc/lightdm/lightdm.conf dosyasın açın ve [Seat:*] altındaki display-setup-script satırını bulun ve şu şekilde düzenleyin:
![lightdm](https://i.hizliresim.com/zG2R1Y.png)

ve şimdi yeniden başlatalım

![video-nvidia-2](https://i.hizliresim.com/kM1raW.png)
Görüldüğü gibi şuan sadece nvidia sürücüsü çalışıyor.

### 3) Optimus Manager Nasıl Kurulur ?

Optimus manager bizim istediğimi zaman nvidia istediğimiz zaman intel kartını kullanmamızı sağlayacak bir araç. Önceki çözülere göre daha mantıklı bir çözüm.

Öncelikle mhwd ile nvidia sürücüsünü kuralım:
{% highlight bash %}
sudo mhwd -i pci video-nvidia
{% endhighlight %}

mhwd ile oluşan config dosyası varsa silin:
{% highlight bash %}
sudo rm /etx/X11/mhwd.d/nvidia.conf
{% endhighlight %}
Aur deposundan optimus manageri kuralım:
{% highlight bash %}
pamac install optimus-manager
{% endhighlight %}

Optimus manager servisini açılışta etkin hale getirelim:
{% highlight bash %}
sudo systemctl enable optimus-manager.service
{% endhighlight %}

Eğer gdm ve gnome kullanıyorsanız gdm-prime paketini kurmalısınız. Aur da mevcut.

Eğer sddm kullanıyorsanız /etc/sddm.conf dosyasında DisplayCommand ve DisplayStopCommand satırlarını bulup başlarına birer # işareti koyun.
sddm.conf dosyası yoksa sorun yoktur.

Şimdi yeniden başlatalım.

optimus-manager –print-mode komutu ile şuan hangi sürücüyü kullandığımız görebiliriz ve optimus-manager –switch komutu ile geçiş yapabiliriz.

![optimus-manager](https://i.hizliresim.com/Z5ANaa.png)
Detaylı bilgi için optimus manager github sayfası: <https://github.com/Askannz/optimus-manager/>

optimus manager sistem çekmesi uygulaması için aur deposundan optimus-manager-qt paketini kurabilirsiniz.
gnome kullananlar için de bir eklenti mevcut: <https://github.com/inzar98/optimus-manager-argos>   
