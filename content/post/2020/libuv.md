---
title: "First impression with libuv"
date: 2020-01-01
tags: ["asynchronous", "libuv"]
---

Happy New Year! May it be **year of the Linux desktop**.

Recently I tried to port my small [DNS relay and dispatcher](https://github.com/h3fang/cdns) to [libuv](https://libuv.org/) - the core of [node.js](https://nodejs.org/).

<!--more-->

After a while I did make the program to run, but with memory leaks. I tried to find solutions from the documentation. The documentation is poorly written, I can hardly learn something from there. After some searching and researching, I did manage to fix it. Here are what I have learned.

To fix memory leaks with libuv, like the other well written libraries, all you have to do is remember to **free whatever is allocated by you**. This could be simple, just try to match every `malloc` with a corresponding `free`, every `new` with corresponding `delete`.

But libuv's API design can be misleading. Like the following example. `nread` indicates how many bytes are there to be read. It could be zero. I thought that if it is zero, `buf->base` should be `NULL`, then there is nothing to free. <http://docs.libuv.org/en/v1.x/udp.html#c.uv_udp_recv_cb>

```c++
void on_relay_recv(uv_udp_t *req, ssize_t nread, const uv_buf_t *buf, const struct sockaddr *addr, unsigned flags) {
    if (nread == 0) {
        return;
    }
    else if (nread < 12) {
        fprintf(stderr, "on_relay_recv, invalid datagram size: %d\n", nread);
        free(buf->base);
        return;
    }

    ...
    free(buf->base);
}
```

But I am wrong. Even if `nread` is zero, there is nothing to read, `buf->base` may not be `NULL`. You still have to free it.

```c++
void on_relay_recv(uv_udp_t *req, ssize_t nread, const uv_buf_t *buf, const struct sockaddr *addr, unsigned flags) {
    if (nread == 0) {
        if (buf->base) { free(buf->base); }
        return;
    }
    else if (nread < 12) {
        fprintf(stderr, "on_relay_recv, invalid datagram size: %d\n", nread);
        free(buf->base);
        return;
    }

    ...
    free(buf->base);
}
```

The other thing I learned is that you have to make every thing live till related functions are called.

For example, you have to pass a `const uv_buf_t *` to `uv_udp_send`, but you don't have to always allocate it on heap, because for most of the time, you will call it immediately.

```c++
...
uv_udp_send_t *send_req = (uv_udp_send_t *) malloc(sizeof(uv_udp_send_t));
uv_buf_t buf_send = uv_buf_init(some_pointer, some_size);
uv_udp_send(send_req, &query_v4, &buf_send, 1, (struct sockaddr *)&server_v4, on_query_send);
...
```

But you do have to allocate `send_req` on heap, because it have to remain valid until `on_query_send` callback is called. And you can safely free it there.

```c++
void on_query_send(uv_udp_send_t *req, int status) {
    free(req);
    ...
}
```

libuv is a great library, but it lacks some good documentation. Hopefully this will be improved in year 2020!

## References
- <https://libuv.org/>
