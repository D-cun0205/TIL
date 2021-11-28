###인터넷 네트워크
IP

    서버에 할당되는 1개의 주소
    Internet Protocol로 지정한 IP Address에 데이터 전달
    하나의 서버에서 요청이 필요한 서버까지 많은 노드(다른 서버)들을 지나가며 전달
    패킷이라는 통신 단위로 전달하며 비연결성, 비신뢰성, 프로그램 구분 문제 발생
    비연결성 : 패킷을 받을 대상이 없거나 서비스 불능이여도 패킷 전송
    비신뢰성 : 패킷 손실, 패킷 순서 변경
    프로그램 구분 : 요청 서버의 어떤 어플리케이션의 요청인지 구분 불가 

PORT

    서버안에서 사용되는 어플리케이션 단위  
    
    PORT 범위 : 0 ~ 65535
    0 ~ 1023 포트는 잘 알려진 포트로 사용하지 않은 것이 좋음
    FTP : 20 or 21, TELNET : 23, HTTP : 80, HTTPS : 443

TCP(전송 제어 프로토콜)

    TCP 3 Way Handshake(가상 연결), 포트 사용, 순서 보장,데이터 전달 보증
    신뢰할 수 있는 프로토콜, 대부분 TCP사용, 장점으로 빠르며 단점으로 최적화를 위한 수정이 불가

UDP(사용자 데이터그램 프로토콜)
    
    IP프로토콜과 비슷하며 PORT, 체크섬(데이터 무결성)이 추가 된 형태
    단점으로 TCP의 기능들이 없으며 장점으로는 성능 최적화 가능

DNS(도메인 네임 시스템)

    도메인 네임 시스템 서버에서 도메인명과 IP주소 매핑하여 데이터로 보관하고 있으며
    특정 서버에서 통신요청을 할 때 도메인명으로 넘어온 요청을 IP주소로 변환하여 연결
    
URI(통합 자원 식별)

    식별 방법으로 URL, URN을 사용하며 UR은 동일한 예약어이며 L : Locator, N : Name
    URL : 저장 위치를 가리킴, URN : 접근하기 위한 명칭을 사용하여 접근하지만 사용하지 않음
    ex) scheme://[userinfo@]host[:port][/path][?query][#fragment]
    scheme : 프로토콜로 http, https, ftp
    userinfo@ : 사용자 인증 객체를 보내는데 거의 사용하지 않음
    host : IP Address or 도메인명
    port : 어플리케이션이 실행되고 있는 포트
    path : 어플리케이션의 컬렉션(디렉토리) 위치, 자원 경로
    query : 통신 요청시 사용되는 파라미터 query parameter 또는 query string이라고 칭함
    fragment : HTML 내부 북마크로 사용, 특정 위치로 가기위함, 서버에 전송하는 정보 아님

###HTTP
클라이언트와 서버 구조

    Request and Response 구조, 클라이언트의 요청을 서버가 받아서 결과를 클라이언트에게 전달
    
상태유지(stateful)와 무상태(stateless) 프로토콜

    stateful : 상태유지의 프로토콜을 사용하여 통신하는 서버와 클라이언트가 지속적으로 연결되어 있어야함
    단점 : 클라이언트 증가에 대한 트래픽 증가 문제를 서버 증설로 해결할 수 없음
            , 서버에 장애가 발생하면 클라이언트는 요청을 새로 시작해야 함
    stateless : 클라이언트와 서버가 고정되어 있을 필요가 없으므로 클라이언트의 요청을 불특정 서버에서 받아도 요청 처리 가능
                스케일아웃(수평 확장으로 서버를 무한정 늘리는 것을 뜻함)이 가능
    단점 : 로그인 같은 상태유지가 필요한 프로세스에 대해 대책이 없음, 요구하는 데이터가 크다

비연결성(Connectionless)

    HTTP는 기본이 연결을 유지하지 않은 모델
    연결을 유지하지 않기 때문에 요청마다 3 Way Handshake 같은 선 작업이 필요하며 요청 단위마다 처리시간 증가
    현재는 지속 연결(Persistent Connections)로 문제점 해

HTTP 메시지

    HTTP 시작 라인, HTTP 헤더, HTTP 바디 3개로 구성
    HTTP 시작 라인 : HTTP version, HTTP status, ex)HTTP/1.1 200 OK(200의 의미가 통신 성공이다라는 의미를 모두 전달하지 않기 때문에 사용)
    HTTP 헤더 : field name, field value, OWS(띄어쓰기 허용), ex) header-field = field-name ":" OWS field-value OWS
    HTTP 바디 : 실제 전송할 모든 데이터
    
