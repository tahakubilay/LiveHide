<div align="center">

# LiveHide

### Datenschutzorientierte 1:1-Audio-/Videokommunikation mit Relay-only WebRTC und E2EE auf Anwendungsebene

<img src="./assets/hero.svg" alt="Animierter Projektkopf" />

<p><a href="./README.md">English</a> · <a href="./README.de.md"><strong>Deutsch</strong></a> · <a href="./README.tr.md">Türkçe</a></p>

<p>
  <a href="https://github.com/tahakubilay"><img src="https://img.shields.io/badge/Taha%20Kubilay-GitHub-181717?style=for-the-badge&logo=github" alt="Taha Kubilay auf GitHub" /></a>
  <a href="https://github.com/tahakubilay/LiveHide"><img src="https://img.shields.io/badge/Repository-LiveHide-0A66C2?style=for-the-badge&logo=github" alt="Repository" /></a>
  <img src="https://img.shields.io/badge/WebRTC-Relay--Only-16A34A?style=for-the-badge" alt="Relay-only WebRTC" />
</p>

</div>

---

## Überblick

LiveHide ist eine datenschutzorientierte Echtzeit-Kommunikationsplattform für 1:1-Audio- und Videoanrufe.

Die Architektur trennt Signalisierung und Medientransport. Ein zustandsloses Go-Gateway koordiniert Room-Mitgliedschaft, verschlüsselte Signalisierungsnachrichten und kurzlebige TURN-Zugangsdaten. Die WebRTC-Medien werden ausschließlich über authentifizierte TURN-Relays transportiert und zusätzlich durch Ende-zu-Ende-Verschlüsselung auf Anwendungsebene geschützt.

## Technische Architektur

LiveHide verwendet einen TypeScript/Vite-Browserclient und ein Go-Gateway mit bewusst minimalem serverseitigem Zustand.

Beim Verbindungsaufbau:

1. Peers treten über das Signalisierungs-Gateway einem Room bei.
2. Peer-Beziehung und kryptografischer Zustand werden aufgebaut.
3. SDP- und ICE-Nachrichten werden in verschlüsselten Envelopes ausgetauscht.
4. Die Clients beziehen kurzlebige TURN-Zugangsdaten.
5. WebRTC wird mit `iceTransportPolicy: "relay"` erstellt.
6. Host-, server-reflexive und peer-reflexive Kandidaten werden verworfen.
7. Das gewählte Kandidatenpaar wird über `RTCPeerConnection.getStats()` überprüft.
8. Media-E2EE wird über WebRTC Encoded Transforms installiert.
9. Eine Short Authentication String (SAS) ermöglicht Peer-Verifikation.
10. Die Signalisierung wird nach dem Setup suspendiert und nur bei kontrollierter ICE-Wiederherstellung erneut geöffnet.

### Kern-Stack

`Go` · `TypeScript` · `Vite` · `WebRTC` · `WebSocket` · `coturn` · `Caddy` · `Docker` · `WebCrypto`

## Verbindungsablauf

```mermaid
sequenceDiagram
    participant A as Peer A
    participant G as Stateless Gateway
    participant T as TURN Relay
    participant B as Peer B

    A->>G: Room beitreten + verschlüsselte Signalisierung
    B->>G: Room beitreten + verschlüsselte Signalisierung
    G-->>A: Pairing / verschlüsselte Peer-Nachrichten
    G-->>B: Pairing / verschlüsselte Peer-Nachrichten

    A->>G: Kurzlebige TURN-Zugangsdaten anfordern
    B->>G: Kurzlebige TURN-Zugangsdaten anfordern
    G-->>A: Temporäre TURN-Zugangsdaten
    G-->>B: Temporäre TURN-Zugangsdaten

    A->>T: Relay-Kandidat allokieren
    B->>T: Relay-Kandidat allokieren

    A->>G: Verschlüsseltes Offer / ICE
    G-->>B: Verschlüsseltes Offer / ICE weiterleiten
    B->>G: Verschlüsselte Answer / ICE
    G-->>A: Verschlüsselte Answer / ICE weiterleiten

    A->>T: E2EE-Mediaframes
    T-->>B: Verschlüsselte Medien weiterleiten
    B->>T: E2EE-Mediaframes
    T-->>A: Verschlüsselte Medien weiterleiten

    Note over A,B: Signalisierung wird nach dem Setup suspendiert
```

## Sicherheitsmodell

| Bereich | Design |
|---|---|
| **Netzwerkpfad** | TURN-only WebRTC; direkte Kandidaten werden gefiltert und verworfen |
| **Runtime-Verifikation** | Gewähltes Kandidatenpaar wird über WebRTC-Stats geprüft |
| **Signalisierung** | AES-256-GCM-Envelopes mit Sequenzschutz und AAD-Bindung |
| **Media E2EE** | AES-256-GCM über WebRTC Encoded Transforms in einem Dedicated Worker |
| **Peer-Verifikation** | Short Authentication String (SAS) |
| **TURN-Authentifizierung** | Kurzlebige temporäre coturn-Zugangsdaten |
| **Server-State** | Flüchtiger Room-State im Arbeitsspeicher |
| **Browser-Persistenz** | Kein localStorage, sessionStorage, IndexedDB oder Cookie-Storage für Call-State |
| **Fehlerbehandlung** | Fehler bei Sicherheitsbedingungen beenden die Session statt die Privatsphäre still zu reduzieren |

LiveHide behandelt Datenschutzanforderungen als Voraussetzungen des Verbindungszustands. Der sichere Connected-State wird erst erreicht, wenn Transport-, Kryptografie- und Verifikationsbedingungen erfüllt sind.

> [!IMPORTANT]
> TURN-Relays transportieren weiterhin Netzwerkverkehr und bleiben Teil der Transportinfrastruktur. Das Privacy-Modell basiert auf authentifiziertem Relay-Einsatz plus Ende-zu-Ende-Medienverschlüsselung.

## Verifikation

Automatisierte Prüfungen decken unter anderem ab:

- WebSocket-Protokollhärtung
- Verschlüsselte Signalisierung
- Relay-only Candidate Enforcement
- coturn-Integration
- Zwei-Browser-Audio-/Videoanrufe
- Media-E2EE auf Anwendungsebene
- Zero-Persistence-Verhalten
- Fail-closed-Szenarien
- Kontrollierte ICE-Restarts
- Go Race Detection und Security Checks

## Engineering-Fokus

LiveHide verbindet Echtzeit-Netzwerktechnik, angewandte Kryptografie und Browser-Media-APIs in einem System.

`Real-Time Systems` · `WebRTC` · `Networking` · `Applied Cryptography` · `Privacy Engineering`

---

<div align="center">

### Entwickelt von [Taha Kubilay](https://github.com/tahakubilay)

</div>