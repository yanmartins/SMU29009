## O GamingAnywhere

O Gaming Anywhere é uma plataforma opensource que permite implementar um servidor em uma máquina rodando Linux ou Windows, podendo ter cliente nestes mesmos sistemas em diferentes computadores ou dispositivos móveis com suporte ao Android.

## Cenário

O cenário consiste em dois computadores. Onde o computador servidor está operando em Linux Ubuntu, e o cliente em um notebook com Windows 10. Ambos dispositivos estão em uma mesma rede local. Logo abaixo, encontram-se os seus respectivos IPs:

1. Servidor: ```191.36.13.49```
2. Cliente: ```191.36.13.41```


## [RTSP](https://tools.ietf.org/html/rfc7826)

O GamingAnywhere utiliza o protocolo *Real Time Streaming Protocol* (RTSP) para controle do transporte de mídia de forma síncrona. De acordo com a RFC 2326, o RTSP é um protocolo da camada de aplicação para controle sobre a entrega de dados com propriedades em tempo real. Este protocolo destina-se a controlar várias sessões de entrega de dados, fornecer um meio para escolher os canais de entrega, como UDP, *multicast* UDP e TCP, e fornecer um meio para escolher os mecanismos de entrega com base no RTP.

## Oferta de mídia

No console do cliente, pôde-se observar que o primeiro método foi o `DESCRIBE`; o par `DESCRIBE` constitui a fase de inicialização de mídia do RTSP. A qual pode ser vista logo abaixo:

- Cliente - Servidor

```
Sending request: 
DESCRIBE rtsp://191.36.13.49:8000/desktop RTSP/1.0
CSeq: 2
User-Agent: RTSP Client (LIVE555 Streaming Media v2014.05.27)
Accept: application/sdp
```

- Servidor - Cliente

```
Received a complete DESCRIBE response:
RTSP/1.0 200 OK   
CSeq: 2                   
Date: Tue, Jun 25 2019 13:41:17 GMT                                                                                                   
Content-Base: rtsp://191.36.13.49:8554/desktop/   
Content-Type: application/sdp              
Content-Length: 523

v=0                          
o=- 1561469908423507 1 IN IP4 191.36.13.49
s=Real-Time Desktop
i=desktop
t=0 0
a=tool:LIVE555 Streaming Media v2018.02.18
a=type:broadcast
a=control:*  
a=range:npt=0-
a=x-qt-text-nam:Real-Time Desktop
a=x-qt-text-inf:desktop
m=video 0 RTP/AVP 96
c=IN IP4 0.0.0.0
b=AS:3000
a=rtpmap:96 H264/90000
a=fmtp:96 packetization-mode=1;profile-level-id=4D402A;sprop-parameter-sets=Z01AKraAeAIn5YQAAAMABAAAAwGSPGDKgA==,aO88gA==
a=control:track1
m=audio 0 RTP/AVP 14
c=IN IP4 0.0.0.0
b=AS:128
a=control:track2
```

## Estabelecimento de sessão de mídia

Em seguida ocorre a requisição `SETUP`, onde esta especifica o mecanismo de transporte a ser usado para o fluxo de mídia da URI desejada. O cabeçalho `Transport` informa os parâmetros de transporte aceitáveis para o cliente para transmissão de dados, consequentemente a resposta conterá os parâmetros de transporte selecionados pelo servidor:

- Cliente - Servidor

```
Sending request: 
SETUP rtsp://191.36.13.49:8554/desktop/track1 RTSP/1.0
CSeq: 3
User-Agent: RTSP Client (LIVE555 Streaming Media v2014.05.27)
Transport: RTP/AVP;unicast;client_port=65348-65349 
```

- Servidor - Cliente

```
Received a complete SETUP response:
RTSP/1.0 200 OK
CSeq: 3
Date: Tue, Jun 25 2019 13:41:17 GMT
Transport: RTP/AVP;unicast;destination=191.36.13.41;source=191.36.13.49;client_port=65348-65349;server_port=6970-6971
Session: BBA6F613;timeout=65
```

- Cliente - Servidor

```
Sending request:
SETUP rtsp://191.36.13.49:8554/desktop/track2 RTSP/1.0CSeq: 4
User-Agent: RTSP Client (LIVE555 Streaming Media v2014.05.27)
Transport: RTP/AVP;unicast;client_port=65350-65351
Session: BBA6F613                                                                                                                       
```

- Servidor - Cliente

