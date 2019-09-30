# More Examples

## Hello World

```
import stdio as io;

func void main()
{
    io::printf("Hello world!\n");
}
```

## Fibonacci recursive

```
func long fib(long n)
{
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```


## HTTP Server

```
import http_server local;
import net local;

func void httpHandler(HttpContext* context) throws HTTPError
{
    context.response.contentType = "text/plain";
    context.response.printf("Hello world!\n");
}

func void main()
{
    HttpServer server;
    server.init();
    InetAddress addr = try server.bindPort(8080);
    printf("Begin listening to on http://%s\n", addr.description());
    server.listen(&httpHandler);
       
    catch (HTTPError e)
    {
        printf("Failed to open server.\n");
        exit(-1);
    }
}
```