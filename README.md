
[Source](http://docs.guzzlephp.org/en/stable/quickstart.html "Permalink to Quickstart — Guzzle Documentation")

# Quickstart — Guzzle Documentation

Trang này giới thiệu nhanh về Guzzle và các ví dụ giới thiệu. Nếu bạn chưa cài đặt, Guzzle, đi tới trang [cài đặt][1].

## Tạo một Request

Bạn có thể gửi request với Guzzle bằng cách sử dụng đối tượng `GuzzleHttpClientInterface`.

### Tạo một Client
    
    
    use GuzzleHttpClient;
    
    $client = new Client([
        // Base URI is used with relative requests
        'base_uri' => 'http://httpbin.org',
        // You can set any number of default request options.
        'timeout'  => 2.0,
    ]);
    

Client là bất biến trong Guzzle 6, có nghĩa là bạn không thể thay đổi các mặc định được sử dụng bởi một client sau khi nó được tạo ra.

Hàm khởi tạo client chấp nhận mảng liên kết các tùy chọn

`base_uri`
: 

(string|UriInterface)URI gốc của client được nối với các URI tương đối. Có thể là một chuỗi hoặc thực thể của UriInterface. Khi một URI tương đối được cung cấp cho một client, client sẽ kết hợp URI gốc với URI tương đối theo các quy tắc được mô tả trong [RFC 3986, section 2][2].
    
    
    // Create a client with a base URI
    $client = new GuzzleHttpClient(['base_uri' => 'https://foo.com/api/']);
    // Send a request to https://foo.com/api/test
    $response = $client->request('GET', 'test');
    // Send a request to https://foo.com/root
    $response = $client->request('GET', '/root');
    

Bạn cảm thấy không thích đọc RFC 3986?? Dưới đây là một số ví dụ nhanh về cách `base_uri` được xử lý với một URI khác.

| base_uri              | URI              | Result                   |  
| --------------------- | ---------------- | ------------------------ |  
| `http://foo.com`      | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `bar`            | `http://foo.com/bar`     |  
| `http://foo.com/foo/` | `bar`            | `http://foo.com/foo/bar` |  
| `http://foo.com`      | `http://baz.com` | `http://baz.com`         |  
| `http://foo.com/?bar` | `bar`            | `http://foo.com/bar`     |  

`handler`
: (callable) Hảm chuyển các request HTTP trên đường dẫn. Hàm được gọi với một `Psr7HttpMessageRequestInterface` và mảng các tùy chọn chuyển, và trả về một `GuzzleHttpPromisePromiseInterface` thỏa mãn một `Psr7HttpMessageResponseInterface` khi thành công. `handler` là tùy chọn duy nhất của hàm tạo mà không thể ghi đè trong các tùy chọn per/request.

`...`
: (mixed) Tất cả các tùy chọn khác được truyền cho hàm tạo được sử dụng như các tùy chọn reuqest  mặc định với mọi request được tạo bởi client.

### Gửi các Request

Các Magic method trên client giúp việc gửi các request đồng bộ trở nên dễ dàng:
    
    
    $response = $client->get('http://httpbin.org/get');
    $response = $client->delete('http://httpbin.org/delete');
    $response = $client->head('http://httpbin.org/get');
    $response = $client->options('http://httpbin.org/get');
    $response = $client->patch('http://httpbin.org/patch');
    $response = $client->post('http://httpbin.org/post');
    $response = $client->put('http://httpbin.org/put');
    

Bạn có thể tạo 1 request và sau đó dùng client để gửi request khi bạn sẵn sàng:
    
    
    use GuzzleHttpPsr7Request;
    
    $request = new Request('PUT', 'http://httpbin.org/put');
    $response = $client->send($request, ['timeout' => 2]);
    

Các đối tượng client cung cấp một giải pháp linh động trong cách chuyển các request bao gồm tùy chọn request mặc định,các stack middleware handler mặc định được sử dụng bởi mỗi request, và một URI gốc cho phép bạn gửi các request với các URI tương đối.

Bạn có thể tìm hiểu thêm về client middleware tại trang [_Handlers and Middleware_][3] trong tài liệu.

### Các request bất đồng bộ

Bạn có thể gửi các request bất đồng bộ bằng cách sử dụng các magic method được cung cấp bởi client:
    
    
    $promise = $client->getAsync('http://httpbin.org/get');
    $promise = $client->deleteAsync('http://httpbin.org/delete');
    $promise = $client->headAsync('http://httpbin.org/get');
    $promise = $client->optionsAsync('http://httpbin.org/get');
    $promise = $client->patchAsync('http://httpbin.org/patch');
    $promise = $client->postAsync('http://httpbin.org/post');
    $promise = $client->putAsync('http://httpbin.org/put');
    

Bạn cũng có thể sử dụng các hàm sendAsync() và requestAsync() của một client
    
    
    use GuzzleHttpPsr7Request;
    
    // Create a PSR-7 request object to send
    $headers = ['X-Foo' => 'Bar'];
    $body = 'Hello!';
    $request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);
    
    // Or, if you don't need to pass in a request instance:
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    

