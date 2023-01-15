---
published: true
title:  "[Programming] C++ Boost asio/beast"
excerpt: "Boost asio와 beast에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, Boost, Asio, Beast]

toc: true
toc_sticky: true
 
date: 2023-01-15
last_modified_at: 2023-01-15
---

## 1. ASIO(ASynchronous IO)

- Cross-platform C++ library
- Windows에서는 IOCP를 사용하고, Linux에서는 EPOLL을 사용
- 이 둘은 호환성이 없었는데 C++11에서 ASIO라는 것이 들어가면서 통일하게 됨
- thread 또한 WIN32 API에 thread 와 Linux의 pthread 라이브러리가 있는데 이부분도 C++ thread가 나오면서 OS와 관계없이 멀티스레드 프로그래밍을 할 수 있게 되었음

### 1.1. 동기식 (사용가능)

- 준비 : io_context, socket

```cpp
boost::asio::io_context io_context;
boost::asio::ip::tcp::socket socket(io_context); // asio에서 지원하는 socket 클래스
```

![image](https://user-images.githubusercontent.com/23397039/212526345-a293123a-aee9-4f3a-96ae-9a43dca8be75.png){: .align-center}

- 동작 방식
1. socket API 호출

```cpp
socket.connect(server_endpoint);
```

1. socket이 요청을 io_context에 전달
2. io_context가 OS를 호출
3. OS가 결과를 io_context로 리턴
4. io_context가 에러 처리 후 socket에 결과를 전달
5. socket은 결과를 프로그램에 전달

### 1.2. 비동기 (unblock, async)

![image](https://user-images.githubusercontent.com/23397039/212526361-a8356ab0-4c8b-4f42-9ef6-4ceb908a150a.png){: .align-center}

- 동작 방식
1. socket API 호출 (handle_connect, handle_read, handle_write와 같은 handler와 함께 등록)

```cpp
socket.async_connect(server_endpoint, handler); // handler는 connect 후 콜백 호출
void handler(const boost::system::error_code& ec);
```

1. socket이 io_context에 전송
2. io_context가 OS를 호출 (동기식 과정과 다르게 리턴 없이 다른 작업으로 넘어감)

![image](https://user-images.githubusercontent.com/23397039/212526374-8b374f86-0bb2-4189-a825-3496accc12e4.png){: .align-center}

1. OS가 결과를 queue에 넣고 io_context에 완료를 알려줌
2. 프로그램에서 io_context::run()을 호출, run()이 완료를 발견할 때까지 blocking
    1. 프로그램에서 Queue에 쌓인 결과들을 뽑아서 처리해야 되기 때문에
    2. run() 은 Queue가 비어있으면 기다리고 OS에서 Queue에 넣어주면 run은 handler를 실행하고 결과를 리턴받음
    3. asio를 쓰면 run은 별도의 thread를 돌리고 io는 다른 thread에서 해줌 
3. run()이 완료를 발견하면 결과를 queue에서 빼낸 후 handler에 전달

### 1.3. 비동기 구조 : Proactor 패턴

- 내부에 event queue를 갖고, epoll과 같은 Reactor 패턴도 Proactor 패턴으로 변환
    
![image](https://user-images.githubusercontent.com/23397039/212526395-13157402-7dde-408f-8a06-c62fa40018a7.png){: .align-center}

- Proactor 패턴은 job이 있을 때, job이 직접 함수를 호출하면서 돌아가는 프로그램 방식
    - epoll과 같은 Reactor 패턴은 loop를 돌면서 데이터가 오는지 감시하다가, 작업이 오면 실행하고, 또 검사하면서 실행하는 방식
- IOCP는 proactor 패턴
    - 소켓에서 데이터가 오면 워커스레드를 깨움
- epoll은 루프를 돌면서 누가 왔나 감시하는 게 reactor 패턴인데, asio를 사용하면 Linux에서도 Proactor 패턴으로 사용됨
- 사실 Linux 내부에서는 epoll로 돌아가는데 run()을 시켜두면 내부적으로 Proactor 방식처럼 실행
- IOCP와의 차이

```cpp
// IOCP
// GQCS를 보고 recv/send 인지 봐서 버퍼에 쌓인 데이터를 처리
worker_thread()
{
    GQCS( g_iocp , ... );
    if (is_recv)
        // 워커스레드에서 ProcessBuufer를 호출해서 일이 다 끝나면 WSARecv를 다시 호출
        ProcessBuffer();
}
// 처리가 끝나면 다시 WSARecv를 함
WSARecv (sock, WSABUF);

// ASIO
// 워커스레드는 따로 데이터 처리를 하지 않고 그냥 run만 시킴
worker_thread()
{
	// 스레드 등록, 큐에 완료가 들어오면 뭔지 봐서 해당 핸들러를 찾아 호출해주는 역할도 다 해줌
    io_context->run();
}
// recv는 async_read를 통해 함
// 데이터가 들어오면 버퍼에 넣어줌, ProcessBuffer와 같은 콜백 handler를 넣어줘서
// 작업을 수행한 후, 처리가 끝나면 async_read를 호출
async_read (sock, buffers, ProcessBuffer);
```

### 1.4. 멀티스레드

- io_context 객체에 대한 모든 접근은 thread safe 함
- 멀티스레드에서 io_context::run()을 호출함으로서 병렬성을 얻음
- 내부적으로 비동기 처리를 위해 별도의 스레드를 사용할 수 있는데, 이는 동작을 최소화 되어있고 사용자에게 숨겨져 있음

### 1.5. Strand

- 어떤 작업을 핸들러 단위로 쪼개서 멀티스레드를 수행하는 것
- 핸들러가 여러 스레드에서 동시에 처리될 때 순서가 뒤바뀌는 것을 막기 위한 구조
- 핸들러를 같은 strand에 등록하면 handler의 호출이 직렬화됨
    - Linux의 경우 내부적으로 mutex를 사용해서 직렬화시킴 (성능 저하의 주범)
- Room을 핸들러 단위로 하고, 명시적인 Lock 없이 멀티스레딩 수행이 가능해짐

![image](https://user-images.githubusercontent.com/23397039/212526412-d5207610-11ca-4af9-a4ce-f9f82fde2ed5.png){: .align-center}

- Lock을 사용하면 위의 그림과 같은 핸들러를 동기화 시키기 위해 끝날 때까지 대기

![image](https://user-images.githubusercontent.com/23397039/212526424-a89a03a9-ffb5-4f18-b5cf-fb6cc2c619da.png){: .align-center}

- Strand를 사용하면 같은 핸들러(room)가 겹치지 않게 Strand가 알아서 함수를 실행시켜주고 호출 순서도 보장해줌

## 2. Boost beast

- Boost.Asio의 비동기 모델을 사용하여 low-level의 HTTP/1, WebSocket 및 네트워킹 프로토콜을 제공함으로써 상호 운용 가능한 네트워킹 라이브러리 역할을 하는 C++ 전용 라이브러리
- C++에서 REST API를 사용하는 방법
- Beast는 서버 및 클라이언트를 생성 할 수 있도록 지원
- HTTP, WebSocket은 WWW에서 구동 가능
- 모든 웹 브라우저는 이러한 프로토콜을 구현하여 웹 페이지를 로드하고 클라이언트 측 프로그램이 대화식으로 통신할 수 있음
    - coroutine은 spawn 으로 생성하며 [boost::asio::spawn](https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio/reference/spawn.html), 프로그램에 [Boost.Coroutine](https://www.boost.org/doc/libs/1_81_0/libs/coroutine/index.html) 라이브러리 추가해야 함
    - [boost::asio::ssl::stream](https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio/reference/ssl__stream.html) 을 사용할 때는, 프로그램에 [OpenSSL](https://www.openssl.org/) 라이브러리를 추가해야 함

### 2.1. HTTP client 예제
- [http_client example](https://www.boost.org/doc/libs/1_79_0/libs/beast/doc/html/beast/quick_start/http_client.html)

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/connect.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <cstdlib>
#include <iostream>
#include <string>

namespace beast = boost::beast;     // from <boost/beast.hpp>
namespace http = beast::http;       // from <boost/beast/http.hpp>
namespace net = boost::asio;        // from <boost/asio.hpp>
using tcp = net::ip::tcp;           // from <boost/asio/ip/tcp.hpp>

// HTTP GET을 수행하고 response를 출력하는 코드
int main(int argc, char** argv)
{
    try
    {
        // 커맨드라인 arg 체크
        if(argc != 4 && argc != 5)
        {
            std::cerr <<
                "Usage: http-client-sync <host> <port> <target> [<HTTP version: 1.0 or 1.1(default)>]\n" <<
                "Example:\n" <<
                "    http-client-sync www.example.com 80 /\n" <<
                "    http-client-sync www.example.com 80 / 1.0\n";
            return EXIT_FAILURE;
        }
        auto const host = argv[1];
        auto const port = argv[2];
        auto const target = argv[3];
        int version = argc == 5 && !std::strcmp("1.0", argv[4]) ? 10 : 11;

        // 모든 I/O는 io_context를 사용O
        net::io_context ioc;

        // I/O를 수행할 objects
        tcp::resolver resolver(ioc);
        beast::tcp_stream stream(ioc);

        // Look up the domain name
        auto const results = resolver.resolve(host, port);

        // lookup에서 얻은 주소로 connection을 만드는 작업
        stream.connect(results);

        // HTTP GET request 메시지 셋업
        http::request<http::string_body> req{http::verb::get, target, version};
        req.set(http::field::host, host);
        req.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);

        // HTTP request 메시지를 remote host에게 보냄 
        http::write(stream, req);

        // 버퍼는 읽기용이며 지속적임
        beast::flat_buffer buffer;

        // reponse 메시지를 보관할 컨테이너
        http::response<http::dynamic_body> res;

        // HTTP response를 받음
        http::read(stream, buffer, res);

        // response 메시지를 출력
        std::cout << res << std::endl;

        // 소켓을 닫음
        beast::error_code ec;
        stream.socket().shutdown(tcp::socket::shutdown_both, ec);
    }
    catch(std::exception const& e)
    {
        std::cerr << "Error: " << e.what() << std::endl;
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

### 2.2. WebSocket 예제

- [websocket_client example](https://www.boost.org/doc/libs/1_79_0/libs/beast/doc/html/beast/quick_start/websocket_client.html)

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/websocket.hpp>
#include <boost/asio/connect.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <cstdlib>
#include <iostream>
#include <string>

namespace beast = boost::beast;         // from <boost/beast.hpp>
namespace http = beast::http;           // from <boost/beast/http.hpp>
namespace websocket = beast::websocket; // from <boost/beast/websocket.hpp>
namespace net = boost::asio;            // from <boost/asio.hpp>
using tcp = boost::asio::ip::tcp;       // from <boost/asio/ip/tcp.hpp>

// 웹소켓 메시지를 보내고 reponse를 출력하는 동작
int main(int argc, char** argv)
{
    try
    {
        // 커맨드라인 args 체크
        if(argc != 4)
        {
            std::cerr <<
                "Usage: websocket-client-sync <host> <port> <text>\n" <<
                "Example:\n" <<
                "    websocket-client-sync echo.websocket.org 80 \"Hello, world!\"\n";
            return EXIT_FAILURE;
        }
        std::string host = argv[1];
        auto const  port = argv[2];
        auto const  text = argv[3];

        // 마찬가지로 I/O를 위한 io_context
        net::io_context ioc;

        // I/O를 수행항 objects
        tcp::resolver resolver{ioc};
        websocket::stream<tcp::socket> ws{ioc}; // 웹소켓 객체

        // Look up the domain name
        auto const results = resolver.resolve(host, port);

        // lookup한 주소로 connection을 생성, HTTP와 달리 ws layer도 함께 전달
        auto ep = net::connect(ws.next_layer(), results);

				// host 문자열을 업데이트, 웹소켓 handshake 동안은 host HTTP 헤더를 사용
        // See https://tools.ietf.org/html/rfc7230#section-5.4
        host += ':' + std::to_string(ep.port());

        // handshake의 user_agent를 변경하는 decorator 설정
        ws.set_option(websocket::stream_base::decorator(
            [](websocket::request_type& req)
            {
                req.set(http::field::user_agent,
                    std::string(BOOST_BEAST_VERSION_STRING) +
                        " websocket-client-coro");
            }));

        // 웹소켓 handshake 수행
        ws.handshake(host, "/");

        // 메세지를 보냄
        ws.write(net::buffer(std::string(text)));

        // incoming message를 보관할 버퍼
        beast::flat_buffer buffer;

        // 버퍼의 메시지를 읽음
        ws.read(buffer);

        // 웹소켓 connection을 닫음
        ws.close(websocket::close_code::normal);

        // 버퍼의 데이터를 출력
        std::cout << beast::make_printable(buffer.data()) << std::endl;
    }
    catch(std::exception const& e)    {
        std::cerr << "Error: " << e.what() << std::endl;
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

## 3. 코드로 알아보기

### 3.1. HTTP, Server asynchronous

- [http_server_async example](https://www.boost.org/doc/libs/1_79_0/libs/beast/example/http/server/async/http_server_async.cpp)

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/dispatch.hpp>
#include <boost/asio/strand.hpp>
#include <boost/config.hpp>
#include <algorithm>
#include <cstdlib>
#include <functional>
#include <iostream>
#include <memory>
#include <string>
#include <thread>
#include <vector>

namespace beast = boost::beast;         // from <boost/beast.hpp>
namespace http = beast::http;           // from <boost/beast/http.hpp>
namespace net = boost::asio;            // from <boost/asio.hpp>
using tcp = boost::asio::ip::tcp;       // from <boost/asio/ip/tcp.hpp>

// 파일의 확장자를 기반으로 적당한 mime type return
beast::string_view
mime_type(beast::string_view path)
{
    using beast::iequals;
    auto const ext = [&path]
    {
        auto const pos = path.rfind(".");
        if(pos == beast::string_view::npos)
            return beast::string_view{};
        return path.substr(pos);
    }();
    if(iequals(ext, ".htm"))  return "text/html";
    if(iequals(ext, ".html")) return "text/html";
    if(iequals(ext, ".php"))  return "text/html";
    if(iequals(ext, ".css"))  return "text/css";
    if(iequals(ext, ".txt"))  return "text/plain";
    if(iequals(ext, ".js"))   return "application/javascript";
    if(iequals(ext, ".json")) return "application/json";
    if(iequals(ext, ".xml"))  return "application/xml";
    if(iequals(ext, ".swf"))  return "application/x-shockwave-flash";
    if(iequals(ext, ".flv"))  return "video/x-flv";
    if(iequals(ext, ".png"))  return "image/png";
    if(iequals(ext, ".jpe"))  return "image/jpeg";
    if(iequals(ext, ".jpeg")) return "image/jpeg";
    if(iequals(ext, ".jpg"))  return "image/jpeg";
    if(iequals(ext, ".gif"))  return "image/gif";
    if(iequals(ext, ".bmp"))  return "image/bmp";
    if(iequals(ext, ".ico"))  return "image/vnd.microsoft.icon";
    if(iequals(ext, ".tiff")) return "image/tiff";
    if(iequals(ext, ".tif"))  return "image/tiff";
    if(iequals(ext, ".svg"))  return "image/svg+xml";
    if(iequals(ext, ".svgz")) return "image/svg+xml";
    return "application/text";
}

// 로컬 파일시스템 패스에 HTTP rel-path를 추가
// return 패스는 플랫폼에 대해 normalized
std::string
path_cat(
    beast::string_view base,
    beast::string_view path)
{
    if(base.empty())
        return std::string(path);
    std::string result(base);
#ifdef BOOST_MSVC
    char constexpr path_separator = '\\';
    if(result.back() == path_separator)
        result.resize(result.size() - 1);
    result.append(path.data(), path.size());
    for(auto& c : result)
        if(c == '/')
            c = path_separator;
#else
    char constexpr path_separator = '/';
    if(result.back() == path_separator)
        result.resize(result.size() - 1);
    result.append(path.data(), path.size());
#endif
    return result;
}

// request에 대한 HTTP response를 생성하는 함수
// response 객체는 request에 따라 다르기 때문에 앞서 만든 람다를 전달
template<
    class Body, class Allocator,
    class Send>
void
handle_request(
    beast::string_view doc_root,
    http::request<Body, http::basic_fields<Allocator>>&& req,
    Send&& send) // send는 람다 
{
    // bad request 인 경우 return
    auto const bad_request =
    [&req](beast::string_view why)
    {
        http::response<http::string_body> res{http::status::bad_request, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = std::string(why);
        res.prepare_payload();
        return res;
    };

    // response를 찾을 수 없는 경우 return
    auto const not_found =
    [&req](beast::string_view target)
    {
        http::response<http::string_body> res{http::status::not_found, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = "The resource '" + std::string(target) + "' was not found.";
        res.prepare_payload();
        return res;
    };

    // 서버 error response 인 경우 return
    auto const server_error =
    [&req](beast::string_view what)
    {
        http::response<http::string_body> res{http::status::internal_server_error, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = "An error occurred: '" + std::string(what) + "'";
        res.prepare_payload();
        return res;
    };

    // method를 처리할 수 있는지 확인
    if( req.method() != http::verb::get &&
        req.method() != http::verb::head)
        return send(bad_request("Unknown HTTP-method"));

    // Request 패스는 ".."를 포함하지 않아야 함
    if( req.target().empty() ||
        req.target()[0] != '/' ||
        req.target().find("..") != beast::string_view::npos)
        return send(bad_request("Illegal request-target"));

    // request 파일에 대한 패스 만듬
    std::string path = path_cat(doc_root, req.target());
    if(req.target().back() == '/')
        path.append("index.html");

    // 파일 열기 시도
    beast::error_code ec;
    http::file_body::value_type body;
    body.open(path.c_str(), beast::file_mode::scan, ec);

    // 파일이 존재하지않는 경우에 대한 핸들링
    if(ec == beast::errc::no_such_file_or_directory)
        return send(not_found(req.target()));

    // unknown 에러에 대한 핸들링
    if(ec)
        return send(server_error(ec.message()));

    auto const size = body.size();

    // HTTP request에 대한 응답
    if(req.method() == http::verb::head)
    {
        http::response<http::empty_body> res{http::status::ok, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, mime_type(path));
        res.content_length(size);
        res.keep_alive(req.keep_alive());
        return send(std::move(res)); // 콜백, 람다
    }

    // GET request에 대한 응답
    http::response<http::file_body> res{
        std::piecewise_construct,
        std::make_tuple(std::move(body)),
        std::make_tuple(http::status::ok, req.version())};
    res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
    res.set(http::field::content_type, mime_type(path));
    res.content_length(size);
    res.keep_alive(req.keep_alive());
    return send(std::move(res)); // 콜백, 람다
}

//------------------------------------------------------------------------------

// Report a failure
void
fail(beast::error_code ec, char const* what)
{
    std::cerr << what << ": " << ec.message() << "\n";
}

// HTTP 서버 connection 핸들링
class session : public std::enable_shared_from_this<session>
{
    // HTTP 메시지를 보내기 위한 함수객체
    struct send_lambda
    {
        session& self_;

        explicit
        send_lambda(session& self)
            : self_(self)
        {
        }

        template<bool isRequest, class Body, class Fields>
        void
        operator()(http::message<isRequest, Body, Fields>&& msg) const
        {
						// 메시지의 수명은 async 수행기간동안 연장되어야 하므로 shared_ptr로 관리
            auto sp = std::make_shared<
                http::message<isRequest, Body, Fields>>(std::move(msg));

						// type-erased shared pointer를 클래스에 저장하여 유지
            self_.res_ = sp;

            // response 메시지를 write
            http::async_write(
                self_.stream_,
                *sp,
                beast::bind_front_handler(
                    &session::on_write,
                    self_.shared_from_this(),
                    sp->need_eof()));
        }
    };

    beast::tcp_stream stream_;
    beast::flat_buffer buffer_;
    std::shared_ptr<std::string const> doc_root_;
    http::request<http::string_body> req_;
    std::shared_ptr<void> res_;
    send_lambda lambda_;

public:
    // stream에 대한 ownership을 가짐
    session(
        tcp::socket&& socket,
        std::shared_ptr<std::string const> const& doc_root)
        : stream_(std::move(socket))
        , doc_root_(doc_root)
        , lambda_(*this)
    {
    }

    // asynchronous 동작 실행
    void
    run()
    {
				// I/O 객체에 대한 async 작업을 수행하기 위해 strand 내에서 실행, thread-safe
        net::dispatch(stream_.get_executor(),
                      beast::bind_front_handler(
                          &session::do_read,
                          shared_from_this()));
    }

    void
    do_read()
    {
				// 읽기전에 메시지를 empty로 설정
        req_ = {};

        // 타임아웃 설정
        stream_.expires_after(std::chrono::seconds(30));

        // request 메시지를 읽음
        http::async_read(stream_, buffer_, req_,
            beast::bind_front_handler(
                &session::on_read,
                shared_from_this()));
    }

    void
    on_read(
        beast::error_code ec,
        std::size_t bytes_transferred)
    {
        boost::ignore_unused(bytes_transferred);

        // connection을 닫음
        if(ec == http::error::end_of_stream)
            return do_close();

        if(ec)
            return fail(ec, "read");

        // HTTP response를 보냄
        handle_request(*doc_root_, std::move(req_), lambda_);
    }

    void
    on_write(
        bool close,
        beast::error_code ec,
        std::size_t bytes_transferred)
    {
        boost::ignore_unused(bytes_transferred);

        if(ec)
            return fail(ec, "write");

        if(close)
        {
            // 소켓을 닫음
            return do_close();
        }

        // response 완료되었으므로 삭제
        res_ = nullptr;

        // 또 다른 request를 읽음
        do_read();
    }

    void
    do_close()
    {
        // TCP shutdown 을 보냄
        beast::error_code ec;
        stream_.socket().shutdown(tcp::socket::shutdown_send, ec);
    }
};

//------------------------------------------------------------------------------

// incoming connection을 accept하고 세션을 실행하기 위한 listener 클래스
class listener : public std::enable_shared_from_this<listener>
{
    net::io_context& ioc_;
    tcp::acceptor acceptor_;
    std::shared_ptr<std::string const> doc_root_;

public:
    listener(
        net::io_context& ioc,
        tcp::endpoint endpoint,
        std::shared_ptr<std::string const> const& doc_root)
        : ioc_(ioc)
        , acceptor_(net::make_strand(ioc))
        , doc_root_(doc_root)
    {
        beast::error_code ec;

        // acceptor 오픈
        acceptor_.open(endpoint.protocol(), ec);
        if(ec)
        {
            fail(ec, "open");
            return;
        }

        // acceptor에서 address 재사용을 허용하도록 옵션 설정
        acceptor_.set_option(net::socket_base::reuse_address(true), ec);
        if(ec)
        {
            fail(ec, "set_option");
            return;
        }

        // 서버 주소(endpoint)로 bind 실행
        acceptor_.bind(endpoint, ec);
        if(ec)
        {
            fail(ec, "bind");
            return;
        }

        // connections에 대한 listen 실행
        acceptor_.listen(
            net::socket_base::max_listen_connections, ec);
        if(ec)
        {
            fail(ec, "listen");
            return;
        }
    }

    // accept 된 incoming connections를 실행
    void
    run()
    {
        do_accept();
    }

private:
    void
    do_accept()
    {
        // 새로운 connection은 자신만의 strand를 가짐
        acceptor_.async_accept(
            net::make_strand(ioc_),
            beast::bind_front_handler(
                &listener::on_accept,
                shared_from_this()));
    }

    void
    on_accept(beast::error_code ec, tcp::socket socket)
    {
        if(ec)
        {
            fail(ec, "accept");
            return; // To avoid infinite loop
        }
        else
        {
            // 세션을 생성하고 실행
            std::make_shared<session>(
                std::move(socket),
                doc_root_)->run();
        }

        // 또 다른 connection을 accept!
        do_accept();
    }
};

//------------------------------------------------------------------------------
// 서버 구동을 위한 동작
int main(int argc, char* argv[])
{
    // 커맨드라인 args 체크
    if (argc != 5)
    {
        std::cerr <<
            "Usage: http-server-async <address> <port> <doc_root> <threads>\n" <<
            "Example:\n" <<
            "    http-server-async 0.0.0.0 8080 . 1\n";
        return EXIT_FAILURE;
    }
    auto const address = net::ip::make_address(argv[1]);
    auto const port = static_cast<unsigned short>(std::atoi(argv[2]));
    auto const doc_root = std::make_shared<std::string>(argv[3]);
    auto const threads = std::max<int>(1, std::atoi(argv[4]));

    // io_context 생성
    net::io_context ioc{threads};

    // listener 객체 생성 및 실행
    std::make_shared<listener>(
        ioc,
        tcp::endpoint{address, port},
        doc_root)->run();

    // 요청된 스레드에서 I/O 서비스 러닝 
    std::vector<std::thread> v;
    v.reserve(threads - 1);
    for(auto i = threads - 1; i > 0; --i)
        v.emplace_back(
        [&ioc]
        {
            ioc.run();
        });
    ioc.run();

    return EXIT_SUCCESS;
}
```

### 3.2. HTTP Client asynchronous

- [http_client_async example](https://www.boost.org/doc/libs/1_79_0/libs/beast/example/http/client/async/http_client_async.cpp)

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/strand.hpp>
#include <cstdlib>
#include <functional>
#include <iostream>
#include <memory>
#include <string>

namespace beast = boost::beast;         // from <boost/beast.hpp>
namespace http = beast::http;           // from <boost/beast/http.hpp>
namespace net = boost::asio;            // from <boost/asio.hpp>
using tcp = boost::asio::ip::tcp;       // from <boost/asio/ip/tcp.hpp>

//------------------------------------------------------------------------------

// Report a failure
void
fail(beast::error_code ec, char const* what)
{
    std::cerr << what << ": " << ec.message() << "\n";
}

// HTTP GET를 수행하고 response를 출력하는 session 클래스
class session : public std::enable_shared_from_this<session>
{
    tcp::resolver resolver_;
    beast::tcp_stream stream_;
    beast::flat_buffer buffer_; // 읽기를 위한 지속적인 버퍼
    http::request<http::empty_body> req_;
    http::response<http::string_body> res_;

public:
		// 객체는 handlers가 동시에 실행되지 않도록 strand와 함께 생성
    explicit
    session(net::io_context& ioc)
        : resolver_(net::make_strand(ioc))
        , stream_(net::make_strand(ioc))
    {
    }

    // asynchronous 동작 실행
    void
    run(
        char const* host,
        char const* port,
        char const* target,
        int version)
    {
        // HTTP GET request 메시지 셋업
        req_.version(version);
        req_.method(http::verb::get);
        req_.target(target);
        req_.set(http::field::host, host);
        req_.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);

        // Look up the domain name
        resolver_.async_resolve(
            host,
            port,
            beast::bind_front_handler(
                &session::on_resolve,
                shared_from_this()));
    }

    void
    on_resolve(
        beast::error_code ec,
        tcp::resolver::results_type results)
    {
        if(ec)
            return fail(ec, "resolve");

        // 함수 동작 중에 타임아웃 설정
        stream_.expires_after(std::chrono::seconds(30));

        // 룩업된 주소로 connection 생성
        stream_.async_connect(
            results,
            beast::bind_front_handler(
                &session::on_connect,
                shared_from_this()));
    }

    void
    on_connect(beast::error_code ec, tcp::resolver::results_type::endpoint_type)
    {
        if(ec)
            return fail(ec, "connect");

        // 타임아웃 설정
        stream_.expires_after(std::chrono::seconds(30));

        // HTTP request를 remote host로 보냄
        http::async_write(stream_, req_,
            beast::bind_front_handler(
                &session::on_write,
                shared_from_this()));
    }

    void
    on_write(
        beast::error_code ec,
        std::size_t bytes_transferred)
    {
        boost::ignore_unused(bytes_transferred);

        if(ec)
            return fail(ec, "write");
        
        // HTTP response를 받음
        http::async_read(stream_, buffer_, res_,
            beast::bind_front_handler(
                &session::on_read,
                shared_from_this()));
    }

    void
    on_read(
        beast::error_code ec,
        std::size_t bytes_transferred)
    {
        boost::ignore_unused(bytes_transferred);

        if(ec)
            return fail(ec, "read");

        // 읽은 response 메시지 출력
        std::cout << res_ << std::endl;

        // 소켓을 닫음
        stream_.socket().shutdown(tcp::socket::shutdown_both, ec);

        // not_connected 에 대해 처리
        if(ec && ec != beast::errc::not_connected)
            return fail(ec, "shutdown");
    }
};

//------------------------------------------------------------------------------

int main(int argc, char** argv)
{
    // 커맨드라인 체크
    if(argc != 4 && argc != 5)
    {
        std::cerr <<
            "Usage: http-client-async <host> <port> <target> [<HTTP version: 1.0 or 1.1(default)>]\n" <<
            "Example:\n" <<
            "    http-client-async www.example.com 80 /\n" <<
            "    http-client-async www.example.com 80 / 1.0\n";
        return EXIT_FAILURE;
    }
    auto const host = argv[1];
    auto const port = argv[2];
    auto const target = argv[3];
    int version = argc == 5 && !std::strcmp("1.0", argv[4]) ? 10 : 11;

    // io_context
    net::io_context ioc;

    // 세션 객체에 대한 asynchronous 동작 실행
    std::make_shared<session>(ioc)->run(host, port, target, version);

    // Run the I/O service. The call will return when
    // the get operation is complete.
    ioc.run();

    return EXIT_SUCCESS;
}
```



### 3.3. WebSocket, Server asynchronous

- [websocket_server_async](https://www.boost.org/doc/libs/1_79_0/libs/beast/example/websocket/server/async/websocket_server_async.cpp) 참고

### 3.4. WebSocket, Client asynchronous

- [websocket_client_async](https://www.boost.org/doc/libs/1_79_0/libs/beast/example/websocket/client/async/websocket_client_async.cpp) 참고

## 참고
[boost.org beast](https://www.boost.org/doc/libs/1_79_0/libs/beast/doc/html/index.html)  
[popcorntree](https://popcorntree.tistory.com/154)  
[elecs](https://elecs.tistory.com/314)  
[developstudy](https://developstudy.tistory.com/63)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}