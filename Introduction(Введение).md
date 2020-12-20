# Что такое Socket.IO:
Socket.IO - это библиотека, которая обеспечивает двустороннюю связь в реальном времени и на основе событий между браузером и сервером.
Это состоит из:
* Node.js server: [Source](https://github.com/socketio/socket.io) | [API](https://socket.io/docs/v3/server-api/)
* И клиентской библиотеки Javascript для браузера (которую также можно запускать из Node.js): [Source](https://github.com/socketio/socket.io-client) | [API](https://socket.io/docs/v3/client-api/)
Format: ![Alt Text](https://socket.io/images/bidirectional-communication.png)

### Есть также несколько клиентских реализаций на других языках, которые поддерживаются сообществом:
* Java: https://github.com/socketio/socket.io-client-java
* C++: https://github.com/socketio/socket.io-client-cpp
* Swift: https://github.com/socketio/socket.io-client-swift
* Dart: https://github.com/rikulo/socket.io-client-dart
* Python: https://github.com/miguelgrinberg/python-socketio
* .Net: https://github.com/Quobject/SocketIoClientDotNet

# Как это работает?
Клиент попытается установить соединение [WebSocket](https://developer.mozilla.org/ru/docs/Web/API/WebSocket), если это возможно, и в противном случае вернется к длинному опросу HTTP.
WebSocket - это протокол связи, который обеспечивает полнодуплексный канал с малой задержкой между сервером и браузером. Более подробную информацию можно найти [здесь](https://ru.wikipedia.org/wiki/WebSocket).

И так, при хорошем раскладке, если:
* Браузер поддерживает WebSocket (97% браузеров в 2020 поддерживает)
* И нет ничего, что блокирует(proxy, firewall...) соединение клиента с сервером
Вы можете рассматривать клиент Socket.IO как «небольшую» оболочку вокруг API WebSocket.
Вместо того, чтобы писать:
```javascript
const socket = new WebSocket('ws://localhost:3000');

socket.onopen(() => {
  socket.send('Hello!');
});

socket.onmessage(data => {
  console.log(data);
});
```
На стороне клиента у вас будет:
```javascript
const socket = io('ws://localhost:3000');

socket.on('connect', () => {
  // either with send()
  socket.send('Hello!');

  // or with emit() and custom event names
  socket.emit('salutations', 'Hello!', { 'mr': 'john' }, Uint8Array.from([1, 2, 3, 4]));
});

// handle the event sent with socket.send()
socket.on('message', data => {
  console.log(data);
});

// handle the event sent with socket.emit()
socket.on('greetings', (elem1, elem2, elem3) => {
  console.log(elem1, elem2, elem3);
});
```