###HTTP API
API URI

    (통합 자원 식별) 설계
        ex) 회원 조회, 회원 등록, 회원 수정, /read-member-?, /create-mmeber, /update-member ...
    식별의 구분값은 자원으로 시작하며 위 예에서 구분값은 동사가아닌 명사로 member가 식별값이 된다
    동사(save, update, delete ..)에 대한 구분은 HTTP 메서드로 구분

HTTP 메서드

    GET : 리소스 조회, 정적 자원에 대한 요청에 주로 사용하며 캐싱 작업에 유리
    POST : 요청 데이터 처리, 주로 등록에 사용, 캐싱 작업 불가
    PUT : 리소스 대체, 리소스 없을 시 생성, 데이터 대체로 인해 잘못된 수정이 발생할 수 있음
        ex) PUT Request : { age : 20} -> { username : 'sanghun', age : 10} username은 제거 age : 20으로 변경 
    PATCH : 리소스 부분 변경
        ex) PUT Request : { age : 20} -> { username : 'sanghun', age : 10} age 컬럼만 20으로 변경
    DELETE : 리소스 삭제

HTTP 메서드 속성

    안전 : GET호출을 제외한 데이터의 변경이 일어나거나 생성 되는 부분은 안전하지 않다
    멱등 : 사전적 의미로 연산을 여러 번 적용하더라도 결과값이 달라지지 않은 일 GET, PUT, DELETE가 해당 POST는 해당 안됨
            멱등은 외부요인으로 중간에 리소스가 변경되 것 까지 고려하지 않음
    캬시 가능 : GET, HEAD 메서드에 사용하며 POST 같은 바디 데이터가 포함된경우를 KEY로 컨트롤 하는게 쉽지 않아서 구현하지 않음

###클라이언트에서 서버로 데이터 전송
데이터 전달 방식
    
    쿼리 파라미터를 통한 데이터 전송 : GET, 정렬 필(검색어)
    메시지 바디를 통한 데이터 전송 : POST, PATCH, PUT, ex)회원가입, 리소스 등록 및 수정

조회 및 전송에 대한 4가지 예시
    
    정적 데이터 조회 : 이미지, 정적 텍스트
    동적 데이터 조회 : 검색, 게시판 목에서 정렬 필(검색어)
    HTML Form을 통한 데이터 전송 : 회원 가입, 상품 주문, 데이터 변경
        Form method GET : query string으로 url에 파라미터를 추가하여 요청
        Form method POST : content-type 설정과, HTTP Body에 내용을 추가하여 요청
        HTML Form 전송은 get, post method만 사용 가능
    HTML API를 통한 데이터 전송 : server to server, app or web client(ajax)

###HTTP API 설계 예시
POST 기반 등록(회원 관리 시스템을 에시로 사용) - 주로 사용

    리소스 식별 구분값으로 member같은 명사를 사용하며 동사(등록하다, 수저하다, 삭제하다)들은 메서드(POST, PUT, DELETE)로 대체
    클라이언트는 등록될 리소스의 URI를 모른다. 서버에서 리소스 URI를 결정하고 생성
        사용자는 신규로 등록되며 DB에서 방금 등록된 사용자의 식별값을 전달해준다는 의미를 리소스 URI라고 비유하는 것 같다
    컬렉션(Collection) : 서버가 관리하는 리소스 디렉토리, ex) 여기서 컬렉션은 /members

PUT 기반 등록(파일 관리 시스템) - 사용 비중이 거의 없음

    파일 호출 /files -> GET, 파일 조회 /files/{filename} -> GET
    파일 등록 /files/{filename} -> PUT
    파일 등록의 경우 기존에 등록된 파일이 있으면 삭제하고 새로 등록해야 되기 때문에 PUT을 사용하여 덮어쓸수 있다
    클라이언트가 리소스 URI를 알고 있거나 지정해야 한다
    스토어(store) : 클라이언트가 관리하는 리소스 저장소, ex) 여기서 스토어는 /files 

