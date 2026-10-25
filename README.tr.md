<div align="center">

# LiveHide

### Relay-only WebRTC ve uygulama katmanı E2EE ile gizlilik odaklı 1:1 sesli/görüntülü iletişim

<img src="./assets/hero.svg" alt="Animasyonlu proje başlığı" />

<p><a href="./README.md">English</a> · <a href="./README.de.md">Deutsch</a> · <a href="./README.tr.md"><strong>Türkçe</strong></a></p>

<p>
  <a href="https://github.com/tahakubilay"><img src="https://img.shields.io/badge/Taha%20Kubilay-GitHub-181717?style=for-the-badge&logo=github" alt="Taha Kubilay GitHub" /></a>
  <a href="https://github.com/tahakubilay/LiveHide"><img src="https://img.shields.io/badge/Repository-LiveHide-0A66C2?style=for-the-badge&logo=github" alt="Repository" /></a>
  <img src="https://img.shields.io/badge/WebRTC-Relay--Only-16A34A?style=for-the-badge" alt="Relay-only WebRTC" />
</p>

</div>

---

## Genel Bakış

LiveHide, 1:1 sesli ve görüntülü görüşmeler için gizlilik odaklı bir gerçek zamanlı iletişim platformudur.

Mimari, signaling ve medya taşıma katmanlarını birbirinden ayırır. Stateless Go gateway oda üyeliğini, şifreli signaling mesajlarını ve kısa ömürlü TURN kimlik bilgilerini koordine eder. WebRTC medyası yalnızca kimlik doğrulamalı TURN relay'ler üzerinden taşınır ve ayrıca uygulama katmanında uçtan uca şifrelenir.

## Teknik Mimari

LiveHide, TypeScript/Vite tarayıcı istemcisi ve minimum sunucu state'i kullanan bir Go gateway üzerine kuruludur.

Bağlantı kurulurken:

1. Peer'ler signaling gateway üzerinden bir odaya katılır.
2. Peer ilişkisi ve kriptografik state oluşturulur.
3. SDP ve ICE mesajları şifreli envelope'lar içinde değiş tokuş edilir.
4. İstemciler kısa ömürlü TURN kimlik bilgilerini alır.
5. WebRTC `iceTransportPolicy: "relay"` ile oluşturulur.
6. Host, server-reflexive ve peer-reflexive candidate'lar reddedilir.
7. Seçilen candidate pair `RTCPeerConnection.getStats()` üzerinden doğrulanır.
8. Uygulama katmanı medya şifreleme WebRTC Encoded Transforms ile kurulur.
9. Short Authentication String (SAS) peer doğrulaması sağlar.
10. Setup tamamlandığında signaling askıya alınır ve yalnız kontrollü ICE recovery gerektiğinde yeniden açılır.

### Temel Teknoloji Yığını

`Go` · `TypeScript` · `Vite` · `WebRTC` · `WebSocket` · `coturn` · `Caddy` · `Docker` · `WebCrypto`

## Bağlantı Akışı

```mermaid
sequenceDiagram
    participant A as Peer A
    participant G as Stateless Gateway
    participant T as TURN Relay
    participant B as Peer B

    A->>G: Odaya katıl + şifreli signaling
    B->>G: Odaya katıl + şifreli signaling
    G-->>A: Eşleşme / şifreli peer mesajları
    G-->>B: Eşleşme / şifreli peer mesajları

    A->>G: Kısa ömürlü TURN bilgisi iste
    B->>G: Kısa ömürlü TURN bilgisi iste
    G-->>A: Geçici TURN kimlik bilgileri
    G-->>B: Geçici TURN kimlik bilgileri

    A->>T: Relay candidate ayır
    B->>T: Relay candidate ayır

    A->>G: Şifreli offer / ICE
    G-->>B: Şifreli offer / ICE ilet
    B->>G: Şifreli answer / ICE
    G-->>A: Şifreli answer / ICE ilet

    A->>T: E2EE medya frame'leri
    T-->>B: Şifreli medyayı relay et
    B->>T: E2EE medya frame'leri
    T-->>A: Şifreli medyayı relay et

    Note over A,B: Signaling setup sonrasında askıya alınır
```

## Güvenlik Modeli

| Alan | Tasarım |
|---|---|
| **Ağ Yolu** | TURN-only WebRTC; doğrudan candidate'lar filtrelenir ve reddedilir |
| **Runtime Doğrulama** | Seçilen candidate pair WebRTC stats üzerinden kontrol edilir |
| **Signaling** | Sequence koruması ve AAD binding içeren AES-256-GCM envelope'ları |
| **Medya E2EE** | Dedicated Worker içinde WebRTC Encoded Transforms üzerinde AES-256-GCM |
| **Peer Doğrulama** | Short Authentication String (SAS) |
| **TURN Kimlik Doğrulama** | Kısa ömürlü geçici coturn kimlik bilgileri |
| **Sunucu State'i** | RAM'de geçici oda state'i |
| **Tarayıcı Kalıcılığı** | Call state için localStorage, sessionStorage, IndexedDB veya cookie kullanılmaz |
| **Hata Davranışı** | Güvenlik ön koşulu başarısızsa session gizliliği düşürmek yerine sonlandırılır |

LiveHide gizlilik koşullarını bağlantı state'inin bir parçası olarak ele alır. Güvenli connected state'e ancak gerekli transport, kriptografi ve doğrulama koşulları sağlandığında geçilir.

> [!IMPORTANT]
> TURN relay'ler ağ trafiğini taşımaya devam eder ve transport altyapısının parçasıdır. Gizlilik modeli, kimlik doğrulamalı relay kullanımı ile uçtan uca medya şifrelemesinin birlikte uygulanmasına dayanır.

## Doğrulama

Otomatik test ve güvenlik kontrolleri şu alanları kapsar:

- WebSocket protokol hardening
- Şifreli signaling
- Relay-only candidate enforcement
- coturn entegrasyonu
- İki tarayıcı arasında sesli/görüntülü görüşme
- Uygulama katmanı medya şifreleme
- Zero-persistence davranışı
- Fail-closed senaryolar
- Kontrollü ICE restart
- Go race detection ve security checks

## Mühendislik Odağı

LiveHide; gerçek zamanlı networking, uygulamalı kriptografi ve browser media API'lerini tek sistemde bir araya getirir.

`Real-Time Systems` · `WebRTC` · `Networking` · `Applied Cryptography` · `Privacy Engineering`

---

<div align="center">

### Geliştiren: [Taha Kubilay](https://github.com/tahakubilay)

</div>