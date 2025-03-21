# **Milestone 1: Single-Threaded Web Server**  

This section covers the implementation of a **basic single-threaded web server** in Rust. The server is structured around two core functions:  

1. **`main()`** – Initializes a TCP listener and waits for incoming connections.  
2. **`handle_connection()`** – Processes each connection by reading the HTTP request.  

---

## **Setting Up the TCP Listener**  

The following code initializes a **TCP listener** on `127.0.0.1:7878`:  

```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```

This binds the server to **localhost (127.0.0.1) on port 7878**. The `unwrap()` function ensures that the server starts correctly, and if binding fails (e.g., if the port is already in use), it will terminate with an error.  

---

## **Handling Incoming Connections**  

Once the listener is set up, the server enters a loop to accept and process incoming requests:  

```rust
for stream in listener.incoming() { 
    let stream = stream.unwrap();
    handle_connection(stream);
}
```

- `listener.incoming()` produces an iterator of incoming connections.  
- `stream.unwrap()` extracts the valid connection or panics if an error occurs.  
- `handle_connection(stream)` is then called to process each request.  

Since this implementation is **single-threaded**, it handles only **one connection at a time**, meaning subsequent requests must wait for the current one to finish.  

---

## **Processing HTTP Requests**  

The **`handle_connection()`** function reads and logs the incoming HTTP request:  

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

This function operates as follows:  

1. **Creates a buffered reader** (`BufReader::new(&mut stream)`) for efficient line-by-line reading.  
2. **Reads the HTTP request**:  
   - `.lines()` returns an iterator over each line of the request.  
   - `.map(|result| result.unwrap())` extracts the string from each `Result`, panicking on errors.  
   - `.take_while(|line| !line.is_empty())` stops reading once it encounters an empty line, marking the end of the HTTP headers.  
   - `.collect()` gathers all header lines into a `Vec<String>`.  
3. **Logs the request** using `println!`, displaying the collected headers in a structured format.  

---

## **Limitations of This Implementation**  

At this stage, the web server:  

- **Only logs incoming requests** and does not send responses.  
- **Processes connections sequentially**, leading to potential delays under heavy load.  
- **Lacks robust error handling**, as it uses `unwrap()` which crashes on errors.  
- **Does not parse request paths or methods**, meaning all incoming requests are treated the same.  

This serves as a foundational step before implementing **multi-threading, request handling, and response generation**. 


# **Milestone 2: Serving HTML Responses**  

![Commit 2 screen capture](/assets/images/commit2.png)  

In this milestone, the `handle_connection()` function has been updated to generate a **proper HTTP response** that includes a **status line, headers, and HTML content**. The server constructs and sends the response with the following components:  

1. A status line indicating success → `(HTTP/1.1 200 OK)`  
2. A `Content-Length` header specifying the response body's size  
3. Two **CRLF** sequences (`\r\n\r\n`) to separate headers from the body  
4. HTML content loaded from a file  

The server reads the HTML content from a file called `"hello.html"` using `fs::read_to_string()` and sends it to the client using `stream.write_all()`.  

At this stage, the server **always** returns a `200 OK` response regardless of the request, **only serves a single HTML file**, and continues to **handle connections sequentially** in a **single-threaded** manner. 🚀

