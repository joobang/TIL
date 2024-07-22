공식문서 내용 학습

# Controllers

> 컨트롤러는 사용자로부터 들어오는 요청을 처리하고 응답을 반환하는 일을 담당한다.

![](https://velog.velcdn.com/images/ehwnghks/post/4765f430-9ca6-4b89-8141-0c6c77156e90/image.png)

컨트롤러의 목적은 애플리케이션에 대한 특정 요청을 수신하는 것이다. 라우팅 메커니즘은 어떤 컨트롤러가 어떤 요청을 수신하는지 제어한다. 각 컨트롤러에는 둘 이상의 경로가 있는 경우가 많으며, 서로 다른 경로가 서로 다른 작업을 수행할 수 있다.

기본 컨트롤러를 생성하기 위해 클래스와 데코레이터를 사용한다.
데코레이터는 클래스를 필수 메타데이터와 연결하고 nest가 라우팅 맵을 생성할 수 있도록 한다.(요청을 해당 컨트롤러에 연결)

## Routing

아래 예시를 살펴보면 @Controller() 데코레이터에 'cats'라는 선택적 경로 접두사를 지정하고 있다. 접두사를 사용하면 관련된 여러 경로를 쉽게 그룹화할 수 있으며 코드의 중복을 줄일 수 있다.

```js
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll() {
    return 'This action returns all cats';
  }

  @Get(':id')
  findOne(id: string) {
    return `This action returns a #${id} cat`;
  }
}

