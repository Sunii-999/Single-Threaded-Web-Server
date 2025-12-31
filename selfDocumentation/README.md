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
