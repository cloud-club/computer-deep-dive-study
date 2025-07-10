# Overview of network-layer

- forwading
    - data plane이고, router마다 일어난다.
    - 보통 hw에 구현된다. 이유는 hw가 해야 빠르기 때문에
    - forwading table을 쓴다.
- routing
    - control plane, network-wide process
    - routing algo.
        - Dijkstra
        - Bellman Ford
        - then determine forwading table
    - SW에서 개발된다.
- control plane
    - two conttrol pahse approach
    - SDN

## Pre-router control plane

- decentralized
- 모든 라우터가 forwading과 routing이 가능하다.

## Logically centralized control plane

- centralized
- 쪼개서 관맇는 것은 굉장히 어렵다. 그래서 하나의 서버에서 다 관리하자.
- extra latency가 있다.

둘다 data plane은 동일하다, 다만 control plane이 중앙화 되었냐, 각각에 있냐 차이일 뿐.

## Network service model

- IP는 말한 부분들을 제공해주지 않는다.
    - IP is best effort...
- 하지만 netowrk layer가 이런걸 못 제공해주는 건 아니다.
  다만 너무 복잡하게 바뀔 뿐... ATM구조는 실제로 그래서 사라졌다.

# What's inside a router

## Router architecture overview

- 실제로 내부에서 송신과 수신이 쪼개져있다.
- routing processor가 라우팅 테이블을 만들고 그걸 각자의 랜카드에 쭉 뿌린다.(soft copy)
    - 매번 routing processor를 호출하지 않는다. 복사본을 보고 local에서 바로 판단해서 넘긴다 .
- 그걸 결정하는 것은 input의 마지막 박스에서 수행한다.
  우리가 말하는 congestion은 output에서 발생한다고 생각해라.

## Input port functions

계속 보다가 마지막에 forwading table을 본다.
fowarding은 도착지를 기준으로 판단한다.
임의로 패킷을 큐에 저장한다.

## Destination-based forwading

지금 보면, 범위이다.

## Longest prefix matching

가장 길게 매칭된걸 가져간다. 그거일 확률이 더 높으니까... 그게 좀더 많이 잘 나눠진 형태일 테니까.
라우터가 요청을 nano sec단위로 하기 위해서는 DRAM으론 안된다. TCAMs라는 걸 쓸수도 있다.

TCAMs에서는 1 clock 안에 조회 가능하다.

## Other Functions in input port

- 물리계층과 링크계층의 역할을 여기서 수행함
-
- packet version number, checksum, and TTL
    - **TTL and checksum must be written (for IPv4)** -> TF에 나오기 너무 좋다.
    - 이게 input port에서 이뤄짐...!

## Switching fabrics

스위칭 패브릭은 라우터나 스위치에서 입력 포트에서 출력 포트로 패킷을 전달하는 내부 경로 구조입니다.

switching rate

- I/O 선로 속도의 배수로 측정함
- N개의 입력 포트가 있다면
    - **스위칭 속도 = N × 선로 속도(line rate)** 가 바람직함 (병목 방지)
- 이상적으로 cross bar는 N Packet들을 바로 전달할 수 있다.
- 그림을 많이 보기

- memory
    - 옛날 방식 별로
- bus
    - 모든 친구한테 쏘지만, 본인(포트)가 아니라면 버림
- crossbar
    - switch controller가 연결점들을 연결하거나 닫는 것으로 switching하게 한다. 이 경우 최대 N개를 동시에 보낼 수 있음 각각의 line들이 유지되므로..?

## Input port queueing

switch fabric이 input 이 들어오는 것보다 느린 경우, queuing 이 발생할 수 있다.

- **HOL blocking** : 입력 큐에서 대기중인 패킷은 실제로 본인이 사용할 output port 사용되고있지 않아도 앞에 때문에 막힘.
- 그림을 보면 실제로, 3번째 라인이 output port로 보내질 못한다.
    - 근데 순서를 바꾸면 blocking이 안 생긴다.

## Output ports

