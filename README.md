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

![Commit 2 screen capture](/assets/images/commit2.jpg)  

In this milestone, the `handle_connection()` function has been updated to generate a **proper HTTP response** that includes a **status line, headers, and HTML content**. The server constructs and sends the response with the following components:  

1. A status line indicating success → `(HTTP/1.1 200 OK)`  
2. A `Content-Length` header specifying the response body's size  
3. Two **CRLF** sequences (`\r\n\r\n`) to separate headers from the body  
4. HTML content loaded from a file  

The server reads the HTML content from a file called `"hello.html"` using `fs::read_to_string()` and sends it to the client using `stream.write_all()`.  

At this stage, the server **always** returns a `200 OK` response regardless of the request, **only serves a single HTML file**, and continues to **handle connections sequentially** in a **single-threaded** manner.


# **Milestone 3: Validating Requests and Selective Responses**  

![Commit 3 screen capture](/assets/images/commit3.jpg)  

In this milestone, the web server has been improved to **validate incoming requests** and selectively respond with the appropriate content. The server now properly handles **invalid routes** by returning a **404 error page** when necessary.  

---

## **Enhancements in `handle_connection()`**  

The function now extracts only the **first line of the HTTP request** (the request line) and uses **pattern matching** to determine the appropriate response. This allows the server to **serve different HTML files based on the requested path**.  

### **Key Improvements**
1. **Efficient request extraction:**  
   - Instead of collecting all HTTP headers, only the **request line** is extracted using:  
     ```rust
     buf_reader.lines().next()
     ```
   - This contains the **HTTP method, request path, and protocol version**.  

2. **Pattern matching for request handling:**  
   - **For root path requests (`GET / HTTP/1.1`)**, the server returns **200 OK** with `"hello.html"`.  
   - **For all other requests**, the server responds with **404 NOT FOUND** and serves `"404.html"`.  

3. **Serving different HTML files based on the request path** ensures **better user feedback**.  

---

## **Refactoring: Using `match` Instead of `if-else`**
Previously, an `if-else` approach was used:  
```rust
if request_line == "GET / HTTP/1.1" {
    status_line = "HTTP/1.1 200 OK";
    filename = "hello.html";
} else {
    status_line = "HTTP/1.1 404 NOT FOUND";
    filename = "404.html";
}
```
### **Why `match` Is a Better Approach**
1. **Cleaner and more readable** – No repeated code blocks for each condition.  
2. **Ensures all variables are properly initialized** – The Rust compiler can verify that all cases are covered.  
3. **More maintainable** – Adding new routes is easier by simply adding **new match arms**.  
4. **Encourages immutability (`let` instead of `mut`)**, which is safer for concurrency and **improves code clarity**.  

---

## **Current Server Functionality**
At this stage, the server:  
✔ **Listens on `localhost:7878`** for incoming connections.  
✔ **Parses the HTTP request line** to determine the response.  
✔ **Serves `"hello.html"` with `200 OK`** for requests to `/`.  
✔ **Serves `"404.html"` with `404 NOT FOUND`** for all other paths.  
✔ **Still operates as a single-threaded server**.  

These updates **enhance response handling and improve maintainability**, setting the stage for future optimizations such as **multi-threading** and **dynamic content generation**. 🚀  


