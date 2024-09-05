### Binance için Kullanıcı Veri Akışları (2024-04-02)

## Genel WSS bilgileri

• Temel API uç noktası şudur:
https://api.binance.com
• Kullanıcı Veri Akışı
/listenKeyoluşturulduktan sonra 60 dakika boyunca geçerlidir.
• /PUT Aktif bir şekilde yapıldığında
/listenKey geçerlilik süresi 60 dakika uzatılır.
• /POST Aktif bir hesaba işlem yapıldığında, /listenKey aktif olan hesap geri döndürülür
/listenKey ve geçerliliği 60 dakika uzatılır.
• Temel websocket uç noktası şudur:
# wss://stream.binance.com:9443
# • Kullanıcı Veri Akışlarına
# /ws/<listenKey> 
veya'
# /stream?streams=<listenKey> 
adresinden erişilir.
# • Stream.binance.com'a
tek bir bağlantı yalnızca 24 saat geçerlidir; 24 saatlik sürenin sonunda bağlantınızın kesilmesini bekleyin
# API Uç 
# Noktaları

# Bir listenKey
# (USER_STREAM)

POST /api/v3/userDataStream