- 얼마나 나갈 수 있는가? 는 link bandwith와 연관이 있다.
- Input switch fabric이 output link bandwith보다 더 많은 속도로 들어올 수 있기에 queue가 필요하다.
- scheduling algorithm : discard policy
    - 여러 스케쥴링이 사용된다.

## Output ports queueing

- 그냥 적당히 읽고 넘어가심

## Scheduling mechanisms

- **FIFO scheduling**
    - FCFS 스케쥴링
    - Transmission Time : L/R
    - discard Policy rules
        - tail drop : 가장 마지막 drop
        - priority : 우선순위 주고 우선순위 낮은 걸 없얘기
        - random : 그냥 랜덤 - for fairness
            - (RED : random early drop)
        - head drop(Presentation엔 없음) : 젤 앞에 있는거 drop하기
            - 재전송을 갑자기 하는 경우
- **Priority Scheduling**
    - queue를 여러개 둔다. 그리고 각각의 큐에 우선순위를 둔다.
    - 요즘 general한 컴퓨터 시스템엔 이걸 잘 안쓸 것 같다.
- **Round Robing scheduling**
    - 큐가 여러개긴 한데 그 큐에서 pop하는건 돌아가면서 한다.
- **Weighted Fair Queueing**
    - 각각의 큐를 돌리긴 하지만, 가중치가 다르다 맨위는 3번, 가운덴 2번, 마지막 1번
    - priority + rr 방식

## NAT: network address translation

NAT는 내부에서 외부로 나갈 때 NAT 테이블에 저장한다.

- motivation
    - 공용 IP 문제 해결, 외부에서는 하나의 IP만 사용하는 것처럼 보임.
    - 로컬 주소 변경 자유로움, 막 바꿔도 문제 없음..!
- implementation
    - NAT 라우터는 내부 IP 패킷의 (src IP, port)을 → (NAT IP, new port)로 **변환**
    - 새로운 포트 번호로 충돌 방지
    - 이 정보를 NAT 테이블에 저장하고 외부에서 요청이 이 테이블로오면 테이블을 보고 라우팅

- 16-bit port-number field
    - 1024 ~65535 개의 연결만 가능하다. 뒤에 더 많으면 안됨..!
- NAT is controversial
    - 라우터는 레이어 3까지만 처리해야 함 근데, port를 처리함...
    - 주소 부족은 IPv6로 해결해야 함
    - 종단 간 인수 위반
        - 앱 설계자는 NAT 가능성을 고려해야 함(예: P2P 애플리케이션)
    - NAT traversal: 클라이언트가 NAT 뒤에 있는 서버에 연결하려는 경우 어떻게 해야 할까요?

## IPv6: motivation

- 기본 동기: IPv4 다 써감;;
- additional motivation
    - header format은 processing/forwading 속도를 높여준다. 몇몇 필드 제거
- ip datagram format
    - **fixed-length** 40 byte header
    - **no fragmentation** allowed
    - **no ip checksum**

## IPv6: datagram format

- proirity : 우선순위 부여
- next header  : IP의 protocol field와 동일 identify upper layer protocol for data
- flow label : 하나의 tx id같은 느낌. 그러나 flow는 정의가 잘 안됨;;

## Other changes from IPv4

- checksum 없애버려 ~
- options: 없얘버려 ~ 대신 확장헤더 사용.
- ICMP v6도 나옴

## Transition from IPv4 to IPv6

- 모든 라우터를 ipv6를 지원하도록 바꾸기? -> 불가능 그러면 어케 해야할까?
- tunneling: IPv6 datagram
    - ipv6 데이터그램을 ipv4 데이터그램 안에 박아버리기?

## Tunneling

- 신기하다. 그 중간에 있는 친구가 터널을 만들어서, 하네
- 반대로도 가능하나? 굳이 그렇게 하지 않는다.
    - 근데 그렇게 하진 않는 이유는 Ipv6 라우터는 이미 ipv4를 지원하고 있다.

## Ipv6: adoption

- 45% 정도의 라우터가 지원해준다.
- 근데 점점 증가속도가 늦어지는 추세
