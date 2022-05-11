# LocalWebRelayStreaming
CCTV &lt;=> Local Relay-Server &lt;=> Client 간 환경 구성에 대한 내용. (RTSP to HTTP Streaming)


## Test Relay-Server 환경 구성

# 준비물
1. CCTV
2. Server (Linux Ubuntu)
3. Client (Windows 10, Chrome)


# 동작 과정
CCTV[RTSP] => Server(RTSP to HTTP) => Client


# 순서
1. CCTV를 구동시킨다.
2. 서버를 구동시킨다. (VLC 사용)
    - Windows based Server
        vlc "{CCTV URL}" --sout "#transcode{vcodec=theo,vb=800,acodec=vorb,ab=128,channels=2,samplerate=44100,scodec=none,fps=30}:http{mux=ogg,dst=:8182/test}" --no-sout-all --sout-keep

    - Linux (Ubuntu)
        vlc -vvv "{CCTV URL}" --sout "#transcode{vcodec=theo,vb=800,acodec=vorb,ab=128,channels=2,samplerate=44100,scodec=none,fps=30}:http{mux=ogg,dst=:8182/test}" --no-sout-all --sout-keep
        
    - Linux (Red Hat)
        cvlc -I http -vvv "{CCTV URL}" --sout "#transcode{vcodec=theo,vb=800,acodec=vorb,ab=128,channels=2,samplerate=44100,scodec=none,fps=30}:http{mux=ogg,dst=:8182/test}" --no-sout-all --sout-keep
3. 클라이언트에서 Chrome으로 접근한다.
        <video id="video" width=1080 height=480 muted="muted">
            <source src="http://{Server IP}:8182/test" type="video/ogg; codecs=theora"/>
        </video>

단, Server IP가 공인 IP 등 신뢰된 URL이 아닐 경우 CORS 에러가 발생한다. (AWS에서는 정상동작)
CORS에러가 발생할 경우 다음 순서대로 환경을 설정한다.

  1) Client 측 Chrome에 "chrome://flags/#block-insecure-private-network-requests"에 접속하여, "Block insecure private network requests."를 Disabled로 변경한다.
  2) Server 측에 최신 VLC 코드를 다운받아 컴파일 환경을 구성한다.(참조 : https://techpiezo.com/linux/install-vlc-in-ubuntu-20-04-lts/)
  3) Server 측에 최신 VLC 코드 중 src > network > httpd.c > httpd_StreamNew 함수 중간에 다음 코드를 추가한다.
    stream->i_http_headers = 2;
    stream->p_http_headers = vlc_alloc(stream->i_http_headers, sizeof(httpd_header));
    stream->p_http_headers[0].name = strdup("Access-Control-Allow-Origin");
    stream->p_http_headers[0].value = strdup("*");
    stream->p_http_headers[1].name = strdup("Access-Control-Allow-Credentials");
    stream->p_http_headers[1].value = strdup("true");
  4) Server 측에 VLC를 컴파일한다. (참조 URL 참고. ./configure, make, sudo make install)