```
위 예제에서 CatsController는 'cats'라는 접두사를 가진 경로를 처리하는 컨트롤러다. @Get() 데코레이터를 사용하여 HTTP GET 요청을 처리하는 메서드를 정의하고 있다. 첫 번째 findAll() 메서드는 '/cats' 경로에 대한 요청을 처리하며 두 번째 findOne(id: string) 메서드는 '/cats/:id' 경로에 대한 요청을 처리한다.

이렇게 @Controller() 데코레이터를 사용하여 경로 접두사를 지정함으로써 관련된 라우트를 쉽게 그룹화하고 코드의 중복을 최소화할 수 있다.

### Nest가 응답을 조작하기 위한 두 가지 옵션
#### standard
JavaScript 객체 또는 배열을 반환할 때 자동으로 JSON으로 직렬화된다. 그러나 JavaScript 기본 유형(예: 문자열, 숫자, boolean)을 반환하는 경우 Nest는 직렬화를 시도하지 않고 값만 보낸다. 이렇게 하면 응답 처리가 간단해지고 값만 반환하면 Nest가 나머지를 처리한다.
또한 응답의 상태 코드는 201을 사용하는 POST 요청을 제외하고 기본적으로 항상 200이다. @HttpCode(...) 데코레이터를 사용하여 응답의 상태 코드를 쉽게 변경할 수 있고 이 데코레이터는 각 핸들러 레벨에서 사용할 수 있다.

#### Library-specific
메소드 핸들러 signature(예: findAll(@Res() 응답))에서 @Res() 데코레이터를 사용하여 주입할 수 있는 라이브러리별(예: Express) 응답 개체를 사용할 수 있다. 이 접근 방식을 사용하면 해당 개체에 의해 노출된 기본 응답 처리 메서드를 사용할 수 있다. 예를 들어 Express를 사용하면 response.status(200).send()와 같은 코드를 사용하여 응답을 구성할 수 있다.

> Nest는 핸들러가 @Res() 또는 @Next()를 사용하는 시기를 감지하여 library-specific 방법을 선택했음을 나타낸다. @Res() 또는 @Next()를 사용하면 특정 라우트에 대해 NestJS의 표준 응답 처리 방식이 자동으로 비활성화된다. @Res() 또는 @Next()를 사용하면서도 표준 응답처리 방법을 유지하려면 @Res({ passthrough: true }) 데코레이터를 사용해야한다. 이는 응답 객체를 직접 조작할 수 있으면서도 (예: 쿠키나 헤더 설정) 나머지 응답은 NestJS가 처리하도록 할 수 있다.

## Request object
핸들러는 사용자 요청 세부정보에 액세스해야 하는 경우가 많다. Nest는 기본 플랫폼(기본적으로 Express)의 요청 객체에 대한 액세스를 제공한다. 핸들러의 signature에 @Req() 데코레이터를 추가하여 Nest에 요청 객체를 삽입하도록 지시하여 요청 객체에 액세스할 수 있다.

```js
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```
requset object는 HTTP 요청을 나타내며 요청 query string, parameters, HTTP 헤더 및 본문에 대한 속성을 갖는다. 대부분의 경우 이러한 속성을 수동으로 가져올 필요는 없다. 대신 기본적으로 사용할 수 있는 @Body() 또는 @Query()와 같은 전용 데코레이터를 사용할 수 있다. 다음은 제공된 데코레이터와 이들이 나타내는 일반 플랫폼별 object 목록이다.

| <center>데코레이터</center> |  <center>일반 오브젝트</center> |
| :- |  -: |
| @Request(), @Req() | req |
| @Response(), @Res()* | res |
|@Next()|	next|
|@Session()	|req.session|
|@Param(key?: string)|	req.params / req.params[key]|
|@Body(key?: string)|	req.body / req.body[key]|
|@Query(key?: string)	|req.query / req.query[key]|
|@Headers(name?: string)	|req.headers / req.headers[name]|
|@Ip()	|req.ip|
|@HostParam()	|req.hosts|

HTTP 플랫폼들(예: Express와 Fastify) 간의 타이핑 호환성을 위해, Nest는 @Res()와 @Response() 데코레이터를 제공한다. 이 두 데코레이터는 개발자에게 HTTP 플랫폼의 기본 응답 객체에 직접 접근할 수 있게 해준다. @Res()는 @Response()의 다른 이름일 뿐이다.
@Res() 또는 @Response()를 메소드 핸들러에 주입하면, 해당 핸들러는 'Library-specific mode'가 됩니다. 즉, 개발자가 직접 응답을 관리해야 한다. 'Library-specific mode'에서는, 개발자가 HTTP 응답을 직접 처리하고 완료해야 한다. 예를 들어, res.json(...) 또는 res.send(...)와 같은 방법으로 응답을 보내야 하며, 그렇지 않으면 서버가 멈추게 된다.

## Resources
앞서 우리는 cats 리소스(GET 경로)를 가져오기 위한 엔드포인트를 정의했다. 또한 일반적으로 새 레코드를 생성하는 엔드포인트를 제공하려고 한다. 이를 위해 POST 핸들러를 생성하면 다음과 같다.
```ts
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```
이것은 간단하다. Nest는 @Get(), @Post(), @Put(), @Delete(), @Patch(), @Options() 및 @Head() 등 모든 표준 HTTP 메서드에 대한 데코레이터를 제공한다. 또한 @All()은 이를 모두 처리하는 엔드포인트를 정의한다.

## Route wildcards
패턴 기반 경로도 지원된다. 예를 들어 별표는 와일드카드로 사용되며 모든 문자 조합과 일치한다.
```js
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```
'ab*cd' 경로 경로는 abcd, ab_cd, abecd 등과 일치한다. ?, +, * 및 () 문자는 경로 경로에 사용될 수 있으며 정규식 대응의 하위 집합이다. 하이픈(-)과 점(.)은 문자열 기반 경로로 문자 그대로 해석된다.

## Status code
언급한 대로 응답 상태 코드는 201인 POST 요청을 제외하고 기본적으로 항상 200이다. 핸들러 수준에서 @HttpCode(...) 데코레이터를 추가하여 이 동작을 쉽게 변경할 수 있다.
```js
// Import HttpCode from the @nestjs/common package.
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```
상태 코드는 정적이 아니지만 다양한 요인에 따라 달라지는 경우가 많다. 이 경우 library-specific (@Res()을 사용하여 주입) 개체를 사용할 수 있다.(또는 오류가 발생하는 경우 예외 발생).

## Headers
사용자 정의 응답 헤더를 지정하려면 @Header() 데코레이터나 라이브러리별 응답 객체를 사용하고 res.header()를 직접 호출하면 된다.
```js
// Import Header from the @nestjs/common package.
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

## Redirection
응답을 특정 URL로 리디렉션하려면 @Redirect() 데코레이터나 라이브러리별 응답 개체를 사용하고 res.redirect()를 직접 호출하면 된다.

@Redirect()는 url과 statusCode라는 두 개의 인수를 사용하며 둘 다 선택 사항이다. 생략된 경우 statusCode의 기본값은 302(Found)이다.
```js
@Get()
@Redirect('https://nestjs.com', 301)
```
때로는 HTTP 상태 코드나 리디렉션 URL을 동적으로 확인해야 할 수도 있다.
그렇게 하려면 (@nestjs/common의) HttpRedirectResponse 인터페이스를 따르는 객체를 반환하면 된다.

메서드가 return 값(예: URL)을 반환하면, 이 값은 데코레이터에 전달된 기존의 인수를 오버라이드(덮어쓰기)한다.
```js
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```
위와 같이 @Redirect('https://docs.nestjs.com', 302) 이지만 최종 리다이렉트 URL은 'https://docs.nestjs.com/v5/' 가 된다.

