API 1. Kullanıcı 
curl 'https://www.binanceapis.com/oauth-api/v1/user-info' \
--header 'Authorization: Bearer {access_token}'

Parametreler:

İsim	Tip	Zorunlu	Tanım
erişim_belirteci	Sicim	EVET	
Cevap:

{
  "code": "000000",
  "message": null,
  "data": {
    "userId": "e10e20b7f20947e7bd206b15ce3dae90"
  },
  "success": true
}

API 2. 
curl --request POST 'https://www.binanceapis.com/oauth-api/v1/revoke-token' \
--header 'Authorization: Bearer {access_token}'

Parametreler:

İsim	Tip	Zorunlu	Tanım
erişim_belirteci	Sicim	EVET	
Cevap:

{
  "code": "000000",
  "message": null,
  "data": true, // true means clear access_token success
  "success": true
}
Binance API'sine Erişmek İçin OAuth Kullanma
Binance API'leri kimlik doğrulama ve yetkilendirme için OAuth 2.0 protokolünü kullanır   . Binance, web sunucusu, tek sayfa (tarayıcı tabanlı), mobil ve yerel uygulamalar gibi yaygın OAuth 2.0 senaryolarını destekler. Bu belge, uygulamanızın bir kullanıcının adına bir API isteği gerçekleştirmesi için onayını güvence altına almak amacıyla Binance'in OAuth 2.0 sunucusuyla nasıl iletişim kurduğu konusunda size rehberlik edecektir.

Başlamak için, uygulamanızın gerekli izinleri tanımlaması veya scopes. Binance Geliştirici Merkezi'ni ziyaret edin , bir Binance varlık hesabı için kaydolun ve oradan konsolunuza giderek bir OAuth uygulaması oluşturun ve kendi istemci kimliğinizi ve istemci sırrınızı edinin. Şimdilik, Binance Girişi (Oauth2.0), yalnızca yakın ekosistem ortaklarına sunulmaktadır. Daha fazla bilgi için lütfen iş ekibimizle iletişime geçin.

Belirli uygulama türünüze bağlı olarak, burada listelenen iki farklı yetkilendirme akışından birini seçebilirsiniz:

Web uygulamaları için lütfen 'Yetkilendirme Kodu Akışı' bölümüne bakınız;
Tarayıcı tabanlı uygulamalar, mobil ve yerel uygulamalar için lütfen 'PCKE Akışı' bölümüne bakın.
1. Yetkilendirme Kodu 
Adım 1. Kullanıcıları Binance erişimi talebinde bulunmaya yönlendirin ve yetkilendirme parametrelerini ayarlayın 
GET https://accounts.binance.com/en/oauth/authorize?
    response_type=code&
    client_id=YOUR_CLIENT_ID&
    redirect_uri=YOUR_REDIRECT_URI&
    state=CSRF_TOKEN&
    scope=SCOPES
    Bir kullanıcıyı uygulamanıza erişim yetkisi vermek için Binance'e yönlendirirken ilk adımınız yetkilendirme isteğini oluşturmaktır.

Parametreler	Tanım
response_type	Gerekli Değercode
client_id	Gerekli Uygulamanızın istemci kimliği.
redirect_uri	gerekli Kullanıcıların yetkilendirmeden sonra yönlendirileceği web uygulamanızdaki URL. Bu değerin URL kodlu olması gerekir.
state	İsteğe bağlı CSRF (cross-site request forgery) saldırılarına karşı koruma sağlamak için CSRF belirteci.
scope	Gerekli Uygulamanızın erişim istediği kapsamların listesi, virgülle ( ,) ayrılmış şekilde.
İşte bir yetkilendirme URL'sinin örneği:

GET https://accounts.binance.com/en/oauth/authorize?
    response_type=code&
    client_id=a28f296f2cbe6c64b4d5dec24735d39b1b6fffcf&
    redirect_uri=https%3A%2F%2Fdomain.com%2Foauth%2Fcallback&
    state=377f36a4557ab5935b36&
    scope=user:openId,create:apikey

