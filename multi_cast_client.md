* [目的](#目的)
* [代码](#代码)
* [编译](#编译)
* [注意点](#注意点)

## 目的
简单写一个程序来看看能否加入多播组，并且是否能够接收数据

## 代码
```
#include <unistd.h>
#include <stdio.h>
#include <netinet/in.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <errno.h>
#include <string.h>

#define BUFFSIZE 4096

int main(int argc, char** argv) {
  if (argc != 4) {
    printf("Usage: mc $server_addr $server_port $local_interface_ip\n");
    return -1;
  }

  int sock;
  int port = atoi(argv[2]);
  if (port > 1024 && port < 65535) {
    printf("The Port args is : %d\n", port);
  } else {
    printf("port args: [%s] may be invalid, Quit\n", argv[2]);
    return -1;
  }
  if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
    perror("Create Socket Error");
    return -1;
  }
  printf("Try to add multicast group, addr: %s, port: %s, using interface: %s\n", argv[1], argv[2], argv[3]);
  //argv[1]: server_addr, argv[2]: server_port
  struct sockaddr_in multi_cast_addr;
  memset(&multi_cast_addr, 0, sizeof(struct sockaddr_in));
  multi_cast_addr.sin_family = AF_INET;
  multi_cast_addr.sin_addr.s_addr = htonl(INADDR_ANY);
  multi_cast_addr.sin_port = htons(port);
  if (bind(sock, (struct sockaddr*)&multi_cast_addr, sizeof(struct sockaddr_in)) < 0) {
    perror("Bind Address Error");
    return -1;
  }
  struct ip_mreq multi_cast_request;
  memset(&multi_cast_request, 0, sizeof(struct ip_mreq));
  multi_cast_request.imr_multiaddr.s_addr = inet_addr(argv[1]);
  multi_cast_request.imr_interface.s_addr = inet_addr(argv[3]);
  if (setsockopt(sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, (void*)&multi_cast_request, sizeof(struct ip_mreq)) < 0) {
    perror("Add Multicast Group Error");
    return -1;
  }
  printf("Add Multicast Group Success\n");

  //try to receive data
  char buf[BUFFSIZE];
  int receive_len = 0;
  for (int i = 0; i < 10; ++i) {
    memset(buf, 0, BUFFSIZE);
    struct sockaddr_in from_addr;
    socklen_t len = sizeof(struct sockaddr_in);
    memset(&from_addr, 0, sizeof(struct sockaddr_in));
    if ((receive_len = recvfrom(sock, buf, BUFFSIZE, 0, (struct sockaddr*) &from_addr, &len)) < 0) {
      perror("Recv Error");
      continue;
    }
    printf("receive msg len is: %d, msg is: %s.\n", receive_len, buf);
    sleep(1);
  }
  if (setsockopt(sock, IPPROTO_IP, IP_DROP_MEMBERSHIP, (void*)&multi_cast_request, sizeof(struct ip_mreq)) < 0) {
    perror("Quit Multicast Group Error");
    return -1;
  }
  printf("Quit Multicast Group Success\n");
  close(sock);
  return 0;
}

```

## 编译
```
gcc multi_cast_client.c -std=gnu11
```

## 注意点

1. 编译的时候需要加上**-sd=gnu11**
2. 需要使用recvfrom函数