## Route parameters
정적 경로가 있는 경로는 요청의 일부로 동적 데이터를 받아들여야 할 때 작동하지 않는다.(예: ID 1의 고양이를 가져오기 위한 GET /cats/1). 매개변수가 있는 경로를 정의하기 위해 경로 경로에 경로 매개변수 토큰을 추가하여 요청 URL의 해당 위치에서 동적 값을 캡처할 수 있다. 아래 @Get() 데코레이터 예시의 경로 매개변수 토큰은 이 사용법을 보여줍니다. @Get() 데코레이터 안에 라우트 파라미터 토큰을 사용하여 동적 값을 캡쳐할 수 있으며, @Param() 데코레이터를 사용하여 메서드에서 이 값을 접근할 수 있다.
```ts
// Import Param from the @nestjs/common package.
@Get(':id')
findOne(@Param() params: any): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```
> 라우트에 파라미터를 사용할 때, 이러한 파라미터가 포함된 라우트는 정적 경로 라우트 후에 선언되어야 한다. 이렇게 하면, 정적 경로로의 트래픽이 파라미터화된 경로에 의해 방해받는 것을 방지할 수 있다.
예를 들어 /users/profile, /users/:id 두개의 라우트가 선언 되어 있을 때 '/users/profile' 먼저 선언되어 있어야한다. 그렇지 않으면, :id 파라미터가 "profile"이라는 값을 캡쳐하게 되어, 의도하지 않은 동작을 유발할 수 있다.

@Param()은 메서드의 파라미터(위 예시의 'params')를 장식하는 데 사용된다. 이렇게 하면 라우트 파라미터가 메서드 본문 내에서 해당 장식된 메서드 파라미터의 속성으로 사용할 수 있다. 위 코드에서 볼 수 있듯이, params.id를 참조하여 id 파라미터에 접근할 수 있다. 데코레이터에 특정 파라미터 토큰을 전달하면, 메서드 본문에서 라우트 파라미터를 이름으로 직접 참조할 수 있다.

```ts
@Get(':id')
@Bind(Param('id'))
findOne(id) {
  return `This action returns a #${id} cat`;
}
```

## Sub-Domain Routing
@Controller 데코레이터는 호스트 옵션을 사용하여 들어오는 요청의 HTTP 호스트가 특정 값과 일치하도록 요구할 수 있다.
```js
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```
라우트 경로와 마찬가지로 호스트 옵션은 토큰을 사용하여 호스트 이름에서 해당 위치의 동적 값을 캡처할 수 있다. 아래 @Controller() 데코레이터 예제의 호스트 매개변수 토큰은 이 사용법을 보여준다. 이런 방법으로 선언된 호스트 파라미터는 @HostParam() 데코레이터를 사용하여 접근할 수 있으며, 이 데코레이터는 메서드 시그니처에 추가되어야 한다.
```js
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}

