PolarDPI - DPI/SNI 감청, 감시 차단
=========================

이 소프트웨어는 ISP 중 DPI/SNI 시스템으로 일부 사이트를 차단하는 것을 우회하는 목적으로 개설되었습니다.

어떠한 데이터도 차단하지 않고 요청된 목적지보다 빠르게 회신하는 광스플리터나 포트 미러링(**패시브 DPI**)을 이용해 접속된 DPI, 그리고 순차적으로 연결된 **액티브 DPI**를 처리합니다.

관리자 권한이 있는 **Windows 7, 8, 8.1 또는 10**이 설치된 컴퓨터가 필요합니다.

# 사용법

[여기서 최신 버전을 받고](https://github.com/polardpi/polardpi/releases) 실행하세요.

```
사용법: polardpi.exe [옵션]
 -p 패시브 DPI 
 -r host를 host로 대체한다.
 -s 호스트 헤더와 값 사이의 공간 제거
 -m mix Host 헤더 케이스 (test.com -> tEsT.cOm)
 -f [value] HTTP fragment를 value로 설정
 -k [value] HTTP 영구(keep-alive) 단편화를 활성화하고 값으로 설정
 -k가 활성화되면 첫 번째 세그먼트 ACK를 기다리지 않음
 -e [value] HTTPS 조각화를 값으로 설정
 -Method와 Request-URI 사이의 추가 공간(enables -s, 사이트 파손 가능)
 -w 모든 처리 포트에서 HTTP 트래픽을 찾아 구문 분석(포트 80은 아님)
 --value [value] 추가 TCP 포트(및 -w로 HTTP tricks)를 실행함)
 --ip-id [값]은 추가 IP ID(십진수, 드롭 리디렉션 및 이 ID의 TCP RST)를 처리한다.
 이 옵션은 여러 번 공급할 수 있다.
 --dns-addr [값] UDP DNS 요청을 제공된 IP 주소(실험적)로 리디렉션
 --dns-port [value] UDP DNS 요청을 공급 포트로 리디렉션(기본값 53)
 --dnsv6-addr [값] UDPv6 DNS 요청을 제공된 IPv6 주소로 리디렉션(실험)
 --dnsv6-port [value] UDPv6 DNS 요청을 공급 포트로 리디렉션(기본값 53)
 --dns-동사 인쇄 상세 DNS 리디렉션 메시지
 --blacklist[txtfile]는 호스트 이름과 하위 도메인에만 HTTP 트릭을 실행한다.
 공급된 텍스트 파일 이 옵션은 여러 번 공급할 수 있다.

 -1 -p -r -s -f 2 -k 2 -n -e 2 (가장 호환 가능한 모드, 기본값)
 -2 -p -r -s -f 2 -k 2 -n -e 40 (HTTPS의 속도가 더 빠르지만 여전히 호환 가능)
 -3 -p -r -s -e 40 (HTTP 및 HTTPS의 경우 속도가 더 빠름)
 -4 -p -r -s (최상의 속도)
```
ISP의 DPI를 회피할 수 있는지 확인하려면 먼저 '3_all_dnsredir_hardcore.cmd'를 실행하십시오. 이것은 이 프로그램이 당신의 ISP와 DPI 벤더에 적합한지 여부를 보여줄 가장 하드코어 모드 입니다. 만약 당신이 이 모드로 차단된 웹사이트를 열 수 있다면, 그것은 당신의 ISP가 피할 수 있는 DPI를 가지고 있다는 것을 의미한다. 이것은 웹 사이트 모드가 가장 느리고 깨지기 쉽지만 대부분의 DPI에 적합하다.

goodbyedpi -1을 사용해 보십시오.

그럼 굿바이드피(goodbyedpi)를 먹어봐.-2를 이탈하다 HTTPS 사이트에서는 더 빨라야 한다. 모드 '-3'은 HTTP 웹사이트의 속도를 높인다.

goodbyedpi를 사용한다.ISP의 DPI에 적용된다면 -4. 이것은 가장 빠른 모드지만 모든 DPI와는 호환되지 않는다.

# 작동 방법
### 패시브 DPI

대부분의 수동적 DPI는 HTTP 302 리디렉션을 전송하는데, 만약 당신이 HTTPS의 경우, HTTP와 TCP 리셋을 통해 차단된 웹 사이트에 접속하려고 하면, 목적지 웹 사이트보다 더 빨리 전송된다. DPI가 전송하는 패킷은 대개 러시아 통신사에서 보듯이 0x000000이나 0x0001에 해당하는 IP 식별 필드를 가진다. 이 패킷들은, 만약 그들이 당신을 다른 웹사이트로 리디렉션한다면, PolarDPI에 의해 차단된다.

### 액티브 DPI

능동적 DPI는 속이기 더 까다롭다. 현재 소프트웨어는 6가지 방법을 사용하여 액티브 DPI를 회피한다.

* 첫 번째 데이터 패킷에 대한 TCP 수준 조각화
* 지속적(keep-alive) HTTP 세션을 위한 TCP-level fragment
* '호스트' 헤더를 '호스트'로 교체
* '호스트' 헤더에서 헤더 이름과 값 사이의 공간 제거
* HTTP Method(GET, POST 등)와 URI 사이에 추가 공간 추가
* Host 헤더 값의 혼합 케이스

이러한 방법은 TCP 및 HTTP 표준과 완전히 호환되므로 어떤 웹사이트도 파괴해서는 안 되지만, DPI 데이터 분류를 방지하고 검열을 피하기에는 충분하다. 추가 공간은 HTTP/1.1 규격으로 허용되지만 일부 웹 사이트를 파괴할 수 있다(19.3 허용오차 응용 프로그램 참조).

이 프로그램은 윈도우즈 필터링 플랫폼을 사용하여 필터를 설정하고 패킷을 사용자 공간으로 리디렉션하는 WinDivert 드라이버를 로드한다. 콘솔 창이 보이는 한 실행되며 창을 닫으면 종료된다.

# 감사의 말씀

Thanks @basil00 for [WinDivert](https://github.com/basil00/Divert). That's the main part of this program.

Thanks for every [BlockCheck](https://github.com/ValdikSS/blockcheck) contributor. It would be impossible to understand DPI behaviour without this utility

Thanks to @ValdikSS for [GoodByeDPI](https://github.com/ValdikSS/GoodbyeDPI).
