# socket能否重复通信？

socket与serverSocket是可以重复通信的

在日常开发的过程中，我们在读写操作一个socket之后，习惯性的关闭 IO 和 socket，

无论是关闭socket还是只关闭 IO 流都会将socket通道关闭，导致不能重用。

* InputStream in = socket.getInputStream()
  * java.net.`SocketImpl`#getInputStream
    * java.net.`AbstractPlainSocketImpl`#socketInputStream
      * java.net.`SocketInputStream`#close
        * java.net.`Socket`#close

> 如果客户端的 socket 不关闭，服务端开的线程将不能关闭，
>
> 可以在客户端重用之后，手动断开socket，那么服务端的线程将自动关闭