```                                                                                                                                     
Received a complete SETUP response:
RTSP/1.0 200 OK
CSeq: 4
Date: Tue, Jun 25 2019 13:41:18 GMT
Transport: RTP/AVP;unicast;destination=191.36.13.41;source=191.36.13.49;client_port=65350-65351;server_port=6972-6973
Session: BBA6F613;timeout=65 
```

A requisição `SETUP` ocorreu duas vezes nesse experimento, a primeira vez definiu o mecanismo de transporte para o decoder de vídeo (h264) e a segunda para o decoder de áudio (MPA).

|      ip      |     port    | codec |
|:------------:|:-----------:|:-----:|
| 191.36.13.41 | 65348-65349 |  h264 |
| 191.36.13.49 | 6970-6971   |  h264 |
| 191.36.13.41 | 65350-65351 |  MPA  |
| 191.36.13.49 | 6972-6973   |  MPA  |

Devido ao parâmetro ```unicast``` do cabeçalho ```Transport```, o canal da entrega selecionado na camada de transporte, foi o protocolo UDP.

## Iniciando fluxo de mídia

Posteriormente, a requisição ``PLAY`` informa ao servidor para começar a enviar dados através do mecanismo especificado na requisição ``SETUP``.

- Cliente - Servidor

```
Sending request: PLAY rtsp://191.36.13.49:8554/desktop/ RTSP/1.0
CSeq: 5
User-Agent: RTSP Client (LIVE555 Streaming Media v2014.05.27)
Session: BBA6F613
Range: npt=0.000-
```

- Servidor - Cliente

```
Received a complete PLAY response:
RTSP/1.0 200 OK
CSeq: 5
Date: Tue, Jun 25 2019 13:41:18 GMT
Range: npt=0.000-
Session: BBA6F613
RTP-Info: url=rtsp://191.36.13.49:8554/desktop/track1;seq=5822;rtptime=2341051175,url=rtsp://191.36.13.49:8554/desktop/track2;seq=45087;rtptime=2254534682
```

## Fluxo de mídia

Durante o streaming do jogo, foram capturados alguns dados referentes ao estado da transmissão:

```
# 1561470708.364522 H264-report: 8069KB rcvd; pkt-loss=63/6363,0.99%; bitrate=2152Kbps; jitter=394 (freq=90000Hz)
# 1561470708.364522 MPA-report: 482KB rcvd; pkt-loss=0/383,0.00%; bitrate=128Kbps; jitter=1940 (freq=90000Hz)
# 1561470715.288526 player frame rate: decoder 0 @ 50.0140 fps
# 1561470727.532938 player frame rate: decoder 0 @ 49.0019 fps

# 1561470738.369904 H264-report: 11308KB rcvd; pkt-loss=16/8603,0.19%; bitrate=3015Kbps; jitter=495 (freq=90000Hz)
# 1561470738.369904 MPA-report: 482KB rcvd; pkt-loss=0/383,0.00%; bitrate=128Kbps; jitter=1823 (freq=90000Hz)
# 1561470739.530119 player frame rate: decoder 0 @ 50.0117 fps
# 1561470751.527185 player frame rate: decoder 0 @ 50.0122 fps

# 1561470768.370254 H264-report: 14316KB rcvd; pkt-loss=0/10667,0.00%; bitrate=3817Kbps; jitter=171 (freq=90000Hz)
# 1561470768.370254 MPA-report: 482KB rcvd; pkt-loss=0/383,0.00%; bitrate=128Kbps; jitter=864 (freq=90000Hz)
# 1561470777.232620 player frame rate: decoder 0 @ 43.7789 fps
# 1561470789.245158 player frame rate: decoder 0 @ 49.9478 fps

# 1561470798.379325 H264-report: 11634KB rcvd; pkt-loss=55/8844,0.62%; bitrate=3102Kbps; jitter=501 (freq=90000Hz)
# 1561470798.379325 MPA-report: 481KB rcvd; pkt-loss=0/382,0.00%; bitrate=128Kbps; jitter=729 (freq=90000Hz)
# 1561470801.229597 player frame rate: decoder 0 @ 50.0649 fps
# 1561470813.235629 player frame rate: decoder 0 @ 49.9749 fps 
```

Durante todo o processos, o valor de *packet loss*, bem como o de *jitter* mantiveram-se estáveis. Raramente apresentando variações bruscas.

## Referencias

- [GamingAnywhere](http://gaminganywhere.org)
