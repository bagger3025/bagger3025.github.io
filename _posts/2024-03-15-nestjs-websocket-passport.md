---
title: Nestjs에서 Websocket을 사용할 때 passportjs로 인증하기
tags:
  - backend
  - Nestjs
post_time: 2024-03-15T18:29:30+09:00
edit_time: 2024-03-15T18:47:31+09:00
---
# 문제점

[Nest.js](https://docs.nestjs.com/)를 이용해 채팅을 구현하기 위해 Websocket을 사용하였다. 이때 로그인한 사용자가 본인 정보를 인증해야 하는데, http Request에서 인증한 정보를 가져오고 싶었다. Websocket에서 Express middleware인 passportjs를 사용하고 싶었으나, 쉽지 않았고 그 해결 기록을 남긴다.

# 해결 과정

[stack overflow 게시글](https://stackoverflow.com/questions/68684439/passport-session-authentication-with-websockets-and-nest-js-not-authenticating)을 많이 참고하였다. 질문글에 벌써 많은 부분이 해결되어 공유되고 있었다.

socket.io의 공식 문서 [Mongodb Adaptor](https://socket.io/docs/v4/mongo-adapter/)과 Nestjs의 공식 문서 Websocket의 [Adaptor파트](https://docs.nestjs.com/websockets/adapter)를 보면서 구현해 놓은 것이 있었는데, 이것을 기반으로 추가와 수정만 하면 되었다.

```typescript
  createIOServer(port: number, options?: ServerOptions): any {
    const server = super.createIOServer(port, options);
    server.adapter(this.adapterConstructor);
    return server;
  }
```

위 코드는 공식 문서에 공유되어 있는 `createIOServer` 메서드이다. 위에서는 이렇게 만들어 서버를 반환하고 있는데, 반환하기 전 middleware를 추가하면 되었다.

위 Stack overflow 게시글과 답변을 토대로 몇 가지를 수정하였다. 일단 `withCredentials: true`를 Frontend에 넣었고 `credentials: true`를 Backend에 넣었다. 이렇게 하니 쿠키가 Backend로 잘 전달되었다.

하지만 middleware를 추가할 때 메인에서 stack overflow 게시글처럼 `app.use()`를 사용하고 싶지 않았다. http Request를 받을 때에도 `app.module.ts`에서 `NestModule`을 implements하여 미들웨어를 적용하고 있었다. (Nest.js 공식 문서 [Applying Middleware](https://docs.nestjs.com/middleware#applying-middleware)  참고)

Stack overflow 게시글에는 다음과 같은 해결법이 제시되어 있었다.

```typescript
    const wrap = (middleware) => (socket, next) =>
      middleware(socket.request, {}, next);
    
    server.use((socket, next) => {
      socket.data.username = 'test'; //passing random property to see if use method is working
      next();
    });
    server.use(wrap(this.session));
    server.use(wrap(passport.initialize()));
    server.use(wrap(passport.session()));
```

`wrap`이라는 함수를 반환하는 함수를 정의해 `wrap(middleware)`형식으로 부르면 `server.use()`함수에 전달할 함수를 반환할 수 있게 만든 것이다. 하지만 Typescript를 사용하고 있었기에 이 방법은 통하지 않았다. `middleware`에 `{}`를 전달하는 부분에서 변환할 수 없다고 에러가 나왔다.

SocketIO에서 middleware를 사용할 수 있는 방법을 검색했더니 바로 SocketIO 공식문서가 나왔다. [Compatibility with Express Middlewares](https://socket.io/docs/v4/middlewares/#compatibility-with-express-middleware)는 찾던 바로 그것이었다. 이것과 Stack Overflow의 첫 번째 답변을 바탕으로 코드를 다음과 같이 수정했다.

```typescript
    server.engine.use(this.session);
    server.engine.use(passport.initialize());
    server.engine.use(passport.session());
```

`this.session`은 생성자에서 받아오는데, 원래 작성한 코드를 다음과 같이 변형하였다.

`main.ts`

```typescript
const FileStore = _FileStore(session);

const passportSession = session({
secret: process.env.SESSION_SECRET || 'development',
resave: false,
saveUninitialized: false,
store: new FileStore(),
});

const mongoIoAdapter = new MongoIoAdapter(app, passportSession);
```

`mongo.adaptor.ts`

```typescript
  constructor(app: INestApplication, session: RequestHandler) {
    super(app);
    this.session = session;
  }
```

Http Request의 세션과 passportjs의 인증까지 다 연동되었다.