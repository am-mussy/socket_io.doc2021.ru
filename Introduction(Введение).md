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
