# 07 · Networking Basics

Java has supported network programming since version 1.0. This module covers
the low-level building block — TCP sockets — and the modern, high-level
`HttpClient` most real applications actually use today.

## TCP sockets — a minimal echo server

A `ServerSocket` listens for incoming connections; each accepted connection is
handed off as a `Socket` you can read from and write to like a stream.

```java
// EchoServer.java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class EchoServer {
    public static void main(String[] args) throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(5000)) {
            System.out.println("Echo server listening on port 5000...");

            try (Socket client = serverSocket.accept();   // blocks until a client connects
                 BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
                 PrintWriter out = new PrintWriter(client.getOutputStream(), true)) {

                String line;
                while ((line = in.readLine()) != null) {
                    System.out.println("Received: " + line);
                    out.println("Echo: " + line);
                    if (line.equalsIgnoreCase("bye")) break;
                }
            }
        }
    }
}
```

## A matching TCP client

```java
// EchoClient.java
import java.io.*;
import java.net.Socket;

public class EchoClient {
    public static void main(String[] args) throws IOException {
        try (Socket socket = new Socket("localhost", 5000);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in))) {

            out.println("Hello server");
            System.out.println(in.readLine());   // Echo: Hello server

            out.println("bye");
            System.out.println(in.readLine());   // Echo: bye
        }
    }
}
```

Run the server first, then the client, in two separate terminals:

```bash
javac EchoServer.java && java EchoServer
# (in a second terminal)
javac EchoClient.java && java EchoClient
```

```text
# Server output:
Echo server listening on port 5000...
Received: Hello server
Received: bye

# Client output:
Echo: Hello server
Echo: bye
```

Each `Socket` gives you an `InputStream`/`OutputStream` pair — everything
above the raw bytes (line framing, protocols like HTTP) is built on this same
foundation.

## The modern HTTP client — GET request

Since Java 11, `java.net.http.HttpClient` provides a clean, built-in API for
HTTP calls — no third-party library needed for basic use.

```java
import java.net.URI;
import java.net.http.*;

public class HttpGetDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.github.com/repos/openjdk/jdk"))
                .header("Accept", "application/vnd.github+json")
                .GET()
                .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        System.out.println("Status: " + response.statusCode());
        System.out.println("Body (first 100 chars): " +
                response.body().substring(0, Math.min(100, response.body().length())));
    }
}
// Output (abridged):
// Status: 200
// Body (first 100 chars): {"id":3971870,"node_id":"...
```

## Making a POST request

```java
import java.net.URI;
import java.net.http.*;

public class HttpPostDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        String jsonBody = """
            {"title": "foo", "body": "bar", "userId": 1}
            """;

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://jsonplaceholder.typicode.com/posts"))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
                .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        System.out.println("Status: " + response.statusCode());   // Status: 201
        System.out.println("Body: " + response.body());
        // Body: {"title":"foo","body":"bar","userId":1,"id":101}
    }
}
```

## Asynchronous requests

`HttpClient` also supports non-blocking calls via `sendAsync`, returning a
`CompletableFuture<HttpResponse<T>>` (see
[Level 3, Module 2](02-util-concurrent.md) for `CompletableFuture` details):

```java
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println)
        .join();
```

| Type | Purpose |
|------|---------|
| `ServerSocket` | Listens for and accepts incoming TCP connections |
| `Socket` | One end of a TCP connection; provides input/output streams |
| `HttpClient` | Modern client for making HTTP requests (Java 11+) |
| `HttpRequest` | Describes a single HTTP request (URI, method, headers, body) |
| `HttpResponse<T>` | The response, with status code, headers, and a typed body |
| `BodyHandlers.ofString()` | Reads the response body as a `String` |

## Exercise

Write a program that uses `HttpClient` to send a GET request to
`https://jsonplaceholder.typicode.com/todos/1`, prints the HTTP status code,
and prints the raw JSON body. Then modify it to send a PUT request to the same
URL with a JSON body of your choosing, and print the response status and
body.