Các promise trả về của những hàm này implements [Promises/A+ spec][4], được cung cấp bởi [Guzzle promises library][5]. Điều này có nghĩa rằng bạn có thể gọi hàm then() nối sau promise. Các lần gọi sau đó được thực hiện với một `PsrHttpMessageResponseInterface` thành công hoặc bị từ chối với một ngoại lệ.
    
    
    use PsrHttpMessageResponseInterface;
    use GuzzleHttpExceptionRequestException;
    
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $promise->then(
        function (ResponseInterface $res) {
            echo $res->getStatusCode() . "n";
        },
        function (RequestException $e) {
            echo $e->getMessage() . "n";
            echo $e->getRequest()->getMethod();
        }
    );
    

### Các request đồng thời

Bạn có thể gửi nhiều request đồng thời sử dụng các promise và các request bất đồng bộ.
    
    
    use GuzzleHttpClient;
    use GuzzleHttpPromise;
    
    $client = new Client(['base_uri' => 'http://httpbin.org/']);
    
    // Initiate each request but do not block
    $promises = [
        'image' => $client->getAsync('/image'),
        'png'   => $client->getAsync('/image/png'),
        'jpeg'  => $client->getAsync('/image/jpeg'),
        'webp'  => $client->getAsync('/image/webp')
    ];
    
    // Wait on all of the requests to complete. Throws a ConnectException
    // if any of the requests fail
    $results = Promiseunwrap($promises);
    
    // Wait for the requests to complete, even if some of them fail
    $results = Promisesettle($promises)->wait();
    
    // You can access each result using the key provided to the unwrap
    // function.
    echo $results['image']['value']->getHeader('Content-Length')[0]
    echo $results['png']['value']->getHeader('Content-Length')[0]
    

Bạn có thể sử dụng đối tượng `GuzzleHttpPool` khi bạn không xác định được số lượng request cần gửi.
    
    
    use GuzzleHttpPool;
    use GuzzleHttpClient;
    use GuzzleHttpPsr7Request;
    
    $client = new Client();
    
    $requests = function ($total) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield new Request('GET', $uri);
        }
    };
    
    $pool = new Pool($client, $requests(100), [
        'concurrency' => 5,
        'fulfilled' => function ($response, $index) {
            // this is delivered each successful response
        },
        'rejected' => function ($reason, $index) {
            // this is delivered each failed request
        },
    ]);
    
    // Initiate the transfers and create a promise
    $promise = $pool->promise();
    
    // Force the pool of requests to complete.
    $promise->wait();
    

Hoặc sử dụng một closure sẽ trả về một promise khi pool gọi closure.
    
    
    $client = new Client();
    
    $requests = function ($total) use ($client) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield function() use ($client, $uri) {
                return $client->getAsync($uri);
            };
        }
    };
    
    $pool = new Pool($client, $requests(100));
    

## Sử dụng các Responses

Trong các ví dụ trên, chúng ta đã lấy được một biến `$response` hoặc  ta nhận được response từ một promise. Đối tưởng response sẽ implements một PSR-7 response, `PsrHttpMessageResponseInterface`, và chứa rất nhiều thông tin hữu ích.

Chúng ta có thể lấy được code trạng thái và reason phrase của response:
    
    
    $code = $response->getStatusCode(); // 200
    $reason = $response->getReasonPhrase(); // OK
    

Bạn có thể lấy header từ response:
    
    
    // Check if a header exists.
    if ($response->hasHeader('Content-Length')) {
        echo "It exists";
    }
    
    // Get a header from the response.
    echo $response->getHeader('Content-Length');
    
    // Get all of the response headers.
    foreach ($response->getHeaders() as $name => $values) {
        echo $name . ': ' . implode(', ', $values) . "rn";
    }
    

Nội dung body của một response có thể được lấy bằng cách sử dụng hàm `getBody`.Nội dung body có thể được sử dụng như một chuỗi, ép thành một chuỗi,hay sử dụng như một luồng giống đối tượng.
    
    
    $body = $response->getBody();
    // Implicitly cast the body to a string and echo it
    echo $body;
    // Explicitly cast the body to a string
    $stringBody = (string) $body;
    // Read 10 bytes from the body
    $tenBytes = $body->read(10);
    // Read the remaining contents of the body as a string
    $remainingBytes = $body->getContents();
    

