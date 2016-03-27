---
layout: post
title: "Android Permissions"
date: 2016-03-26 23:20:20+00:00
image: '/assets/img/permission_marsh.png'
description: 'Marshmallow'da Yenilenen Izin Sistemi

tags:
- permission
- java
- android
- marshmellow
- dangerous
categories:
- Java
- Programming
- Android

---

Android Marshmallow oncesinde uygulama izin sistemi tek bir akistan calisiyordu ve uygulama yuklenirken alinan izinler uygulama hayati boyunca devam ediyordu.

{% include image.html url="https://cdn-images-1.medium.com/max/800/1*G7v3-Taoa23yLmJWRF3Vsw.png" description="Permissions on Dropbox" %}

Bu durum uygulamayi yayinlayan acisindan iyi olsa da kullanicilar acisindan sorun teskil edebiliyordu. Android Marshmallow ile birlikte kullanici, uygulama yukluyken uygulamanin kullandigi  _tehlikeli_ kategorisinde olan izinleri iptal edebilecek.

{% include image.html url="http://sercangedik.com/images/Screenshot_20160326-223245.png" description="Runtime Permissions on WhatsApp" %}


Peki bu izin iptal edildi ancak sizin uygulamanizin o izne ihtiyaci var ve o izin olmadan islem yapamiyorsunuz. Bu durumda ne olacak ? 

Android’in sundugu yeni cozum Runtime Permissions. Uygulama calisirken kullaniciya acilan bildirim ekraniyla bu izin yeniden istenir, kullanici o izni verirse ise devam edebilirsiniz.

{% include image.html url="https://cdn-images-1.medium.com/max/800/1*ILNgIlYHcw5JBaF5XRZ3eA.png" description="Permissions on Hangouts" %}


**Izin Kategorileri**
-----------------------

Android sisteminde kullanilan izinler 2 kategoriden olusuyor,

**Dangerous (Tehlikeli)**<br />
**Normal**

Burada tehlikeli permissionlari gorebiliriz.

{% include image.html url="https://www.captechconsulting.com/blogs/library/FE2851C39CA340F5A7B7E8AC74B4BFE9.ashx" description="Dangerous Permissions" %}

Runtime Permissions
-------------------

Uygulama calisirken bu izni nasil alacagiz ? Asagidaki kod ornegi uzerinden gidelim,

{% highlight java %}
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //yazdigimiz metodu cagiralim
        //kullanicidan rehberine erisme izni isteyecegiz.
        getPermissionToReadUserContacts();
    }

    // bu integer bizim payload degerimiz olacak. Kullanicidan cevap geldiginde 
    // bu degerle kontrol yapacagiz
    private static final int READ_CONTACTS_PERMISSIONS_REQUEST = 1;

    
    public void getPermissionToReadUserContacts() {
        // 1) Genel olarak ContextCompat.checkSelfPermission(...) metodunu
        // kullanin. Bu metod support kutuphanesiyle birlikte (23.0.2) 
        // geliyor. Alternatifi olan Context.checkSelfPermission(...) metodu 
        // sadece Marshmallow icerisinde var.
        // 2) Izinleri her zaman kontrol edin. Izin verilmis olsa bile. 
        // Cunku kullanici herhangi bir zamanda ayarlardan uygulamanin 
        // iznini kaldirmis olabilir.
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS)
                != PackageManager.PERMISSION_GRANTED) {

            // Bu iznin verilmedigi sartin kod blogu icerisindeyiz.
            // shouldShowRequestPermissionRationale metodu kullaniciya detayli 
            // aciklama yapmamizi saglayan 2. sans diyebilecegimiz bir alan. 
            // Eger kullanici izin vermez ise neden istedigimizi burada 
            // aciklayabiliriz. Bu metodun true donmesi ise kullanicinin 
            // izin isteme uyarisina hayir cevabi vermesi sartinda olusur.
            // Bu durumun bir ornegini asagidaki gorselden gorebilirsiniz.
            if (shouldShowRequestPermissionRationale(
                    Manifest.permission.READ_CONTACTS)) {
                //Bu kisimda kendi arayuzumuz ile kullaniciya derdimizi anlatabiliriz.
            }

            //Bu metod android'in kendi arayuzuyle kullanicidan izin istemesini saglar.
            //Anrdoid arayuzuyle iznin nasil istendigini asagida gorebilirsiniz.
            requestPermissions(new String[]{Manifest.permission.READ_CONTACTS},
                    READ_CONTACTS_PERMISSIONS_REQUEST);
        }
    }

    // requestPermissions metodunun callback kismi
    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           @NonNull String permissions[],
                                           @NonNull int[] grantResults) {
        // Sinifin basinda tanimladigimiz payload degerini kontrol ediyoruz.
        // Kullanici gercekten de bizim istedigimiz izne mi cevap verdi diye.
        if (requestCode == READ_CONTACTS_PERMISSIONS_REQUEST) {
            if (grantResults.length == 1 &&
                    grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Read Contacts permission granted", Toast.LENGTH_SHORT).show();
                //PERMISSION_GRANTED ise kullanicidan izni kaptik diyebiliriz.
            } else {
                Toast.makeText(this, "Read Contacts permission denied", Toast.LENGTH_SHORT).show();
                // Degilse kullanici izin vermedi anlamina gelir. Burada 
                // sizin uygulamaniza bagli olarak alternatif yollar 
                // uretmeniz gerekebilir. Eger rehber ile uygulama 
                // acisindan hayati bir is yapiyorsaniz izni bi 
                // sekilde kapmak zorundasiniz. Aksi takdirde calisma 
                esnasinda exception almaniz olasi.
            }
        } else {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }
    }
}
{% endhighlight %}

{% include image.html url="http://sercangedik.com/images/Screenshot_20160326-223305.png" description="shouldShowRequestPermissionRationale'nun true donmesi" %}

{% include image.html url="http://sercangedik.com/images/Screenshot_20160326-223317.png" description="Android'in kendi arayuzuyle izin istemesi" %}

Eger kullanici "Bir daha sorma" kutusunu isaretleyip reddederse, bir sonraki izin isteme durumunda kullaniciya, ayarlardan bu izni acmasini soylemeniz gerekebilir.

Peki manifest.xml dosyasinda ne gibi degisiklikler yapmamiz gerekecek ?

{% highlight xml %}
<uses-sdk
        android:minSdkVersion="19"
        android:targetSdkVersion="23"/>

<uses-permission android:name="android.permission.READ_CONTACT"/>

{% endhighlight %}

Sadece targetSdkVersion'u 23 yapmaniz yeterli. Bu sayede siz Android'in son surumu olan (Android N'yi ayri tutuyorum) Marsmallow'u desteklediginizi belirtiyorsunuz. Izinler kisminda olan READ_CONTACT izni ise aynen duruyor, silmiyoruz.

Genel hatlariyla Android Marsmallow ile birlikte gelen Runtime Permission yeniligini inceledik. Bir sonraki yazimda gorusmek dilegiyle.

Sayanora.