Adım 2. Binance kullanıcıdan 
Bu adımda, kullanıcı uygulamanıza talep edilen erişimi verip vermemeye karar verir. Bu aşamada, Binance uygulamanızın adını ve kullanıcının yetkilendirme kimlik bilgileriyle erişim izni istediği Binance API hizmetlerini gösteren bir onay penceresi görüntüler. Kullanıcı daha sonra uygulamanıza erişim izni vermeyi kabul edebilir veya reddedebilir.

Uygulamanızın bu aşamada herhangi bir şey yapmasına gerek yok çünkü Binance'in OAuth 2.0 sunucusunun geri yönlendirmesini bekliyor.

Adım 3. Binance, 
Kullanıcı başvurunuzu onaylarsa Binance'in OAuth sunucusu redirect_urigeçici bir yetkilendirme codeparametresiyle sizi tekrar size yönlendirecektir.

1. adımda bir parametre belirttiyseniz state, parametre de dahil edilecektir. Rastgele bir dize oluşturursanız veya bir çerezin veya istemcinin 'sini yakalayan başka bir değerin karma değerini kodlarsanız state, isteğin ve yanıtın aynı tarayıcıda kaynaklandığından emin olmak için yanıtı doğrulayabilir ve böylece siteler arası istek sahteciliği gibi saldırılara karşı koruma sağlayabilirsiniz

GET https://domain.com/oauth/callback?
    code=cf6941ae8918b6a008f1377f36a4557ab5935b36&
    state=377f36a4557ab5935b36

    Adım 4. Yenileme ve erişim 
Uygulamanız yetkilendirmeyi aldıktan sonra code, yetkilendirmeyi bir erişim belirteci ile değiştirebilir code; bu da bir POST çağrısı yapılarak yapılabilir:

POST https://accounts.binance.com/oauth/token?client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&grant_type=authorization_code&code=STEP3_CODE&redirect_uri=YOUR_REDIRECT_URI


Parametre	Tanım
grant_type	gerekli değerauthorization_code
code	gerekli Step3 dönüş kodu
client_id	gerekli Uygulamanızın istemci kimliği.
client_secret	gerekli Uygulamanızın istemci sırrı.
redirect_uri	gerekli Kullanıcıların yetkilendirmeden sonra yönlendirileceği web uygulamanızdaki URL. Bu değerin URL kodlu olması gerekir.

curl https://accounts.binance.com/oauth/token \
  -X POST
  -d 'client_id=je-client&client_secret=je-client-secret&grant_type=authorization_code&code=95OfIm&redirect_uri=https%3A%2F%2Fdomain.com%2Foauth%2Fcallback'

  {
  "access_token": "83f2bf51-a2c4-4c2e-b7c4-46cef6a8dba5",
  "refresh_token": "fb5587ee-d9cf-4cb5-a586-4aed72cc9bea",
  "scope": "read",
  "token_type": "bearer",
  "expires_in": 30714
}
POST https://accounts.binance.com/oauth/token?client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&grant_type=refresh_token&refresh_token=STEP4_REFRESH_TOKEN


Parametre	Tanım
grant_type	gerekli  değer refresh_token
refresh_token	Gerekli  Step4 yenileme belirteci
client_id	gerekli  Uygulamanızın istemci kimliği.
client_secret	gerekli  Uygulamanızın istemci sırrı.
Örnek POST çağrısı:	
curl https://accounts.binance.com/oauth/token \
  -X POST
  -d 'client_id=je-client&client_secret=je-client-secret&grant_type=refresh_token&refresh_token=95OfIm

Başarılı bir istekten sonra, yanıtta geçerli bir değer  döndürülür ve yanıttaki saniye cinsinden süreyi access_token aşarsa geçersiz sayılır .expires_in

