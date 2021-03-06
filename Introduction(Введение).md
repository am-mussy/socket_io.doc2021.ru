# Что такое Socket.IO:
Socket.IO - это библиотека, которая обеспечивает двустороннюю связь в реальном времени и на основе событий между браузером и сервером.
Это состоит из:
* Node.js server: [Source](https://github.com/socketio/socket.io) | [API](https://socket.io/docs/v3/server-api/)
* И клиентской библиотеки Javascript для браузера (которую также можно запускать из Node.js): [Source](https://github.com/socketio/socket.io-client) | [API](https://socket.io/docs/v3/client-api/)
![](https://socket.io/images/bidirectional-communication.png)

### Есть также несколько клиентских реализаций на других языках, которые поддерживаются сообществом:
* Java: https://github.com/socketio/socket.io-client-java
* C++: https://github.com/socketio/socket.io-client-cpp
* Swift: https://github.com/socketio/socket.io-client-swift
* Dart: https://github.com/rikulo/socket.io-client-dart
* Python: https://github.com/miguelgrinberg/python-socketio
* .Net: https://github.com/Quobject/SocketIoClientDotNet

# Как это работает?
Клиент попытается установить соединение [WebSocket](https://developer.mozilla.org/ru/docs/Web/API/WebSocket), если это возможно, а в противном случае вернется к длинным запросам(long polling*) HTTP.
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
API на стороне сервера аналогичен, вы также получаете объект сокета, который расширяет класс Node.js EventEmitter:
```javascript
const io = require('socket.io')(3000);

io.on('connection', socket => {
  // либо с send()
  socket.send('Hello!');

  // или emit() и кастомным(своим) именем события
  socket.emit('greetings', 'Hey!', { 'ms': 'jane' }, Buffer.from([4, 3, 3, 1]));

  // обработка события, отправленного socket.send()
  socket.on('message', (data) => {
    console.log(data);
  });

  // обработка события, отправленного socket.emit()
  socket.on('salutations', (elem1, elem2, elem3) => {
    console.log(elem1, elem2, elem3);
  });
});
```
Socket.IO предоставляет дополнительные функции по сравнению с простым объектом WebSocket, которые перечислены ниже.

Но сначала давайте подробно рассмотрим, чем не является библиотека Socket.IO.

#Чем Socket.IO не является

Socket.IO НЕ является реализацией WebSocket. Хотя Socket.IO действительно использует WebSocket в качестве доставщика, когда это возможно, он добавляет дополнительные метаданные к каждому пакету.
Вот почему клиент WebSocket не сможет подключиться к серверу Socket.IO, а клиент Socket.IO не сможет подключиться к обычному серверу WebSocket.
```javascript
// ВНИМАНИЕ: клиент НЕ сможет подключиться!
const socket = io('ws://echo.websocket.org');
```
Если вы ищете простой сервер WebSocket, обратите внимание на [ws](https://github.com/websockets/ws) или [uWebSockets.js](https://github.com/uNetworking/uWebSockets.js). 
Также [ведутся](https://github.com/nodejs/node/issues/19308) переговоры о включении сервера WebSocket в ядро Node.js. 
На стороне клиента вас может заинтересовать пакет [robust-websocket](https://github.com/nathanboktae/robust-websocket).


# Минимальный рабочий пример
Если вы новичок в экосистеме Node.js, ознакомьтесь с [руководством](https://socket.io/get-started/chat) (ПЕРЕВЕСТИ) по началу работы, которое идеально подходит для начинающих.
А если нет, приступим прямо сейчас!
Серверную библиотеку можно установить из NPM:
```javascript
npm install socket.io
```
Более подробную информацию об установке можно найти на [странице](https://socket.io/docs/v3/server-installation/)(ПЕРЕВЕСТИ) установки Сервера.
Затем давайте создадим файл index.js со следующим содержанием:
```javascript
const content = require('fs').readFileSync(__dirname + '/index.html', 'utf8');

const httpServer = require('http').createServer((req, res) => {
  // serve the index.html file
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('Content-Length', Buffer.byteLength(content));
  res.end(content);
});

const io = require('socket.io')(httpServer);

io.on('connection', socket => {
  console.log('connect');
});

httpServer.listen(3000, () => {
  console.log('go to http://localhost:3000');
});
```
Здесь запускается классический HTTP-сервер Node.js для обслуживания файла index.html, и к нему подключен сервер Socket.IO. Пожалуйста, посетите [страницу](https://socket.io/docs/v3/server-initialization/)(ПЕРЕВЕСТИ) инициализации сервера, чтобы узнать о различных способах создания сервера.

Давайте создадим рядом с ним файл index.html:
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Minimal working example</title>
</head>
<body>
    <ul id="events"></ul>

    <script src="/socket.io/socket.io.js"></script>
    <script>
        const $events = document.getElementById('events');

        const newItem = (content) => {
          const item = document.createElement('li');
          item.innerText = content;
          return item;
        };

        const socket = io();

        socket.on('connect', () => {
          $events.appendChild(newItem('connect'));
        });
    </script>
</body>
</html>
```
Наконец, давайте запустим наш сервер:
```javascript
node index.js
```
И voilà(вуаля)!
![](https://socket.io/images/minimal-example-connect.gif)

Объект сокета с обеих сторон расширяет класс EventEmitter, поэтому: 
* отправка события выполняется с помощью: socket.emit () 
* получение события осуществляется путем регистрации слушателя: socket.on (<имя события>, <слушатель>)

### Чтобы отправить событие с сервера клиенту
Давайте обновим файл index.js (на стороне сервера):
```javascript
io.on('connection', socket => {
  let counter = 0;
  setInterval(() => {
    socket.emit('hello', ++counter);
  }, 1000);
});
```
И файл index.html (на стороне клиента):
```javascript
const socket = io();

socket.on('connect', () => {
  $events.appendChild(newItem('connect'));
});

socket.on('hello', (counter) => {
  $events.appendChild(newItem(`hello - ${counter}`));
});
```
Вот что получилось:
![](https://socket.io/images/minimal-example-server-to-client.gif)
### Чтобы отправить сообщение от клиента на сервер
Давайте обновим файл index.js (на стороне сервера):
```javascript
io.on('connection', socket => {
  socket.on('hey', data => {
    console.log('hey', data);
  });
});
```
И файл index.html (на стороне клиента):
```javascript
const socket = io();

socket.on('connect', () => {
  $events.appendChild(newItem('connect'));
});

let counter = 0;
setInterval(() => {
  ++counter;
  socket.emit('hey', { counter }); // the object will be serialized for you
}, 1000);
```
Получилось:
![](https://socket.io/images/minimal-example-client-to-server.gif)

Теперь давайте подробно рассмотрим функции, предоставляемые Socket.IO.

## Особенности

Его основные особенности:

### Надежность
Соединения устанавливаются даже при использовании:
* прокси и балансировщика нагрузки
* персонального firewall и антивирусного программного обеспечения

Для этой цели он использует [Engine.IO](https://github.com/socketio/engine.io), который сначала устанавливает соединение с длительным опросом, а затем пытается перейти на более качественные транспорты, которые «тестируются» на стороне, например, WebSocket. Пожалуйста, смотрите раздел [Цели](https://github.com/socketio/engine.io#goals) для получения дополнительной информации.

### Поддержка автоматического переподключения

Если не задано иное, отключенный клиент будет бесконечно пытаться повторно подключиться, пока сервер снова не станет доступным. Пожалуйста, ознакомьтесь с доступными вариантами повторного подключения [здесь](https://socket.io/docs/v3/client-api/#new-Manager-url-options) (ПЕРЕВЕСТИ).

### Обнаружение отключения

На уровне Engine.IO реализован механизм контрольных сообщений, позволяющий как серверу, так и клиенту знать, когда другой больше не отвечает.
Эта функциональность достигается с помощью таймеров, установленных как на сервере, так и на клиенте, с использованием значений тайм-аута (параметры pingInterval и pingTimeout), совместно используемых во время подтверждения соединения. Эти таймеры требуют, чтобы все последующие клиентские вызовы направлялись на тот же сервер, отсюда и требование закрепления сеанса при использовании нескольких узлов.

### Бинарная поддержка
Могут быть созданы любые сериализуемые структуры данных, в том числе:
* [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) и [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) в браузере
* [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) и  [Buffer](https://nodejs.org/api/buffer.html) в Node.js

### Поддержка мультиплексирования
Чтобы разделить задачи в вашем приложении (например, для каждого модуля или на основе разрешений), Socket.IO позволяет вам создавать несколько [пространств имен](https://socket.io/docs/v3/namespaces/) (ПЕРЕВЕСТИ), которые будут действовать как отдельные каналы связи, но будут использовать одно и то же базовое соединение.

##### **Long Polling** - это технология, которая позволяет получать информацию о новых событиях с помощью «длинных запросов». Сервер получает запрос, но отправляет ответ на него не сразу, а лишь тогда, когда произойдет какое-либо событие (например, поступит новое входящее сообщение), либо истечет заданное время ожидания.
