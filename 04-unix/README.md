# 1 вариант (мини либа)

Напишем мини либу `blocknet.c`

```c
#define _GNU_SOURCE
#include <dlfcn.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <errno.h>

// Перекрываем socket()
int socket(int domain, int type, int protocol) {
    errno = ENETUNREACH;
    return -1;
}

// Перекрываем connect()
int connect(int sockfd, const struct sockaddr *addr, socklen_t len) {
    errno = ENETUNREACH;
    return -1;
}

// Перекрываем getaddrinfo() — чтобы даже DNS-запросы не шли
int getaddrinfo(const char *node, const char *service,
                const struct addrinfo *hints,
                struct addrinfo **res) {
    return EAI_SERVICE;
}

```

Компилируем (нужен gcc и libc-dev, ставим их, если надо):


```bash
apt update && apt install -y build-essential nano 
gcc -shared -fPIC blocknet.c -o /tmp/libblocknet.so -ldl
```

Активируем хук

```bash
export LD_PRELOAD=/tmp/libblocknet.so
```

## Пример 
```bash
root@646193f94d8e:/# apt update
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

root@646193f94d8e:/# apt install nano 
root@646193f94d8e:/# nano blocknet.c 
# вставялем код выше

root@646193f94d8e:/# apt update && apt install -y build-essential
root@646193f94d8e:/# gcc -shared -fPIC blocknet.c -o /tmp/libblocknet.so -ldl
root@646193f94d8e:/# export LD_PRELOAD=/tmp/libblocknet.so

root@646193f94d8e:/# apt update 
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Err:1 http://archive.ubuntu.com/ubuntu noble InRelease
  Could not resolve 'archive.ubuntu.com'
Err:2 http://security.ubuntu.com/ubuntu noble-security InRelease
  Could not resolve 'security.ubuntu.com'
Err:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
  Could not resolve 'archive.ubuntu.com'
Err:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
  Could not resolve 'archive.ubuntu.com'
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble/InRelease  Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-updates/InRelease  Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-backports/InRelease  Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://security.ubuntu.com/ubuntu/dists/noble-security/InRelease  Could not resolve 'security.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
```



# 2 вариант (Ломаем DNS)

Решение через 

```bash
echo "" > /etc/resolv.conf
```

## Пример 

```bash
root@14645b752588:/# apt update
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
root@14645b752588:/# echo "" > /etc/resolv.conf
root@14645b752588:/# apt update
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:1 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Err:1 http://archive.ubuntu.com/ubuntu noble InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:4 http://security.ubuntu.com/ubuntu noble-security InRelease
  Temporary failure resolving 'security.ubuntu.com'
Err:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-updates/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-backports/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://security.ubuntu.com/ubuntu/dists/noble-security/InRelease  Temporary failure resolving 'security.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
```