POST https://accounts.binance.com/oauth/token?client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&grant_type=refresh_token&refresh_token=STEP4_REFRESH_TOKEN


Parametre	Tanım
grant_type	gerekli  değer refresh_token
refresh_token	Gerekli  Step4 yenileme belirteci
client_id	gerekli  Uygulamanızın istemci kimliği.
client_secret	gerekli  Uygulamanızın istemci sırrı.
Örnek POST çağrısı:	
curl https://accounts.binance.com/oauth/token \
  -X POST
  -d 'client_id=je-client&client_secret=je-client-secret&grant_type=refresh_token&refresh_token=95OfIm

Başarılı bir istekten sonra, yanıtta geçerli bir değer  döndürülür ve yanıttaki saniye cinsinden süreyi access_token aşarsa geçersiz sayılır .expires_in

Geçerli bir 'niz olduktan sonra access_tokenilk API çağrınızı yapabilirsiniz:

curl 'https://www.binanceapis.com/oauth-api/v1/user-info' \
--header 'Authorization: Bearer {access_token}'

Cevap:

{
  "code": "000000",
  "message": null,
  "data": {
    "userId": "e10e20b7f20947e7bd206b15ce3dae90",
    "email": "xx@xx.com"
  },
  "success": true
}

2. PKCE 
PKCE uzantısı, yetkilendirme kodunun kötü niyetli bir istemci tarafından ele geçirilip bir erişim belirteci ile değiştirildiği bir saldırıyı, yetkilendirme sunucusuna yetkilendirme kodunu değiştiren aynı istemci örneğinin akışı başlatanla aynı olduğunu doğrulama yolu sağlayarak önler. Daha fazla ayrıntı için https://tools.ietf.org/html/rfc7636 adresine bakın

Adım 1. Kullanıcıları Binance erişimi talebinde bulunmaya yönlendirin ve yetkilendirme parametrelerini ayarlayın 
GET https://accounts.binance.com/en/oauth/authorize?
    response_type=code&
    client_id=YOUR_CLIENT_ID&
    redirect_uri=YOUR_REDIRECT_URI&
    state=CSRF_TOKEN&
    scope=SCOPES&
    code_challenge=CODE_CHALLENGE&
    code_challenge_method=S256

    Bir kullanıcıyı Binance'e yönlendirerek uygulamanıza erişimi yetkilendirmek için ilk adımınız yetkilendirme isteğini oluşturmaktır. Yeni bir PKCE code_verifier oluşturmanız ve depolamanız gerekir, ayrıca STEP4'te kullanılacaktır

    İşte javascript generate code_verifier'

// Generate a secure random string using the browser crypto functions
function generateRandomString() {
  var array = new Uint32Array(28);
  window.crypto.getRandomValues(array);
  return Array.from(array, (dec) => ("0" + dec.toString(16)).substr(-2)).join(
    ""
  );
}

// Calculate the SHA256 hash of the input text.
function sha256(code_verifier) {
  const encoder = new TextEncoder();
  const data = encoder.encode(code_verifier);
  return window.crypto.subtle.digest("SHA-256", data);
}

// Base64-urlencodes the input string
function base64urlencode(hashed) {
  return btoa(String.fromCharCode.apply(null, new Uint8Array(hashed)))
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=+$/, "");
}

// Return the base64-urlencoded sha256 hash for the PKCE challenge
async function generateCodeChallenge(code_verifier) {
  hashed = await sha256(code_verifier);
  return base64urlencode(hashed);
}

GET https://accounts.binance.com/en/oauth/authorize?
    response_type=code&
    client_id=a28f296f2cbe6c64b4d5dec24735d39b1b6fffcf&
    redirect_uri=https%3A%2F%2Fdomain.com%2Foauth%2Fcallback&
    state=377f36a4557ab5935b36&
    scope=user:openId,create:apikey&
    code_challenge=ARU184muFVaDi3LObH5YTZSxqA5ZdYPLspCl7wFwV0U
    code_challenge_method=S256