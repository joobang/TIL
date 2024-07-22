## Nest.js

Nest.js의 공식문서에 따르면 아래와 같이 설명하고 있다.
> Nest(NestJS)는 효율적이고 확장 가능한 Node.js 서버 측 애플리케이션을 구축하기 위한 프레임워크입니다. 이 프레임워크는 진보적인 자바스크립트를 사용하며, 타입스크립트를 완벽하게 지원하면서도 개발자가 순수 자바스크립트로 코딩할 수 있도록 구축되었고, OOP(객체지향 프로그래밍), FP(함수형 프로그래밍), FRP(함수형 반응형 프로그래밍)의 요소를 결합하고 있습니다.
>
Nest는 기본적으로 Express HTTP 프레임워크를 사용합니다. 그러나 Nest는 플랫폼에 구애받지 않으므로 모든 Node HTTP 프레임워크에서 작동할 수 있습니다. 예를 들어, 노드 프레임워크를 개선할 수 있는 Fastify를 사용하도록 선택적으로 구성할 수 있습니다.
>
Nest는 이러한 일반적인 Node.js 프레임워크(Express/Fastify)보다 높은 수준의 추상화를 제공할 뿐만 아니라 해당 API를 개발자에게 직접 노출합니다. 따라서 개발자는 기본 플랫폼에서 사용할 수 있는 무수히 많은 타사 모듈을 자유롭게 사용할 수 있습니다.

Nest.js는 Express.js와 마찬가지로 Node.js 기반 백엔드 프레임워크이다.
Nest.js를 한 줄로 요약하면 다음과 같이 할 수 있을거 같다.
_"기본적으로 Express HTTP 프레임워크를 사용하고 Express와 달리 정형화된 구조로, Typescript 기반의 프레임워크이다."_

## javascript vs typescript
nest.js에 대해 알아보기전에 javascript와 typescript에 대해 알아보았다.

### javascript
- 웹 개발에 주로 사용된다.
- 동적 타입의 언어. 변수의 타입은 런타임에 결정되며 개발자는 변수를 선언할 때 타입을 지정하지 않는다.
- 클라이언트 측 스크립팅 언어 (Node.js로 서버 측에서도 사용 가능)
- 웹 페이지를 대화식으로 만드는 프로그래밍 언어

### typescript
- javascript의 상위 집합으로 javascript의 모든 기능을 사용할 수 있다.
- 정적 타입의 언어로 변수, 함수의 매개변수, 객체의 속성 등에 타입을 지정할 수 있다.
- typescript 컴파일러로 ts파일을 js파일로 컴파일된 파일을 실행한다.
- 클래스 기반 객체를 만들 수 있다.
- 클래스 기반이므로 객체 지향 프로그래밍 언어로 상속, 캡슐화 및 생성자를 지원할 수 있다.

## Nest.js 설치

우선 Node.js(version >= 16)가 설치되어 있어야 한다.
```
npm i -g @nestjs/cli
nest new project-name
```

위의 명령어로 Nest.js를 설치하고 새로운 프로젝트를 생성할 수 있다.
```
src
|-app.controller.spec.ts # 컨트롤러의 단위테스트를 위한 파일.
|-app.controller.ts      # 앱의 컨트롤러
|-app.module.ts			 # 앱의 루트 모듈
|-app.service.ts		 # 앱의 서비스
 -main.ts				 # 앱의 시작점
```
프로젝트를 생성하면 프로젝트 디렉토리 안에 위와 같은 구조의 파일이 생성된다.

## 구조
### main.ts
```js
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```
Nest 애플리케이션 인스턴스를 생성하기 위해 핵심 NestFactory 클래스를 사용한다. 
create() 메서드는 INestApplication 인터페이스를 충족하는 애플리케이션 객체를 반환한다.
main.ts 예시에서는 애플리케이션이 인바운드 HTTP 요청을 기다릴 수 있도록 HTTP 리스너를 시작하기만 하면 된다.

### app.module.ts

```js
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
데코레이터가 사용되었다. imports, controllers와 providers를 배열로 받아서 AppModule 클래스를 만들어주는 역할을 하는 것 같다. 여기저기 흩어져있는 코드를 AppModule로 묶어서 export한 뒤 NestFactory에서 받아서 서버를 만들어주는 걸로 예상된다.

### app.controller.ts

```js
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```
이번에도 역시 데코레이터가 사용되고 있다. 아무래도 Nestjs는 데코레이터를 적극적으로 사용해서 class 구조를 조작하는 듯 하다.

하나씩 살펴보면 **@Controller** 데코레이터를 달고있는 AppController클래스에서 AppService클래스를 인자의 타입으로 받은 뒤 **@Get** 데코레이터를 통해 AppService 타입 내의 getHello 메소드를 호출해 사용하는 듯 하다. 타입 지정만 받았을 뿐인데, 어떻게 그 안에 있는 메소드를 사용할 수 있는지는 모르겠다. 아마 app.module.ts내에서 providers로 전달됐기 때문에 호출되는 게 아닐까 추측

**@Controller** 데코레이터를 통해 Controller 클래스를 정의해 줄 수 있는 듯하다. Controller는 Express에서 Route 기능을 대신 해 주는 듯 하고 각 Controller안에 **@Get, @Post, @Put, @Delete** 등의 HTTP request를 정의하는 모양이다.

### app.service.ts

```js
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```
이번에도 역시 **@Injectable** 데코레이터를 사용해 클래스를 정의한다. 위의 app.controller.ts에서 사용한 getHello메소드가 정의되어 있다. 'Hello World!' 라는 문자열을 출력하는데 실제로 서버를 작동시키고 localhost:3000으로 접속하면 화면에 Hello World! 라는 문자열이 출력된다. 이를 통해 providers에 전달된 클래스의 메소드를 Controller를 통해서 사용할 수 있다고 확인할 수 있다.

지금까지의 구조를 대략적인 구조를 이미지로 설명하자면 아래와 같다.

![](https://velog.velcdn.com/images/ehwnghks/post/4188395b-832d-47e9-9001-ff634a6daea8/image.png)

전체적으로 NestJS는 분리할 수 있는 단위의 기능들을 모듈로 정의해서 가능한 독립적으로 사용한다. 물론, 다른 모듈과 상호작용을 하게 설계할 수도 있는 듯 하다. 가장 근본이 되는 App Module을 통해 또 다른 모듈을 imports를 통해 배열로 전달받을 수 있다. 하나의 모듈 내에는 복수의 Controller와 Service도 존재할 수 있다. 하지만 하나의 단위 기능은 모듈로 묶어야 NestJS를 효율적으로 사용할 수 있다.

## 💡 출처
--- 
https://docs.nestjs.com/
https://koras02.tistory.com/160
https://velog.io/@kimjeongwonn/NestJS-%EB%8F%85%ED%95%99-%EC%86%8C%EA%B0%9C
