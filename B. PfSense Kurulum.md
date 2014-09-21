 
PfSense'i vSphere 5.1 üzerine kurulumunu anlatacağım.

Kurulumdan önce vSphere sunucumuzun üzerinde 2 tane internet kartı olması gerekiyor. Biri gelen internet için diğer ise interneti içerde dağıtabilmemiz için. Gelen internet için WAN, içerde dağıtacağımız internet için ise LAN. LAN kablosunu ise switch'e bağlamalıyız.


![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/vmwareesxi52.png)

Son versiyonu indirmek için [tıklayınız](https://www.pfsense.org/download/mirror.php?section=downloads)

Ben burada `Computer Architecture` için `AMD64` seçtim ve `Platform` için ise  `Live CD with Installer` seçtikten sonra aşağıda çıkan ülkelerden kendinize en yakın olana tıklayıp indirmeyi başlatabilirsiniz. 

İndirme işlemimiz tamamlandıktan sonra sunucumuza vSphere Client üzerinden bağlanalım.
Buradan Configuration -> Networking e geldiğimizde varolan networku görmemiz lazım. Resimde görüldüğü gibi Fiziksel Ethernet kartı olarak vmnic0 görülmektedir. Management Network yazan `switch`imizin adını LAN olarak değiştireceğiz. Bu bizim internete bağlanmamızı sağlayacak.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_2_1a.png)

`Properties` e tıklayalım ve açılan pencereden ismi editleyip LAN yapalım.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_2_2a.png)

Bir tane daha Virtual Network oluşturmamız gerekiyor sağ Üstte `Add Network Wizard`a tıklarak bunu yapabilriliriz.
Açılan ekranda `Virtual Machine` seçip `Next` diyelim ve ardından bir network kartı seçelim

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_2_3a.png)

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_2_3a.png)

Tekrar Next yaptığımız zaman `Network Label`e WAN yazıp ağ ayarlarımızı tamamlayalım. Bittiğinde aşağdaki gibi görünmesi gerekiyor.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_2_6a.png)

>>> NOT: İsimlendirmeyi illa ki LAN ve WAN diye yapacağız diye bir şey yok ama başka birisi baktığı zaman neyin ne olduğunu anlaması için bu isimlendirmeler işin racondur.

## PfSense Sanal Makine Kurulum

Ağ ayarlamalarımız tamamlandığına göre şimdi pfSense kurulumuna başlayabililiriz. `Create New Virtual Machine` diyelim

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-1a.png)

pfSense için özel ayarlamalar yapacağımız için `Custom`u seçip `Next`e tıklayalım.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-1a.png)

Sanal Makinenin sunucu üzerinde görünecek ismini yazıp `Next`e tıklayalım

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-2a.png)

Kuracağımız yeri seçelim

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-3a.png)

`Virtual Machine Version 8` seçip yolumuza devam edelim.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-4a.png)

Burada çıkan ekrandan other diyip dropbox tan FreeBSD 64 bit i seçelim ve kaldığımız yerden devam edelim.

Kendimize göre CPUs ve REM ayarlarını yaptıktan sonra fazla uzatmadan ağ kısmına geçelim.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-8a.png)

Ayarlarımız resimde görüldüğü gibi olmalıdır.

>>> NIC 1 için LAN

>>> NIC 2 için WAN 

Next dedikten sonra 

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-9a.png)

Seçip tekrar Next diyelim.

`Create a new virtual disk`i seçip tekrar Next diyelim

Kullanacağımız alana göre disk boyutu verip Next e devam.

Finish yazısını görene kadar Next diyelim ve  son olarak 

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-13a.png)

işaretleyip Finsih diyelim ki bitirir bitirmez bize Edit kısmını açsın, işaretlemeden tıklarsanız sol taraftan pfSensi seçip sağ tıklayıp Propertiesten aynı pencereye gelebilirsiniz.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_3-14b.png)

Buradan `Hardware` kısmından Diskimizi seçip indirdiğimiz CD yi yerleştirmemiz lazım. Eğer pfSense kurulum imageini sanal makineye atmışsanız `Datastore ISO file` ı seçip `Browse` dan yolunu seçmelisiniz. Yok ben kendi bilgisayarıma indirdim diyorsanız `Client Device`ı seçip Finish deyiniz. Şimdi sol taraftan sunucumuzu seçip start diyelim.
`Client Device`ı seçmişseniz start anında CD yi yerleştirmelisiniz.

Start dedikten sonra Console kısmında bu ekrnaı görmelisiniz.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/Esxi_pfs_4-2a.png)

Eğer bunu görmüyorsanız CD yerleştirme işleminde bir sıkıntı olabilir.

Buradan 1i seçerek ilerleyelim

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/24.jpg)

Yükleme yapmak için I ya basınız diyor bizde öyle yapıp `I` ya basalım.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/34.jpg)

İstiyorsanız font/screenmap/klavye ayarlarını yapabilirsiniz ya da direk Accept diye devam edebiliriz.

İlk defa pfSense kuracağımız için `Quick/Easy Install` diyip geçelim.

Gelen uyarıları okumadan OK diyelim 

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/54.jpg)

Standar KERNEL imizide seçtikten sonra

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/74.jpg)

Reboot diyelim ve bitirelim. / Uyarıda da söylediği gibi CD yi çıkarmayı unutmayınız.

>>> Eğer kurulum sırasında hata almışsanız büyük ihtimalle işletim sisteminizin bitleri tutmamaktadır siz 64 indirip 32bit(x86) seçmiş olabilirsiniz veya tam tersi.

Reboot işleminden sorna Ethernet kartlarımızı görmüş olması lazım. Eğer birini görmemişse hata yapmışısızdır.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/93.jpg)

bizden VLAN ı yapılandırmak isteyip istemediğimizi soruyor `N` diyelim ve sıradaki adıma geçelim.

Şimdi bizden WAN için neyi kullanacaksınız diyor ki biz `le0` ı yazalım. Bunu daha önceden belirlemiştik pfSense kurulumuna geçmeden önce

bundan sorna bizde LAN için ne istediğimizi soracak ve diğerini yazmalıyız `le1` i.

Kurmak istediğiniz başka bir interface varsa bu şekilde seçmeye devam edebilirsiniz. 

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/123.jpg)

Bir şey yazmadan Enter a basalım ve sonraki adıma geçelim

Bizden değiştirdiğimiz ayarları uygulamak isteyip istemediğimizi soruyor.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/132.jpg)

`Y` ye basıp bu adımıda hızlıca geçelim.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/141.jpg)

Eğer üstteki resimi pfSense ekranımızda görmüşsek kurulum tamamlanmıştır. Şimdi tarayıcımıza koşalım ve serverımızın ip sini yazalım ve web arayüz kontrolüne geçip bir ohh diyelim.

![alt tag](http://sametyilmaz.com.tr/wp-content/uploads/212.jpg)

Sorularınız için önce google'ı sonra burayı kullanabilirsiliniz. Sağlıcakla kalın...
