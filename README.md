Bu entegrasyon için docker kurulumlarını halletikten sonra bir mimari oluşturmamız gerekli.

1. NGINX’in 8080 portunu (host üzerinde) dinlediğini,
2. Keycloak’ın ise 8086 portunda (host üzerinde) çalıştığını,
3. Docker üzerinde bu iki servisi ayağa kaldırdığını varsayıyoruz.

# Webde http://localhost:8086/ adresine gidip şu adımları takip et.
1.	admin / admin bilgilerini gir. (Compose dosyası içerisinde belirtilir.)
              
  ```yaml
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
  ```

2.	Giriş yaptıktan sonra sol üstte “Master” adlı bir Realm görürsün. İstersen “Master”’ı kullanabilirsin, ama pratikte yeni bir realm oluşturmak daha temizdir.
 Örnek:
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

Build1 bu kadar.

# Peki her seferinde bu dosyayı elle mi değiştireceğiz?

Build içeriği sürekli güncellenir. Her güncellemede biz bu entegrasyon için index.html dosyasının içeriğini değiştirmekle uğraşmamalıyız.

Asset klasorününün altına WebGLTemplates onun altına MyCustomTemplate adında iki klasor açın.

C:\Program Files\Unity\Hub\Editor\2023.2.19f1\Editor\Data\PlaybackEngines\WebGLSupport\BuildTools\WebGLTemplates dizinine gidin. 2023.2.19f1 benim kullandığım sürümüm. Hangi sürümü kullanıyorsanız o klasorü seçmeyi unutmayın.

Bu klasorün altında Default, Minimal, PWA, WebGLIncludes adında klasorler göreceksiniz. Biz PWA templateini kullanacağız. PWA'nın içersindeki her şeyi, Unity'de açmış olduğumuz MyCustomTemplate klasorünün altına yapıştırın.

index.html dosyasının içeriğini yukarıda bulunan Build2 klasoründeki index.html kodlarıyla değiştirin.

Unity'de File -> BuildSettings -> PlayerSettings -> Resulation and Presentation sekmesine gidin. Burada açmış olduğunuz MyCustomTemplate seçeneğini göreceksiniz. Onu seçin.

Clean Build alın. 

!!!ÖNEMLİ

index.html dosyasının içeriğinde dataUrl: buildURL + "/Build.data" olarak girildiği için build alacağınız klasorün adı Build olması elzemdir. Ya da bu yolu adını verdiğiniz klasorün adı olarak güncelleyin.

'''html
  var config = {
        arguments: [],
        dataUrl: buildUrl + "/Build.data",
        frameworkUrl: buildUrl + "/Build.framework.js",
        codeUrl: buildUrl + "/Build.wasm",
        streamingAssetsUrl: "StreamingAssets",
        companyName: "DefaultCompany",
        productName: "KeycloackEntegration",
        productVersion: "1.0",
        showBanner: unityShowBanner,
      };
'''


Build dosyalarının hepsini compose(yani containerlarınızın kurulu olduğu yer)\volumes\nginx\web dizininin altına yapıştırın.

http://localhost:8080 adresine gittiğinizde keycloackta açmış olduğunuz user ve password alanlarını doldurun ve unity sahnesine geçiş yapın.

Bu sistemle her seferinde build dosyasını değiştirmek zorunda kalmazsınız.

Bu kadar!








