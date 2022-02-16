# More Examples

## Hello World

```
fn void main()
{
    io::printf("Hello world!\n");
}
```

## Fibonacci recursive

```
fn long fib(long n)
{
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```


## HTTP Server

```
import net::http_server;

fn void! httpHandler(HttpContext* context)
{
    context.response.contentType = "text/plain";
    context.response.printf("Hello world!\n");
}

fn void main()
{
    HttpServer server;
    server.init();
    InetAddress! addr = server.bindPort(8080);
    if (catch addr)
    {
        printf("Failed to open server.\n");
        exit(-1);
    }
    
    printf("Begin listening to on http://%s\n", addr.description());
    server.listen(&httpHandler);
    
}
```