```

## Scopes
Nest에서는 거의 모든 것이 들어오는 요청 간에 공유된다. 예를 들어, 데이터베이스 커넥션 풀이나 싱글턴 서비스와 같이 전역 상태를 가지는 서비스 등이 있다.
Node.js는 각 요청을 별도의 스레드에서 처리하는 요청/응답 멀티 스레드 상태 없음 모델을 따르지 않는다. 따라서, 싱글턴 인스턴스를 사용하는 것은 애플리케이션에 완전히 안전하다.
그러나 일부 경우에는, 예를 들어 GraphQL 애플리케이션에서의 요청별 캐싱, 요청 추적, 또는 멀티 테넌시와 같은 경우에 각 요청이 독립적으로 처리되어야 하며, 각 요청에 대해 컨트롤러의 새로운 인스턴스가 생성되어야 하는 것이 좋다.

## Asynchronicity
우리는 최신 JavaScript를 사랑하며 데이터 추출이 대부분 비동기식이라는 것을 알고 있다. 그렇기 때문에 Nest는 비동기 함수를 지원하고 잘 작동한다.
모든 비동기 함수는 프로미스를 반환해야 한다. NestJS는 이러한 비동기 객체(Promise 등)를 직접 처리할 수 있으며, 해당 비동기 객체가 완료될 때까지 기다린 다음, 최종 결과를 HTTP 응답으로 반환한다.
```ts
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```
게다가 Nest 라우트 핸들러는 RxJS observable streams을 반환할 수 있어 훨씬 더 강력하다. 
```ts
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```
Nest는 아래 소스를 자동으로 구독하고 마지막으로 내보낸 값을 가져온다(스트림이 완료되면).
위의 두 접근 방식 모두 작동하며 요구 사항에 맞는 것을 사용할 수 있다.

## Request payloads
이전 POST 라우트 핸들러 예에서는 클라이언트 매개변수를 허용하지 않았다. 여기에 @Body() 데코레이터를 추가하여 이 문제를 해결할 수 있다.

> DTO를 인터페이스가 아닌 클래스로 선언하는이유

하지만 먼저(TypeScript를 사용하는 경우) DTO(Data Transfer Object) 스키마를 결정해야 한다. DTO는 네트워크를 통해 데이터가 전송되는 방법을 정의하는 개체이다. 
TypeScript 인터페이스나 간단한 클래스를 사용하여 DTO 스키마를 결정할 수 있다. 
흥미롭게도 여기에서 _**클래스를 사용하는 것이 좋다.**_ 왜? **클래스는 JavaScript ES6 표준의 일부이므로 컴파일된 JavaScript에서 실제 엔터티로 유지된다. 반면에 TypeScript 인터페이스는 코드가 JavaScript로 트랜스파일(컴파일)되는 과정에서 제거된다.** 즉, 런타임에는 인터페이스 정보를 참조할 수 없게 된다. 이는 Pipes 같은 Nest의 기능이 런타임에 변수의 메타타입(변수의 타입 정보)에 액세스할 수 있을 때 추가적인 기능을 제공할 수 있기 때문에 중요하다.

```ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```
위와 같은 DTO를 생성하여 Controller 내부에서 새로 생성된 DTO를 사용할 수 있다.
```ts
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```
ValidationPipe는 메서드 핸들러가 수신해서는 안 되는 속성을 필터링할 수 있다. 이 경우 허용되는 속성을 화이트리스트에 추가할 수 있으며, 화이트리스트에 포함되지 않은 속성은 결과 개체에서 자동으로 제거된다. CreateCatDto 예에서 허용 목록은 이름, 나이 및 품종 속성이다. [관련링크](https://docs.nestjs.com/techniques/validation#stripping-properties).

## Handling errors
[여기](https://docs.nestjs.com/exception-filters)에 오류 처리(예: 예외 작업)에 대한 별도의 장이 있다.

## Full resource sample
다음은 기본 컨트롤러를 생성하기 위해 사용 가능한 여러 데코레이터를 사용하는 예다. 이 컨트롤러는 내부 데이터에 액세스하고 조작하는 몇 가지 방법을 제공한다.
```ts
// cats.controller.ts
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

## Getting up and running
위의 컨트롤러가 완전히 정의된 경우 Nest는 여전히 CatsController가 존재하는지 알지 못하므로 결과적으로 이 클래스의 인스턴스를 생성하지 않는다.

컨트롤러는 항상 모듈에 속하므로 @Module() 데코레이터 내에 컨트롤러 배열을 포함시킨다. 루트 AppModule을 제외한 다른 모듈을 아직 정의하지 않았으므로 이를 사용하여 CatsController를 소개한다.
```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```
@Module() 데코레이터를 사용하여 모듈 클래스에 메타데이터를 연결했으며 이제 Nest는 어떤 컨트롤러를 마운트해야 하는지 쉽게 반영할 수 있다.

## Library-specific approach
지금까지 응답을 조작하는 Nest 표준 방식에 대해 논의했다. 응답을 조작하는 두 번째 방법은 라이브러리별 응답(library-specific) 개체를 사용하는 것이다. 특정 응답 객체를 주입하려면 @Res() 데코레이터를 사용해야 한다. 차이점을 보여주기 위해 CatsController를 다음과 같이 다시 작성할 수 있다.
```ts
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```
이 접근 방식은 효과가 있고 실제로 응답 개체에 대한 전체 제어(헤더 조작, 라이브러리 관련 기능 등)를 제공하여 어떤 면에서 더 많은 유연성을 허용하지만 주의해서 사용해야 한다. 
일반적으로 이 접근 방식은 덜 명확하며 몇 가지 단점이 있다. 가장 큰 단점은 코드가 플랫폼에 종속되고(기본 라이브러리(underlying libraries)가 응답 개체에 대해 다른 API를 가질 수 있으므로) 테스트하기가 더 어렵다는 것이다(응답 개체를 Mock해야 하는 등).

모킹이란, 테스트에서 실제 객체 대신 사용되는 가짜 객체를 만드는 것을 의미한다.

또한 위 예제에서는, 인터셉터와 @HttpCode() / @Header() 데코레이터와 같이 Nest 표준 응답 핸들링에 의존하는 Nest 기능들과의 호환성을 잃어버리게 된다. passthrough 옵션을 true로 설정해 이것을 수정할 수 있다.

```ts
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
```
이제 기본 응답 객체와 상호 작용할 수 있게 되었다.(예: 특정 조건에 따라 쿠키 또는 헤더 설정) 나머지는 Nest에게 맡기면 된다.