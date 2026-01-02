# Own documentation explained in own terms

Official Documentation [here](https://doc.rust-lang.org/book/ch21-01-single-threaded.html)

## TcpListener

_Transmission Control Protocol (TCP)_

Desribes the detail of how information gets from one server to another (does not specift what the info is). HTTP builds on top of TCP by defining the contents of the requests and responses.

> Import: `use std::net::TcpListener`

The `bind` function in this scenario works like the new function in that it will return a new TcpListener instance. The function is called bind because, in networking, connecting to a port to listen to is known as **“binding to a port.”**

The incoming `method` on TcpListener returns an iterator that gives us a sequence of streams (more specifically, streams of type TcpStream). `for` loop will process each connection in turn and produce a series of streams for us to handle.

> Sometimes you’ll see multiple messages printed for one browser request; the reason might be that the browser is making a request for the page as well as a request for other resources, like the favicon.ico icon that appears in the browser tab.

`handle_connection` function, we create a new `BufReader` instance that wraps a reference to the stream. The BufReader adds buffering by managing calls to the `std::io::Read` trait methods for us.

`BufReader` implements the `std::io::BufRead` trait, which provides the lines method. The lines method returns an iterator of `Result<String, std::io::Error>` by splitting the stream of data whenever it sees a newline byte. To get each String, we map and unwrap each Result.

> The Result might be an error if the data isn’t valid UTF-8 or if there was a problem reading from the stream. Again, a production program should handle these errors more gracefully, but we’re choosing to stop the program in the error case for simplicity.

## unwrap

A method for handling `Result` and `Option` values.

> Ex. `let listener = TcpListener::bind("127.0.0.1:7878").unwrap();` > `bind` returns a `Result<TcpListener, std::io::Error>` > `unwrap()`: returns the TcpListener if binding succeeds, if it fails it will panic and exit (e.g. port is already in use.)

the method is equivalent to this:

```rust
let listener = match TcpListener::bind("127.0.0.1:7878") {
    Ok(l) => l,
    Err(e) => panic!("Failed to bind: {}", e),
};
```

## Workers

Worker is one OS thread (smallest unit of execution that a program can run independently.) that waits for _work_ and executes the job.

## Arc (Atomic Reference Counted)

Allows multiple workers to own the same Mutex (Ensures only one worker can access the receiver at a time)

```rust
let (sender, receiver) = mpsc::channel();
let receiver = Arc::new(Mutex::new(receiver));
```

# Functions

## fn handle_connetion

Handle a single incoming TCP request and return an HTML response.

```rust
let buf_reader = BufReader::new(&stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();
```

> Wraps the raw byte stream in a `BufReader` to enable reading by text lines.

```rust
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "index.html"),
    // ... sleep case
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

> Uses a rust match statement on the request string slice.

```rust
"GET /sleep HTTP/1.1" => {
    thread::sleep(Duration::from_secs(5));
    ("HTTP/1.1 200 OK", "index.html")
}
```

> Simulates a slow task

```rust
stream.write_all(response.as_bytes()).unwrap();
```

Convert the formatted string into bytes

# ThreadPool

Creates the channel and the shared receiver

```rust
pub fn new(size: usize) -> ThreadPool {
    // ... channel creation ...
    for id in 0..size {
        workers.push(Worker::new(id, Arc::clone(&receiver)));
    }
    // ...
}
```

> It calls Arc::clone(&receiver) for each worker. This increases the reference count so every worker has a handle to the exact same input queue.

## Graceful shutdown

```rust
fn drop(&mut self) {
    drop(self.sender.take());

    for worker in &mut self.workers {
        if let Some(thread) = worker.thread.take() {
            thread.join().unwrap();
        }
    }
}
```

- Step 1: Close the Door. drop(self.sender.take()) destroys the sender.

- Step 2: The Signal. When the sender is dropped, the recv() in the workers returns an Err (indicating the channel is closed).

- Step 3: The Break. The workers catch this Err, print "shutting down," and break their loop.

- Step 4: The Cleanup. thread.join() ensures the main thread waits for every worker to finish their current task and exit before the program stops.