## Thám số trong chuỗi query

Bạn có thể tạo ra các tham số trong chuỗi query trong một request bằng nhiều cách.

Bạn có thể đặt các tham số của chuỗi query trong URI của request:
    
    
    $response = $client->request('GET', 'http://httpbin.org?foo=bar');
    

Bạn có thể chỉ định tham số chuỗi query bằng cách sử dụng tùy chọn request `query` dưới dạng mảng.
    
    
    $client->request('GET', 'http://httpbin.org', [
        'query' => ['foo' => 'bar']
    ]);
    

Tạo tùy chọn dưới dạng mảng sẽ sử dụng hàm `http_build_query`của PHP để định dạng chuỗi query.

Và cuối cùng, bạn cũng có thể tạo được tùy chọn request `query` dưới dạng một chuỗi.
    
    
    $client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);
    

## Uploading Data

Guzzle cung cấp một số phương pháp để upload dữ liệu.

Bạn có thể gửi các request chứa luồng của dữ liệu bằng cách truyền một chuỗi, resource trả về từ hàm `fopen`, hoặc một thực thể của `PsrHttpMessageStreamInterface` cho tùy chọn request `body`.
    
    
    // Provide the body as a string.
    $r = $client->request('POST', 'http://httpbin.org/post', [
        'body' => 'raw data'
    ]);
    
    // Provide an fopen resource.
    $body = fopen('/path/to/file', 'r');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    
    // Use the stream_for() function to create a PSR-7 stream.
    $body = GuzzleHttpPsr7stream_for('hello!');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    

Một cách đơn giản để upload dữ liệu JSON và đặt header phù hợp là sử dụng tùy chọn request `json` :
    
    
    $r = $client->request('PUT', 'http://httpbin.org/put', [
        'json' => ['foo' => 'bar']
    ]);
    

### POST/Form Requests

Ngoài việc chỉ định dữ liệu thô của một request sử dụng các tùy chọn request `body` , Guzzle cũng cấp các lớp trừu tượng hữu ích trong quá trình gửi dữ liệu POST.

#### Gửi các trường trong form

Gửi các request POST `application/x-www-form-urlencoded` yêu cầu bạn chỉ định trường POST dưới dạng một mảng trong các tùy chọn request `form_params`
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'form_params' => [
            'field_name' => 'abc',
            'other_field' => '123',
            'nested_field' => [
                'nested' => 'hello'
            ]
        ]
    ]);
    

#### Gửi các file trong form

Bạn có thể gửi các file cùng với một form (`multipart/form-data` POST requests), sử dụng tùy chọn `multipart` . `multipart` chấp nhận các mảng liên kết,trong đó mỗi mảng liên kết chứa các khóa sau:

* name: (required, string) khóa ánh xạ với trường tên của form .
* contents: (required, mixed) Chứa một chuỗi để gửi nội dụng của file dưới dạng một chuỗi, chứa một fopen resource để chuyển nội dụng của một luồng PHP hoặc chứa một `PsrHttpMessageStreamInterface` để chuyển nội dung từ một luồng PSR-7.
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'multipart' => [
            [
                'name'     => 'field_name',
                'contents' => 'abc'
            ],
            [
                'name'     => 'file_name',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'other_file',
                'contents' => 'hello',
                'filename' => 'filename.txt',
                'headers'  => [
                    'X-Foo' => 'this is an extra header to include'
                ]
            ]
        ]
    ]);
    

## Cookies

Guzzle có thể duy trì một phiên cookie cho bạn nếu được yêu cầu sử dụng tùy chọn request `cookies` . Khi  gửi một request,tùy chọn `cookies` phải được gắn với một thực thể `GuzzleHttpCookieCookieJarInterface`.
    
    
    // Use a specific cookie jar
    $jar = new GuzzleHttpCookieCookieJar;
    $r = $client->request('GET', 'http://httpbin.org/cookies', [
        'cookies' => $jar
    ]);
    

Bạn có thể đặt `cookies` về `true`trong khởi tạo client nếu bạn muốn sử dụng một  cookie jar chung cho tất cả các request..
    
    
    // Use a shared client cookie jar
    $client = new GuzzleHttpClient(['cookies' => true]);
    $r = $client->request('GET', 'http://httpbin.org/cookies');
    

## Điều hướng

