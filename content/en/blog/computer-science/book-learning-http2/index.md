---
title: '[Book] Learning HTTP/2'
date: 2023-06-11T22:00:00+09:00
weight: 17
categories:
  - Book
  - Network
image: preview.png
---
## tl;dr

★★★★☆ - Highly recommended! But, a bit replaceable with other books or articles.

This book provides valuable practical handson examples and clear explanations of the progress of HTTP protocols.
It also offers a clear comparison between HTTP 2 and HTTP 1.

The link about this book is [_Learning HTTP/2_](https://www.oreilly.com/library/view/learning-http2/9781491962435/).

## Impressions

- I appreciated the book's inclusion the history of HTTP protocols.
  - What I value most in technical books like this is actually the clusion of historical context.
    While I can learn about the specifications and features of technologies through
    videos and articles in the internet, 
    understanding the history of a technology is something I can only achieve through books.
  - It included the progress of HTTP protocol from versions 0.9, 1.0 and 1.1 to 2.0
- I found the detailed explanations about HTTP/2 headers and accompanying examples to be highly valuable.
- The book was super easy to read. Despite of my slower reading pace, I managed to finish it within a few days.
- I highly recommend this book to students who are students looking for a job or engineers who may be unaware or unconsciously using HTTP 2.
  - I, too, didn't know much about HTTP2 but now I've gained some confidence in the subject.
- I intend to revisit this book serveral more times later!
  - Through this book, I was able to become somewhat familiar with HTTP 2, but honestly speaking, I still don't fully understand it.

## Summaries

### The history of HTTP protocols

1. The birth of HTTP 0.9
    - A simple protocol with limited features
    - Only supports the GET method
    - No headers are included.
    - Desipite its poor features, HTTP 0.9 was widely used. 
2. The birth of HTTP 1.0
    - Developed a few years after HTTP 0.9.
    - Introduced significant enhancements compared to HTTP0.9, including headers, response codes, redirects, errors, conditional requests, contents encoding and compressions and various methods
    - Cannot keep connections.
    - The `Host` headers is optional, not required.
    - Limited caching capabilities.
3. The birth of HTTP 1.1
    - Dominated web communications for over 20 years.
    - Allows to keep connections using the `Connection` directive.
    - Introduction of the `Host` header enables virtual hosting, serving multiple web services with a single IP address.
    - The `Upgrade` header allows negotiation for higher-level protocols.
    - Improved caching features
4. The birth of HTTP 2.0
    - Multiplexing - Enables the use of a single TCP connection for multiple requests to the same destination with the same domain name and certificate.
    - Framing - The unit of data transmitted. Data transmission occurs in framed units.
    - Header compression - Optimizes transmitting similar headers by compressing them using HPACK.

### Transition from HTTP 1 to HTTP 2

When transitioning our services from HTTP 1 to HTTP 2, we need to consider certain cases
where performance optimization tips for HTTP 1 may deteriorate the performances in HTTP 2.

- A pattern integrating files for fewer requests - In HTTP 2, there's no need to integrate files 
  because we can request each files concurrently even with just 1 TCP connection.
- A pattern using domain sharding for increased concurrent connections - In HTTP 2, typically,
  a single TCP connection is sufficient, making the pattern of domain sharding less necessary.
- A pattern using specific domains without HTTP cookies - In HTTP 2 those specific domains aren't as efficient because HPACK
  effectively compresses HTTP Headers well.
- Spriting which means integrating multiple images into a huge image and then cut it in-place in the memory - In HTTP 2,
  multiple contents can be requested concurrently, reducing the need for spriting.

### Headers which are used in HTTP 2

The content related to organizing headers in HTTP 2 is so extensive that I decided to skip this part in this article.

### Miscellaneous

Technically, TLS is not required but recommend to use TLS.