HTML FORM 사용

    Form은 GET, POST 지원
    AJAX 기술을 사용해서 HTML FORM 문제를 해결할 수 있다
    순수 HTML FORM의 경우 GET, POST만 사용하여 제약이 있다
    HTTP 메서드로 해결하기 애매한 경우 컨트롤러, 컨트롤 URI를 사용해서 제약을 해결 /new, /edit, /delete

참고하면 좋은 URI 설계 개념 : https://restfulapi.net/resource-naming

###HTTP 상태 코드
상태 코드
    
    1xx(Informational) : 요청이 수신되어 처리중 
    2xx(Successful) : 요청 정상 처리
    3xx(Redirection) : 요청을 완료하려면 추가 행동이 필요
    4xx(Client Error) : 클라이언트 오류 잘못된 문법등으로 요청 수행 불가
    5xx(Server Error) : 서버 오류, 서버가 정상 요청을 처리하지 못함
    xx의 값이 모르는 값이 나오더라도 첫번째 자리 숫자와 연관된 상태메시지를 던지니 비슷한 형태로 유추하여 진행

    400 Bad Request : 클라이언트가 잘못된 요청을 해서 서버가 요청을 처리하지 못함
        요청 구문, 메시지 오류
    401 Unauthorized : 클라이언트가 해당 리소스에 대한 인증이 필요
        인증, 인가 필요
    403 Forbidden : 서버가 요청을 이해했지만 승인을 거부
        인증 자격 증명은 있지만 접근 권한이 불충분
        사용자가 어드민 권한에 대한 리소스를 요청한 경우
    404 Not Found : 요청 리소스를 찾을 수 없음, 있지만 권한에 의해 없는 리소스인것처럼 보여주고 싶은 경우

    500 Internal Server Error : 서버 문제로 오류 발생, 애매한 경우
    503 Service Unavailable : 서비스 이용 불가, 예정 작업에 의해 화면을 보여주지 못한 경우

###HTTP 헤더 상세 개요

    HTTP 전송에 필요한 모든 부가 정보
    메시지 바디 타입, 메시지 바디 크기, 압축, 인증, 요청 클라이언트, 캐시, 관리정보 ..

    표현 : 표현 헤더 + 표현 본문
    Content-Type : 표현 데이터의 형식 
        text/html; charset=utf-8
        application/json (default : charset=utf-8)
        image/png
    Content-Encoding : 표현 데이터 인코딩
        gzip, deflate, identity(압축 안함)
    Content-Language : 표현 데이터의 자연 언어
        ko, en, en-US
    Content-Langth : 표현 데이터 길이
        byte 단위

    협상 : 컨텐츠 네고시에이션 (클라이언트가 선호하는 표현 요청) *요청시에만 사용
    Accept : 클라이언트가 선호하는 미디어 타입 전달
        ex) text/*, text/plain, text/plain;format=flowed, */* 구체적인 것이 우선 순위 적용
    Accept-Charset : 클라이언트가 선호하는 문자 인코딩
    Accept-Encoding : 클라이언트가 선호하는 압축 인코딩
    Accept-Language : 클라이언트가 선호하는 자연 언어
        ex) ko-KR,ko;q=0.9,en-US;q=0.8 q값이 클수록 높은 우선순위로 언어가 적용

###전송 방식 설명

    단순 전송 : Content-Length에 대해 알고 있을 때, 한번에 전송하고 한번에 받음
    압축 전송 : Content-Encoding 압축하여 전송하면 인코딩 타입을 지정해야 한다
    분할 전송 : Content-Length 사용 불가 Transfer-Encoding : chunked 짤라서 지속적으로 보내는 형태
    범위 전송 : Range, Content-Range, byte 크기를 지정하여 받으며 중간부터 다시 받을 수 있는 형태

###일반 정보

    From : 유저 에이전트의 이메일 정보, 요청에서 사용하며 검색 엔진 같은 곳에서 주로 사용하고 일반적으로 사용되지 않음
    referrer : 현재 요청된 페이지의 이전 웹 페이지 주소
        A -> B로 이동하는 경우 B를 요청할 때 Referrer: A를 포함해서 요청, 요청에서 사용
    User-Agent : 유저 에이전트 어플리케이션 정보 = 클라이언트 브라우저 정보
    Server : 요청을 처리하는 ORIGIN(요청을 실질적으로 처리하고 응답해주는 서버) 서버의 소프트웨어 정보, 응답에서 사용
    Date : 응답 메시지의 날짜 정보

###특별한 정보

    