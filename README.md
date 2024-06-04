Shortlist of actions taken on Orange Pi 3b:
Installation of Orange Pi 1.0.6 OS.
Configuration of vsftpd and creation ftp folder.
Allowed ports 20,21,22, 4444
Created a low-privileged user inside the system and configured sshd for the user.
Encrypted credentials for this user and stored them inside ftp banner.
Created a C vulnerable server running on port 4444
Add this server to systemd as a service so it worked after rebooting the system.

Here's a misconfigurations added to our Orange Pi 3b:
1.  Way In: Exploit a misconfiguration in FTP to allow anonymous access.
steps done for this:
sudo ufw allow 20
sudo ufw allow 21
mkdir /var/ftp
chown nobody:nogroup /var/ftp
inside vsftpd.conf I made leaked credentials and allowed anonymous access
sudo systemctl restart vsftpd
2. Encoded and ciphered ssh creds as the initial entry point.
sudo useradd -m hacker
sudo echo hacker:’letmein’ && sudo chpassword
In sshd_config allowed PasswordAuthentication
sudo systemctl restart sshd
3. Root flag cannot be read by low-privileged user.
4. SUID binary for privilege escalation.
#include<unistd.h>
void main()
{    setuid(0);
     setgid(0);
     system("cat /flag.txt");
     system("/bin/bash");
}
gcc -o break.c break

5. Vulnerable to buffer overflow C program at port 4444.
The code of the vulnerable program:
#include#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
void handle_client(int client_socket) {
    char buffer[1024];

    char welcome_message[] = "Nice, have you ever heard about binary exploitation?\n";
    send(client_socket, welcome_message, sizeof(welcome_message), 0);
    int bytes_received = recv(client_socket, buffer, 2048, 0); // Vulnerability: buffer overflow
    if (bytes_received < 0) {
        perror("Error receiving data");
        close(client_socket);
        return;
    }
 
    printf("Accepted: %s\n", buffer);

    system(buffer);
 
    close(client_socket);
}
 
int main(int argc, char *argv[]) {
    int server_socket, client_socket;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int port = 4444; // Port
 
    if (argc == 2) {
        port = atoi(argv[1]);
    }
 
    server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket < 0) {
        perror("Socket creation error");
        exit(EXIT_FAILURE);
    }
 
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(port);
 
    if (bind(server_socket, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Error");
        close(server_socket);
        exit(EXIT_FAILURE);
    }
 
    if (listen(server_socket, 5) < 0) {
        perror("Listenning error");
        close(server_socket);
        exit(EXIT_FAILURE);
    }
    printf("Server listens port %d\n", port);
    while (1) {
        client_socket = accept(server_socket, (struct sockaddr *)&client_addr, &client_addr_len);
        if (client_socket < 0) {
            perror("Error accepting connection");
            continue;
        }
  printf("Connecion from client%s:%d\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));
        handle_client(client_socket);
    }
    close(server_socket);
    return 0;
}
}
# OrangePi3b
OrangePi3bserver
