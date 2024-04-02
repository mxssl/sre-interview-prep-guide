- [웹브라우저에 `www.tech.com`을 입력하면 어떻게 될까?](#--------wwwtechcom---------------)
  * [DNS](#dns)
  * [Network](#network)
  * [HTTP](#http)

# 웹브라우저에 `www.tech.com`을 입력하면 어떻게 될까?

- Client <-> Web Server 구성요소(linux 클라이언트 사용 + Comcast home cable connection을 통해 연결 시도)
  - DNS
  - Network
  - HTTP
- linux client란?
- Comcast란?

## DNS
- `www.tech.com` DNS lookup 수행
- linux client -> name service cache daemon
  - `www.tech.com` DNS lookup 요청이 이전부터 캐싱되었는지 확인합니다.
  - 만약 아니라면?
    - name server로 DNS 쿼리 요청
      - DNS 요청이 클라이언트의 /etc/resolv.conf에 지정된 name server로 전송될 때 발생
      - DNS resolver는 로컬 라우터를 의미
        - 라우터는 모든 DNS 요청을 configuration에 지정된 resolver(Comcast에서 수신했을 수 있음)로 전달
    - Comcast DNS 서버는 DNS 쿼리 요청을 `.com` root name server로 전달합니다.
    - 그 후, `tech.com`의 name server가 검색됩니다.
      - `tech.com` name server는 registrar(등록기관)에 도메인이 등록될 때 지정됨
    - 그 다음 요청으로, zone files에서 `http://www.tech.com`을 조회하고, 요청에 응답하는 `tech.com` name server로 전송됩니다.
      - forward lookup zone file : host name을 IP 주소와 일치시킴
        - recursive 요청
          - client -> DNS 서버가 찾아서 반환해야 하는 응답을 요청
        - non recursive 요청
          - DNS 서버가 최종 응답을 반환하지 않으면, 클라이언트는 다음 resolver와 communication하여 이를 알아내야 합니다
          - 각 DNS 서버에는 자체 캐시가 있을 수 있으므로 캐싱 값이 있으면 언제든지 캐시에서 응답이 올 수 있습니다.
      - reverse lookup zone file : (내용 추가필요)

## Network
(DNS 쿼리를 통해 최종 IP를 가져온 이후)
- 클라이언트와 서버 간에 `3 way TCP handshake`를 통해 네트워크 통신이 이루어집니다.
  - sync, sync-ack, ack
- Client가 DNS 쿼리를 통해 최종 IP 주소를 얻게 된 이후 TCP connection이 이뤄집니다.
  - 클라이언트는 라우팅 테이블을 사용하여 `tech.com`의 네트워크에 대한 entry가 있는지 확인하고, entry가 없다면 클라이언트는 연결 요청 또는 동기화 패킷을 default 게이트웨이로 보냅니다.
    - destination 네트워크에 대한 entry를 찾으려고 시도하고 이를 찾을 수 없으면 패킷이 default 게이트웨이로 전송됩니다.
    - 이는 패킷이 BGP를 실행하는 인터넷 게이트웨이에 도달할 때까지 발생합니다.
      - BGP 라우팅 테이블 
        - ISP가 할당한 모든 공용 IP 주소 목록이 포함됨
        - 여기에는 `tech.com`의 public IP 주소와 해당 주소로 가는 방법이 포함됨
  - 그러면 Comcast는 패킷의 destination 주소를 tech.com에 속하도록 설정하고, 이를 인터넷을 통해 보냅니다.
  - 

## HTTP
