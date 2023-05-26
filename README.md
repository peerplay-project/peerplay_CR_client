# Peerplay CR Client

Fork of switch-lan-play by Spacemeowx2 for improving it

```
Console <-----------> Peerplay <------------> Peerplay <===========> Peerplay <------------> Peerplay <----------> Console
            PCAP      CRClient     V-LAN      CRServer     V-LAN     CRServer      V-LAN     CRClient     PCAP
          ARP,IPv4             ^ UDP (LAN) ^               UDP          ^     ^ UDP (LAN) ^             ARP,IPv4
                              (Client/Server)         (P2P Network)     |    (Client/Server)
                                                                        |  
                                                                        |
                                                                        \ -----------------> Peerplay <----------> Console
                                                                           UDP (Internet)    CRClient     PCAP
                                                                                                        ARP,IPv4
```

# Usage

This Project is a part of the Peerplay project, to install the program and play we recommend using the main repo accessible here [peerplay-project/peerplay: Main Application of peerplay-project](https://github.com/peerplay-project/peerplay)

# Build

## Debug or Release

`cmake -DCMAKE_BUILD_TYPE=Debug ..`
`cmake -DCMAKE_BUILD_TYPE=Release ..`

## Ubuntu / Debian

This project depends on libpcap, you can install libpcap0.8-dev on Ubuntu or Debian:

`sudo apt install libpcap0.8-dev git gcc g++ cmake`

Prepare a cmake, gcc, and run like this:

```sh
mkdir build
cd build
cmake ..
make
```

## Windows

Use [MSYS2](http://www.msys2.org/) to compile.

```sh
pacman -Sy
pacman -S make \
    mingw-w64-x86_64-cmake \
    mingw-w64-x86_64-gcc
```

To compile 32bit program:

```sh
pacman -S mingw-w64-i686-cmake \
    mingw-w64-i686-gcc
```

Open `MSYS2 MinGW 64-bit` or `MSYS2 MinGW 32-bit`.

```sh
mkdir build
cd build
cmake -G "MSYS Makefiles" ..
make
```

## Mac OS

```sh
brew install cmake
```

```sh
mkdir build
cd build
cmake ..
make
```

# Server

## Docker

`docker run -d -p 11451:11451/udp -p 11451:11451/tcp spacemeowx2/switch-lan-play`

## Node

```sh
git clone https://github.com/spacemeowx2/switch-lan-play
cd switch-lan-play/server
npm install
npm run build # build ts to js. run it again when code changed.
npm start
```

Use `--port` pass the port parameter, or else it will use  `11451/udp` as default.

Use `--simpleAuth` pass the auth via username and password, or else there's no authentication.

Use `--httpAuth` pass the auth via http url, or else there's no authentication.

Use `--jsonAuth` pass the auth via json file, or else there's no authentication.

Example:

```sh
npm run build
npm start -- --port 10086 --simpleAuth username:password
```

Meanwhile the monitor service will be started on port `11451/tcp` by default, you can get online client count via HTTP request:

Request: `GET http://{YOUR_SERVER_IP}:11451/info`

Response: `{ "online": 42 }`

# Protocol

The protocol is very simple now, but I'm going to add some fileds to calculate network quality(packet loss, ping), like timestamp, seq_id, etc.

```c
struct packet {
    uint8_t type;
    uint8_t payload[packet_len - 1];
};
```

```c
enum type {
    KEEPALIVE = 0,
    IPV4 = 1,
    PING = 2,
    IPV4_FRAG = 3
};
```

The server can read IP addresses from payload and save source IP -> LAN IP to a cache table. If target ip address shown in payload doesn't hit the cache, broadcast this packet to the entire room(now a server is a room).
