# Providers

providers는 Nest의 기본 개념입니다. 기본 Nest 클래스 중 상당수는 서비스, 저장소, 팩토리, 도우미 등 공급자로 취급될 수 있습니다. providers의 주요 아이디어는 종속성으로 주입될 수 있다는 것입니다. 이는 객체들이 서로 다양한 관계를 맺을 수 있으며, 이러한 객체들을 연결("wiring up")하는 기능이 주로 Nest 런타임 시스템에 위임될 수 있습니다. 

이 말인 즉 개발자가 직접 객체를 관리하고 연결하는 대신, NestJS의 런타임 시스템이 이러한 작업을 대부분 자동으로 처리해줄 수 있어, 개발자는 더 복잡한 로직이나 기능의 구현에 집중할 수 있게 된다는 내용입니다.

![](https://docs.nestjs.com/assets/Components_1.png)

이전 장에서는 간단한 CatsController를 구축했습니다. 컨트롤러는 HTTP 요청을 처리하고 더 복잡한 작업을 공급자(Providers)에게 위임해야 합니다. 공급자는 모듈에서 공급자로 선언되는 일반 JavaScript 클래스입니다.

## Services

간단한 CatsService를 만드는 것부터 시작해 보겠습니다. 이 서비스는 데이터 저장 및 검색을 담당하며 CatsController에서 사용하도록 설계되었으므로 공급자로 정의하기에 좋은 후보입니다.
```ts
// cats.service.ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

CatsService는 하나의 속성과 두 개의 메서드를 가진 기본 클래스입니다. 유일한 새로운 기능은 @Injectable() 데코레이터를 사용한다는 것입니다. 

@Injectable() 데코레이터는 CatsService가 Nest IoC 컨테이너에서 관리할 수 있는 클래스임을 선언하는 메타데이터를 첨부합니다. 그런데 이 예제에서는 다음과 같은 Cat 인터페이스도 사용합니다.

>IoC 컨테이너는 객체의 생성과 생명 주기를 관리하며, 필요한 의존성을 주입해주는 역할을 합니다.

```ts
// interfaces/cat.interface.ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이를 검색하는 서비스 클래스가 있으므로 CatsController 내에서 이를 사용해 보겠습니다.
```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```
CatsService는 클래스 생성자를 통해 주입됩니다. private 구문의 사용에 주목하세요. 이 단축어를 사용하면 동일한 위치에서 즉시 catsService 멤버를 선언하고 초기화할 수 있습니다.

## Dependency injection

Nest는 일반적으로 Dependency injection(종속성 주입)이라고 알려진 강력한 디자인 패턴을 기반으로 구축되었습니다. 공식 Angular 문서에서 이 개념에 대한 [훌륭한 기사](https://angular.io/guide/dependency-injection)를 읽는 것이 좋습니다.

NestJS는 TypeScript를 기반으로 하고 있어, 타입을 기반으로 의존성을 해결(주입)하는 것이 매우 쉽습니다. 예를 들어, CatsService라는 서비스가 있다면, NestJS는 이 서비스의 인스턴스를 생성하고, 필요한 곳에 이 인스턴스를 주입합니다.

일반적으로 서비스는 싱글턴 패턴을 따르며, 애플리케이션에서 한 번만 인스턴스가 생성됩니다. 만약 이미 어딘가에서 CatsService의 인스턴스가 요청되어 생성되었다면, NestJS는 새로 인스턴스를 생성하지 않고 기존의 인스턴스를 반환합니다.

NestJS는 생성자를 통해 의존성을 주입합니다. 즉, CatsService의 인스턴스는 필요한 컨트롤러의 생성자로 전달됩니다.

간단한 예를 들면,
```ts
import { Injectable, Controller } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}
}

```
위 코드에서 CatsController는 CatsService에 의존하고 있습니다. CatsService의 인스턴스는 CatsController의 생성자를 통해 주입됩니다. NestJS의 DI 시스템은 자동으로 CatsService의 인스턴스를 생성하거나 재사용하여 CatsController에 주입합니다.

## Scopes
Providers는 일반적으로 애플리케이션 수명 주기와 동기화된 수명("scope")을 갖습니다. 애플리케이션이 부트스트랩되면 모든 종속성을 해결해야 하므로 모든 공급자를 인스턴스화해야 합니다. 마찬가지로 애플리케이션이 종료되면 각 공급자가 삭제됩니다. 그러나 공급자 수명 요청 범위를 지정하는 방법도 있습니다. 이러한 기술에 대한 자세한 내용은 [여기](https://docs.nestjs.com/fundamentals/injection-scopes)에서 확인할 수 있습니다.

## Custom providers
Nest에는 공급자 간의 관계를 해결하는 제어 반전('IoC') 컨테이너가 내장되어 있습니다. 이 기능은 위에서 설명한 종속성 주입 기능의 기초가 되지만 실제로는 지금까지 설명한 것보다 훨씬 더 강력합니다. 공급자를 정의하는 방법에는 여러 가지가 있습니다. 일반 값, 클래스, 비동기 또는 동기 팩토리를 사용할 수 있습니다. [여기](https://docs.nestjs.com/fundamentals/custom-providers)에 더 많은 예제가 제공됩니다.

## Optional providers
선택적 의존성이란, 클래스가 작동하는데 있어 반드시 필요하지 않은, 즉 없어도 코드가 에러 없이 작동할 수 있는 의존성을 말합니다.

예를 들어, 클래스가 특정 구성 객체(configuration object)에 의존할 수 있지만, 구성 객체가 주어지지 않더라도 기본값을 사용하여 클래스가 정상적으로 작동해야 하는 경우가 있을 수 있습니다. 이러한 경우, 구성 제공자(configuration provider)가 없더라도 에러가 발생하지 않으므로 의존성이 선택적이 됩니다.

선택적인 의존성을 나타내기 위해 생성자의 시그니처에 @Optional() 데코레이터를 사용할 수 있습니다. 이 데코레이터를 사용하면, 해당 의존성이 없어도 NestJS가 에러를 발생시키지 않고 기본값을 사용하거나 무시하게 됩니다.

```ts
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```
위의 예시에서는 사용자 정의 공급자를 사용하고 있으며, 이것이 바로 HTTP_OPTIONS 사용자 정의 토큰을 포함하는 이유입니다. 이전 예제에서는 생성자의 클래스를 통해 종속성을 나타내는 생성자 기반 인젝션을 보여주었습니다. [여기](https://docs.nestjs.com/fundamentals/custom-providers)에서 사용자 정의 공급자와 관련 토큰에 대해 자세히 알아보세요.

## Property-based injection
공급자가 생성자 메서드를 통해 주입되므로 지금까지 우리가 사용한 기술을 생성자 기반 주입이라고 합니다. 매우 특정한 경우에는 속성 기반 주입(Property-based injection)이 유용할 수 있습니다.

예를 들어 최상위 클래스가 하나 또는 여러 공급자에 의존하는 경우 생성자의 하위 클래스에서 super()를 호출하여 공급자를 끝까지 전달하는 것은 매우 지루할 수 있습니다. 이를 방지하려면 속성 수준에서 @Inject() 데코레이터를 사용할 수 있습니다.

```ts
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```
클래스가 다른 클래스를 확장(extend)하지 않는다면, 항상 생성자 기반의 의존성 주입을 사용해야 합니다.

생성자 기반의 의존성 주입은 코드의 명확성, 테스트 용이성 등 여러 면에서 이점이 있습니다. 클래스의 의존성이 생성자에서 명시적으로 주입되기 때문에, 클래스가 어떤 의존성을 가지고 있는지 쉽게 파악할 수 있으며, 테스트 시에도 특정 의존성을 쉽게 목(mock) 객체로 교체할 수 있기 때문입니다.

## Provider registration
이제 공급자(CatsService)를 정의했고 해당 서비스의 소비자(CatsController)가 있으므로 주입을 수행할 수 있도록 Nest에 서비스를 등록해야 합니다. 모듈 파일(app.module.ts)을 편집하고 @Module() 데코레이터의 공급자 배열에 서비스를 추가하여 이를 수행합니다.
```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```
이제 Nest는 CatsController 클래스의 종속성을 해결할 수 있습니다.
이제 디렉토리 구조는 다음과 같습니다.

```bash
src
├──cats
│   ├──dto
│       ├──create-cat.dto.ts
│   ├──interfaces
│       ├──cat.interface.ts
│   ├──cats.controller.ts
│   ├──cats.service.ts
├──app.module.ts
├──main.ts
```

## Manual instantiation
지금까지 Nest가 종속성 해결에 대한 대부분의 세부 사항을 자동으로 처리하는 방법을 논의했습니다. 특정 상황에서는 기본 제공 종속성 주입 시스템 외부로 나가 수동으로 공급자를 검색하거나 인스턴스화해야 할 수도 있습니다. 우리는 아래에서 그러한 두 가지 주제에 대해 간략하게 논의합니다.

기존 인스턴스를 가져오거나 공급자를 동적으로 인스턴스화하려면 모듈 참조[(Module reference)](https://docs.nestjs.com/fundamentals/module-ref)를 사용할 수 있습니다.

bootstrap() 함수 내에서 공급자를 얻으려면(예를 들어 컨트롤러가 없는 독립 실행형 애플리케이션의 경우 또는 부트스트래핑 중에 구성 서비스를 활용하려는 경우) 독립 실행형 애플리케이션[(Standalone applications)](https://docs.nestjs.com/standalone-applications)을 참조하세요.