Guzzle sẽ tự động chuyển điều hướng trừ khi bạn bảo nó không làm vậy.Bạn có thể tùy chỉnh hành đồng điều hướng bằng cách sử dụng tùy chọn request `allow_redirects`.

* Đặt thành `true` để bật điều hướng bình thường với tối đa 5 lần điều hướng. Đây là cài đặt mặc định
* Đặt `false` để tắt chức năng điều hướng.
* Truyền một mảng liên kết chứa từ khóa 'max' để chỉ định số lần điều hướng tối đa và có t hể thêm khóa  'strict' để chỉ định điều hướng có tuân thủ RFC nghiêm ngặt hay không (có nghĩa là điều hướng các POST request với các POST so với việc mà hầu hết các trình duyệt sẽ làm đó là điều hướng các POST request với các GET request).
    
    
    $response = $client->request('GET', 'http://github.com');
    echo $response->getStatusCode();
    // 200
    

Ví dụ sau cho thấy tính năng điều hướng có thể được tắt.
    
    
    $response = $client->request('GET', 'http://github.com', [
        'allow_redirects' => false
    ]);
    echo $response->getStatusCode();
    // 301
    

## Ngoại lệ

Guzzle ném ngoại lệ cho các lỗi xảy ra trong quá trình chuyển.

* Trong trường hợp có lỗi mạng (connection timeout, DNS errors, etc.), một `GuzzleHttpExceptionRequestException` sẽ được ném ra. Ngoại lệ này kế thừa từ `GuzzleHttpExceptionTransferException`. Việc bắt ngoại lệ này sẽ bắt bất cứ ngoại lệ nào có thể được ném ra trong quá trình chuyển request.
    
        use GuzzleHttpPsr7;
    use GuzzleHttpExceptionRequestException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (RequestException $e) {
        echo Psr7str($e->getRequest());
        if ($e->hasResponse()) {
            echo Psr7str($e->getResponse());
        }
    }
    

* Một ngoại lệ  `GuzzleHttpExceptionConnectException`được ném ra trong trường hợp lỗi mạng. Ngoại lệ này kế thừa từ `GuzzleHttpExceptionRequestException`.
* Một `GuzzleHttpExceptionClientException` được ném ra mức lỗi 400 nếu tùy chọn request `http_errors` được đặt thành true. Ngoại lệ này kế thừa từ `GuzzleHttpExceptionBadResponseException` và `GuzzleHttpExceptionBadResponseException` kế thừa từ `GuzzleHttpExceptionRequestException`.
    
        use GuzzleHttpExceptionClientException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (ClientException $e) {
        echo Psr7str($e->getRequest());
        echo Psr7str($e->getResponse());
    }
    

* Một `GuzzleHttpExceptionServerException` được ném ra cho lỗi mức 500 nếu tùy chọn request `http_errors` được gán thành true. Ngoại lệ này kế thừa từ `GuzzleHttpExceptionBadResponseException`.
* Một `GuzzleHttpExceptionTooManyRedirectsException` sẽ được ném ra khi có quá nhiều lần điều hướng xảy ra. Ngoại lệ này kế thừa từ `GuzzleHttpExceptionRequestException`.

Tất cả những ngoại lệ trên đều kế thừa từ`GuzzleHttpExceptionTransferException`.

## Biến môi trường

Guzzle cung cấp một vài biến môi trường để có thể sử dụng để tuỳ chỉnh hành động của thư viện.

`GUZZLE_CURL_SELECT_TIMEOUT`
: điều chỉnh khoảng thời gian tính bằng giây mà xử lý một curl_multi_*sẽ sử dụng khi chọn xử lý curl bằng `curl_multi_select()`. Một vài hệ thống có vấn đề với các triển khai của PHP của  `curl_multi_select()` khi mà việc gọi hàm này luôn dẫn tới việc phải chờ trong khoảng thời gian timeout tối đa.


`HTTP_PROXY`
: 

Định nghĩa proxy được sử dụng khi gửi các request sử dụng giao thức "http".

Note: ví biến HTTP_PROXY variable có thể chứa đầu vào tùy ý từ người dùng trên một vài môi trường (CGI) , nên nó chỉ được sử dụng trên CLI SAPI. Xem thêm thông tin phía dưới.

`HTTPS_PROXY`
:Định nghĩa proxy được sử dụng khi gửi các request mà sử dụng giao thức "https".

[1]: http://docs.guzzlephp.org/overview.html#installation
[2]: http://tools.ietf.org/html/rfc3986#section-5.2
[3]: http://docs.guzzlephp.org/handlers-and-middleware.html
[4]: https://promisesaplus.com/
[5]: https://github.com/guzzle/promises

  
