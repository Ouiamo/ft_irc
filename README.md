# ft_irc: Internet Relay Chat Server

## 📝 Description

**ft_irc** is a custom IRC (Internet Relay Chat) server implementation in C++98. The project aims to deepen understanding of network protocols, socket programming, and event-driven I/O multiplexing. The server handles multiple simultaneous clients using non-blocking I/O with a single `poll()`  system call, faithfully implementing core IRC protocol features.

The server accepts connections from any standard IRC client, allowing users to authenticate, set nicknames, join channels, exchange public and private messages, and manage channel operations.

## 🛠️ Infrastructure Overview

### Core Features (Mandatory)

- **Multi-client Handling**: Non-blocking I/O with a single `poll()` loop
- **Authentication**: Password-protected connections with nickname and username setup
- **Channel System**: Create and join channels, with public message broadcasting
- **Operator Privileges**: Channel operators with special management commands
- **Core IRC Commands**:
  - `NICK` - Set/change nickname
  - `USER` - Set username
  - `JOIN` - Join a channel
  - `PRIVMSG` - Send private or channel messages
  - `QUIT` - Disconnect from server

### Operator Commands

| Command | Description |
|---------|-------------|
| `KICK` | Eject a client from the channel |
| `INVITE` | Invite a client to a channel |
| `TOPIC` | Change or view the channel topic |
| `MODE` | Modify channel modes (i, t, k, o, l) |

### Channel Modes

| Mode | Description |
|------|-------------|
| `i` | Invite-only channel |
| `t` | Topic restricted to operators |
| `k` | Channel key (password) |
| `o` | Give/take operator privilege |
| `l` | User limit for the channel |

### Bonus Features

- **File Transfer**: DCC protocol support for direct client-to-client file transfers
- **IRC Bot**: Automated bot responding to commands and interactions

## 🚀 Instructions

### 1. Compilation

Use the provided **Makefile** at the root of the directory:

```bash
make        # Compiles the server with C++98 and flags -Wall -Wextra -Werror
```

Available rules:
- `make all` - Compile the server
- `make clean` - Remove object files
- `make fclean` - Remove object files and executable
- `make re` - Recompile from scratch

### 2. Execution

Run the server with a port number and connection password:

```bash
./ircserv <port> <password>
```

**Example:**
```bash
./ircserv 6667 secretpassword
```

| Argument | Description |
|----------|-------------|
| `port` | Listening port (e.g., 6667) |
| `password` | Connection password required by clients |

### 3. Connecting with an IRC Client

Use any standard IRC client (reference client: **LimeChat**, **WeeChat**, **HexChat**, or **Irssi**):

```bash
# Example with netcat (manual IRC commands)
nc -C 127.0.0.1 6667
PASS secretpassword
NICK mynick
USER myuser 0 * :Real Name
JOIN #channel
PRIVMSG #channel :Hello everyone!
```

**Or use a graphical client:**
- Server: `127.0.0.1`
- Port: `6667` (or your chosen port)
- Password: As specified on server launch

### 4. Testing Commands

| Action | Command |
|--------|---------|
| Join channel | `JOIN #channel` |
| Send message to channel | `PRIVMSG #channel :Your message` |
| Send private message | `PRIVMSG nickname :Private message` |
| Change channel topic | `TOPIC #channel :New topic` |
| Kick user | `KICK #channel username :Reason` |
| Invite user | `INVITE username #channel` |
| Set channel password | `MODE #channel +k mypassword` |
| Set user limit | `MODE #channel +l 10` |
| Give operator status | `MODE #channel +o username` |
| Make channel invite-only | `MODE #channel +i` |

## 🧠 Technical Choices

### Non-blocking I/O with `poll()`
The server uses a single `poll()` to monitor all socket file descriptors. This event-driven approach prevents blocking operations and ensures efficient resource usage. Direct `read()`/`recv()`  calls without `poll()` are strictly forbidden and would result in project failure.

### C++98 Standard
All code complies with the C++98 standard, compiled with `-std=c++98` in addition to `-Wall -Wextra -Werror`. C++ features are preferred (e.g., `<string>` over `<string.h>`), though standard C functions are permitted when necessary.

### TCP/IP Communication
Clients connect via TCP/IP (IPv4). Each connection is handled through a dedicated socket, with the server managing all I/O operations in a non-blocking manner.

### Message Parsing
The server aggregates incoming packets to rebuild complete IRC messages before processing. This handles fragmented transmissions (e.g., receiving "com", then "man", then "d\n" via `nc`) and partial data scenarios.

### No Forking
Forking is prohibited. All client handling occurs within a single process using the `poll()` event loop.

## 📋 IRC Protocol Implementation Notes

### Authentication Flow
1. Client sends `PASS <password>`
2. Client sends `NICK <nickname>`
3. Client sends `USER <username> <mode> <unused> :<realname>`
4. Server responds with `RPL_WELCOME`, `RPL_YOURHOST`, `RPL_CREATED`, `RPL_MYINFO`
5. Client is fully registered

### Message Broadcasting
When a client sends `PRIVMSG #channel :message`, the server forwards the message to all other clients in that channel (including proper prefix formatting as per IRC protocol).

### Error Handling
The server gracefully handles:
- Partial/truncated messages
- Invalid commands
- Non-existent channels/users
- Insufficient privileges
- Connection drops

## 🔧 Management Commands

| Command | Description |
|---------|-------------|
| `make` | Compile the server |
| `make clean` | Remove object files |
| `make fclean` | Full cleanup (objects + executable) |
| `make re` | Recompile everything |
| `Ctrl+C` | Stop the server (clean shutdown) |

## 🧪 Test Example

Testing fragmented command transmission with `nc`:

```bash
$> nc -C 127.0.0.1 6667
PASS pass
NICK test
USER test 0 * :Test User
JOI       # Type "JOI"
N         # Then "N"
#channel  # Then " #channel"
```

The server must aggregate "JOI" + "N" + " #channel" to form `JOIN #channel` before processing.

## 📚 Resources
- [RFC 1459 - Internet Relay Chat Protocol](https://datatracker.ietf.org/doc/html/rfc1459)
- [RFC 2810-2813 - IRC Architecture and Protocol](https://datatracker.ietf.org/doc/html/rfc2810)
- [Socket Programming in C++](https://www.geeksforgeeks.org/socket-programming-cc/)

### IRC Clients for Testing
- [HexChat](https://hexchat.github.io/) (Cross-platform)

---

**Author:** oaoulad- | 42 Curriculum | December 2024
