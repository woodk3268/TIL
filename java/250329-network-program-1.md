# 네트워크 - 프로그램1

## 1. 예제1
**클라이언트**
```
Socket socket - new Socket("localhost", PORT);
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());

String toSend = "Hello";
output.writeUTF(toSend);

String received = input.readUTF();

input.close();
output.close()
socket.close();
```

**서버**
```
ServerSocket serverSocket = new ServerSocket(PORT);

Socket socket = serverSocket.accept();
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());

String received = input.readUTF();

String toSend = reveived + " World!";
output.writeUTF(toSend);

input.close();
output.close();
socket.close();
serverSocket.close();
```

### 1-1. DNS 탐색
```
InetAddress localhost = InetAddress.getByName("localhost");
```
- TCP/IP 통신은 실제 네트워크 계층에서 작동하는 IP 주소를 기반으로 동작한다.
- 따라서, 통신할 대상 서버를 찾을 때 호스트 이름이 아니라, IP 주소가 필요하다.
- 자바는 InetAddress 클래스를 사용하면 호스트 이름으로 대상 IP를 찾을 수 있다.
  - 자바는 InetAddress.getByName("호스트명") 메서드를 사용해서 해당하는 IP 주소를 조회한다.
  - 이 과정에서 시스템의 호스트 파일을 먼저 확인한다.
  - 호스트 파일에 정의되어 있지 않다면, DNS 서버에 요청해서 IP 주소를 얻는다.

### 1-2. 클라이언트
**클라이언트와 서버의 연결은 Socket을 사용한다.**
```
Socket socket = new Socket("localhost", 12345)
```
- localhost를 통해 자신의 컴퓨터에 있는 12345 포트에 TCP 접속을 시도한다.
  - localhost는 IP가 아니므로 해당하는 IP를 먼저 찾는다. 내부에서 InetAddress를 사용한다.
  - localhost는 127.0.0.1이라는 IP에 매핑되어 있다.
  - 127.0.0.1:12345에 TCP 접속을 시도한다.
- 연결이 성공적으로 완료되면 Socket 객체를 반환한다.
- Socket은 서버와 연결되어 있는 연결점이라고 생각하면 된다.
- Socket 객체를 통해서 서버와 통신할 수 있다.

**클라이언트와 서버간의 데이터 통신은 Socket이 제공하는 스트림을 사용한다.**
```
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());
```

- Socket은 서버와 데이터를 주고받기 위한 스트림을 제공한다.
- InputStream : 서버에서 전달한 데이터를 클라이언트가 받을 때 사용한다.
- OutputStream : 클라이언트에서 서버에 데이터를 전달할 때 사용한다.
- InputStream, OutputStream을 그대로 사용하면 모든 데이터를 byte로 변환해서 전달해야 한다.
- 따라서, DataInputStream, DataOutputStream이라는 보조 스트림을 사용하면, 자바 타입의 메시지를 편리하게 주고 받을 수 있다.

```
String toSend = "Hello";
output.writeUTF(toSend);
```
- OutputStream을 통해 서버에 "Hello" 메시지를 전송한다.

```
String received = input.readUTF();
```
- InputStream을 통해 서버가 전달한 메시지를 받을 수 있다.

**사용한 자원은 반드시 정리해야 한다.**

### 1-3. 서버
**서버 소켓**
서버는 특정 포트를 열어두어야 한다. 그래야 클라이언트가 해당 포트를 지정해서 접속할 수 있다.
```
ServerSocket serverSocket = new ServerSocket(12345);
```
- 서버는 서버 소켓(ServerSocket)이라는 특별한 소켓을 사용한다.
- 지정한 포트를 사용해서 서버 소켓을 생성하면, 클라이언트는 해당 포트로 서버에 연결할 수 있다.
  - 서버가 12345 포트로 서버 소켓을 열어둔다. 클라이언트는 이제 12345 포트로 서버에 접속할 수 있다.
  - 클라이언트가 12345 포트에 연결을 시도한다.
  - 이때 OS 계층에서 TCP 3 way handshake가 발생하고, TCP 연결이 완료된다.
  - TCP 연결이 완료되면 서버는 OS backlog queue 라는 곳에 클라이언트와 서버의 TCP 연결 정보를 보관한다.
 
**클라이언트와 랜덤 포트**
TCP 연결시에는 클라이언트 서버 모두 IP, 포트 정보가 필요하다.
- 서버의 경우 포트가 명확하게 지정되어 있어야 한다.
- 그래야 클라이언트에서 서버에 어떤 포트에 접속할지 알 수 있다.
- 반면에 서버에 접속하는 클라이언트는 보통 포트를 생략하는데, 생략 시 클라이언트 PC에 남아있는 포트 중 하나가 랜덤으로 할당된다.

**accept()**
```
Socket socket = serverSocket.accept();
```
- 서버 소켓은 단지 클라이언트와 서버의 TCP 연결만 지원하는 특별한 소켓이다.
- 실제 클라이언트와 서버가 정보를 주고 받으려면 Socket 객체가 필요하다.
- serverSocket.accept() 메서드를 호출하면 TCP 연결 정보를 기반으로, Socket 객체를 만들어서 반환한다.
  - accept()를 호출하면 backlog queue에서 TCP 연결 정보를 조회한다.
    - 만약 TCP 연결 정보가 없다면, 연결 정보가 생성될 때까지 대기한다.(블로킹)
  - 해당 정보를 기반으로 Socket 객체를 생성한다.
  - 사용한 TCP 연결 정보는 backlog queue에서 제거된다.
- 클라이언트와 서버의 Socket은 TCP로 연결되어 있고, 스트림을 통해 메시지를 주고 받을 수 있다.

```
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());
```
- Socket은 클라이언트와 서버가 데이터를 주고받기 위한 스트림을 제공한다.
- InputStream : 서버 입장에서 보면 클라이언트가 전달한 데이터를 서버가 받을 때 사용한다
- OutputStream : 서버에서 클라이언트에 데이터를 전달할 때 사용한다.
- 클라이언트의 Output은 서버의 Input이고 반대로 서버의 Output은 클라이언트의 Input이다.

## 2. 예제2
**클라이언트**
```
Socket socket - new Socket("localhost", 12345);
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());

Scanner scanner = new Scanner(System.in);
while(true) {
  String toSend = scanner.nextLine();
  output.writeUTF(toSend);

  if(toSend.equals("exit")) {
    break;
  }
  String received = input.readUTF();
}

input.close();
output.close()
socket.close();
```

**서버**
```
ServerSocket serverSocket = new ServerSocket(12345);

Socket socket = serverSocket.accept(); // 블로킹
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());

while(true) {
  String received = input.readUTF();

  if(received.equals("exit")) {
    break;
  }
  
  String toSend = reveived + " World!";
  output.writeUTF(toSend);
}

input.close();
output.close();
socket.close();
serverSocket.close();
```
