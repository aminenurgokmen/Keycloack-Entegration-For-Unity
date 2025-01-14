Bu entegrasyon için docker kurulumlarını halletikten sonra bir mimari oluşturmamız gerekli.

1. NGINX’in 8080 portunu (host üzerinde) dinlediğini,
2. Keycloak’ın ise 8086 portunda (host üzerinde) çalıştığını,
3. Docker üzerinde bu iki servisi ayağa kaldırdığını varsayıyoruz.

# Webde http://localhost:8086/ adresine gidip şu adımları takip et.
1.	admin / admin bilgilerini gir. (Compose dosyası içerisinde belirtilir.)
              
  keycloak:
    image: quay.io/keycloak/keycloak:23.0.6
    ports:
      - "8086:8080"
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_HOSTNAME_STRICT: "false"
      KC_HOSTNAME: "localhost"
    command:
      - "start-dev"
    restart: unless-stopped

2.	Giriş yaptıktan sonra sol üstte “Master” adlı bir Realm görürsün. İstersen “Master”’ı kullanabilirsin, ama pratikte yeni bir realm oluşturmak daha temizdir. Örnek:
o	Add Realm butonuna tıkla,
o	Name: “myrealm” (sen dilediğin adı verebilirsin).
3.	Yeni realm’ine geç. Sol menüde “Clients” bölümüne gir, “Create client” butonuna bas.
o	Client ID: unity-webgl-client (dilediğin bir isim)
o	“Client type” olarak “OpenID Connect”
o	Alt kısımda “Root URL” kısmına http://localhost:8080/ gibi değeri gir. (NGINX 8080 port üzerinden yayın yapacak ya, o yüzden 8080)
o	Kaydet/Devam et.
4.	Oluşan client’ın ayar ekranında şu kısımları kontrol et:
o	Access Type: public (ya da “confidential” istersen, ama o zaman client secret vb. konfigürasyonu gerekebilir. Yeni başlayanlar için public yeterli.)
o	Valid redirect URIs: http://localhost:8080/*
o	Web Origins: http://localhost:8080
o	Bu sayede tarayıcı Keycloak’tan dönerken http://localhost:8080/... adresine redirect edebilecek.
Not: 8080, NGINX’ten Unity yayını yaptığın port. Keycloak ise 8086’da çalışıyor. Bu sayede cross-origin (CORS) ayarlarını “Web Origins” ile halletmiş oluyoruz.
3. Test Kullanıcısı Oluşturma
•	Sol menüde “Users” → “Add User” ile bir kullanıcı ekle (örnek: amine / şifre: 123).
•	Sonrasında “Credentials” sekmesinden şifreyi girip “Set password” diyerek atayabilirsin.
Bu kadar. Keycloak tarafı hazır.


# Unityde açılmasını istediğin sayfayı WebGL olarak build al. 

Build klasorünün içerisindeki dosyaları docker klasoründeki volumes-> ngnix -> web klasorüne taşı.

Burada keykloackı aktif etmemiz için index dosyasında değişiklikler yapmamız gerekli. Bu dosyayı notepad ile aç ve içerisini yukarıda verilen index.html dosyasının içeriği ile değiştir. 

Tekrardan docker-compose up -d yapıp localhost:8080 adresine gidiyoruz ve keycloackta belirlediğimiz user ve password ile uygulamamıza erişebiliyoruz.

Bu kadar! 




