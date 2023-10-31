# Modules
모듈은 @Module() 데코레이터로 주석이 달린 클래스입니다. @Module() 데코레이터는 Nest가 애플리케이션 구조를 구성하는 데 사용하는 메타데이터를 제공합니다.

![](https://docs.nestjs.com/assets/Modules_1.png)

## Feature modules
이전의 CatsController와 CatsService는 동일한 애플리케이션 도메인에 속합니다. 서로 밀접하게 관련되어 있으므로 기능 모듈(feature module)로 이동하는 것이 좋습니다. 기능 모듈(feature module)은 단순히 특정 기능과 관련된 코드를 구성하여 코드를 체계적으로 유지하고 명확한 경계를 설정합니다. 이는 특히 애플리케이션 및/또는 팀의 규모가 커짐에 따라 복잡성을 관리하고 SOLID 원칙으로 개발하는 데 도움이 됩니다.

### SOLID 원칙

**단일 책임 원칙 (Single Responsiblity Principle)**

모든 클래스는 각각 하나의 책임만 가져야 한다. 클래스는 그 책임을 완전히 캡슐화해야 함을 말한다.

- 사칙연산 함수를 가지고 있는 계산 클래스가 있다고 치자. 이 상태의 계산 클래스는 오직 사칙연산 기능만을 책임진다. 이 클래스를 수정한다고 한다면 그 이유는 사직연산 함수와 관련된 문제일 뿐이다.

**개방-폐쇄 원칙 (Open Closed Principle)**

확장에는 열려있고 수정에는 닫혀있는. 기존의 코드를 변경하지 않으면서( Closed), 기능을 추가할 수 있도록(Open) 설계가 되어야 한다는 원칙을 말한다.

- 캐릭터를 하나 생성한다고 할때, 각각의 캐릭터가 움직임이 다를 경우 움직임의 패턴 구현을 하위 클래스에 맡긴다면 캐릭터 클래스의 수정은 필요가없고(Closed) 움직임의 패턴만 재정의 하면 된다.(Open)

**리스코프 치환 원칙 (Liskov Substitution Principle)**

자식 클래스는 언제나 자신의 부모 클래스를 대체할 수 있다는 원칙이다. 즉 부모 클래스가 들어갈 자리에 자식 클래스를 넣어도 계획대로 잘 작동해야 한다.
자식클래스는 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 수행하도록 해야 LSP를 만족한다.

**인터페이스 분리 원칙 (Interface Segregation Principle)**

한 클래스는 자신이 사용하지않는 인터페이스는 구현하지 말아야 한다. 하나의 일반적인 인터페이스보다 여러개의 구체적인 인터페이스가 낫다.

**의존 역전 원칙 (Dependency Inversion Principle)**

의존 관계를 맺을 때 변화하기 쉬운 것 또는 자주 변화하는 것보다는 변화하기 어려운 것, 거의 변화가 없는 것에 의존하라는 것이다. 한마디로 구체적인 클래스보다 인터페이스나 추상 클래스와 관계를 맺으라는 것이다.

---

아래는 CatsModule을 생성코드 입니다.
```ts
//cats/cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

위에서는 cats.module.ts 파일에 CatsModule을 정의하고 이 모듈과 관련된 모든 것을 cats 디렉토리로 옮겼습니다. 마지막으로 해야 할 일은 이 모듈을 루트 모듈(app.module.ts 파일에 정의된 AppModule)로 가져오는 것입니다.

```ts
//app.modules.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

디렉토리 구조는 다음과 같이 됩니다.

```bash
src
├──cats
│   ├──dto
│       ├──create-cat.dto.ts
│   ├──interfaces
│       ├──cat.interface.ts
│   ├──cats.controller.ts
│   ├──cats.module.ts
│   ├──cats.service.ts
├──app.module.ts
├──main.ts
```

## Shared modules


Nest에서 모듈은 기본적으로 싱글톤이므로 여러 모듈 간에 모든 공급자의 동일한 인스턴스를 쉽게 공유할 수 있습니다.
![](https://docs.nestjs.com/assets/Shared_Module_1.png)

모든 모듈은 자동으로 공유 모듈(shared module)이 됩니다. 일단 생성되면 모든 모듈에서 재사용할 수 있습니다. 여러 다른 모듈 간에 CatsService 인스턴스를 공유한다고 가정해 보겠습니다.

이를 위해서는 먼저 아래와 같이 모듈의 exports 배열에 CatsService 공급자(provider)를 추가하여 내보내야 합니다.

```ts
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

이제 CatsModule을 import하는 모든 모듈은 CatsService에 액세스할 수 있으며 이를 import하는 다른 모든 모듈과 동일한 인스턴스를 공유합니다.

## Module re-exporting
위에서 보았듯이 모듈은 내부 공급자를 내보낼 수(export) 있습니다. 또한 가져온 모듈을 다시 내보낼 수도 있습니다. 아래 예시에서는 CommonModule을 CoreModule에서 가져오고 내보내서 이 모듈을 가져오는 다른 모듈에서 사용할 수 있도록 합니다.

```ts
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}

```

## Dependency injection
모듈 클래스는 공급자(provider)도 주입할 수 있습니다(예: 구성 목적으로)

```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```
그러나 모듈 클래스 자체는 순환 종속성(Circular dependency)으로 인해 공급자(provider)를 주입할 수 없습니다.

### Circular dependency
순환 종속성은 두 클래스가 서로 종속될 때 발생합니다. 예를 들어 클래스 A에는 클래스 B가 필요하고 클래스 B에도 클래스 A가 필요합니다. Nest에서는 모듈 간 및 공급자 간에 순환 종속성이 발생할 수 있습니다.

가능하면 순환 종속성을 피해야 하지만 항상 그렇게 할 수는 없습니다. 이러한 경우 Nest는 두 가지 방법으로 공급자 간의 순환 종속성을 해결할 수 있습니다. 한 가지 기법으로 정방향 참조(forward referencing)를 사용하고, 다른 기법으로 ModuleRef 클래스를 사용하여 DI 컨테이너에서 공급자 인스턴스를 검색하는 방법을 설명합니다.

또한 모듈 간의 순환 종속성을 해결하는 방법도 설명합니다.

> "barrel files"/index.ts 파일을 사용하여 가져오기를 그룹화할 때 순환 종속성이 발생할 수도 있습니다. 모듈/프로바이더 클래스에 대해서는 배럴 파일을 생략해야 합니다. 예를 들어, 배럴 파일과 같은 디렉터리 내의 파일을 가져올 때는 배럴 파일을 사용해서는 안 됩니다. 즉, cats/cats.controller에서 cats/cats.service 파일을 가져오기 위해 cats를 가져와서는 안 됩니다.

#### Forward reference
정방향 참조를 사용하면 Nest가 forwardRef() 유틸리티 함수를 사용하여 아직 정의되지 않은 클래스를 참조할 수 있습니다. 

예를 들어 CatsService와 CommonService가 서로 종속된 경우, 관계의 양쪽에서 @Inject() 및 forwardRef() 유틸리티를 사용하여 순환 종속성을 해결할 수 있습니다. 그렇지 않으면 모든 필수 메타데이터를 사용할 수 없으므로 Nest에서 인스턴스화하지 않습니다.
```ts
@Injectable()
export class CatsService {
  constructor(
    @Inject(forwardRef(() => CommonService))
    private commonService: CommonService,
  ) {}
}
```
이는 관계의 한 측면을 설명한 것입니다. 이제 CommonService에 대해서도 똑같이 해보겠습니다.

```ts
@Injectable()
export class CommonService {
  constructor(
    @Inject(forwardRef(() => CatsService))
    private catsService: CatsService,
  ) {}
}
```

>인스턴스화 순서는 불확실합니다. 코드가 먼저 호출되는 생성자에 따라 달라지지 않는지 확인해야합니다.

#### ModuleRef class alternative
forwardRef()를 사용하는 대신 코드를 리팩터링하고 ModuleRef 클래스를 사용하여 순환 관계의 한쪽에서 provider를 받아오도록 할 수 있습니다. [여기](https://docs.nestjs.com/fundamentals/module-ref)에서 ModuleRef 유틸리티 클래스에 대해 자세히 알아보세요.

#### Module forward reference
모듈 간의 순환 종속성을 해결하려면 모듈 연결의 양쪽에서 동일한 forwardRef() 유틸리티 함수를 사용하세요.
```ts
@Module({
  imports: [forwardRef(() => CatsModule)],
})
export class CommonModule {}
```
CatsModule 에서도 동일하게 작성합니다.
```ts
@Module({
  imports: [forwardRef(() => CommonModule)],
})
export class CatsModule {}
```

## Global modules
모든 곳에서 동일한 모듈 세트를 가져와야 한다면(import) 지루할 수 있습니다.
Nest와 달리 Angular의 provider는 글로벌 범위에 등록됩니다. 한번 정의하면 어디서나 사용할 수 있습니다. 

그러나 Nest는 모듈 범위 내에 provider를 캡슐화합니다. 캡슐화된 모듈을 먼저 가져오지 않으면(import) 다른 곳에서 모듈의 provider를 사용할 수 없습니다.

모듈에 @Global() decorator를 붙히면 import 없이 해당 모듈의 Provider들을 사용할 수 있습니다.

```ts
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```
@Global() 데코레이터는 모듈을 전역 범위로 만듭니다. 전역 모듈은 일반적으로 루트 또는 코어 모듈에 의해 한 번만 등록되어야 합니다. 

위의 예에서 CatsService 공급자는 어디에나 있으며 서비스를 주입하려는 모듈은 imports 배열에서 CatsModule을 가져올 필요가 없습니다.

## Dynamic modules
Nest 모듈 시스템에는 동적 모듈이라는 강력한 기능이 포함되어 있습니다. 이 기능을 사용하면 공급자를 동적으로 등록하고 구성할 수 있는 사용자 지정 가능한 모듈을 쉽게 만들 수 있습니다.

Dynamic modules의 자세한 내용은 [여기](https://docs.nestjs.com/fundamentals/dynamic-modules)에서 확인할 수 있다.


다음은 DatabaseModule에 대한 동적 모듈 정의의 예입니다.

```ts
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```
> forRoot() 메서드는 동적 모듈을 동기식 또는 비동기식(즉, 프로미스를 통해)으로 반환할 수 있습니다.

이 모듈은 기본적으로 Connection provider를 정의하지만(@Module() 데코레이터 메타데이터에 있음), 추가로 forRoot() 메서드에 전달된 엔티티 및 옵션 객체에 따라 레포지토리와 같은 공급자 컬렉션을 노출합니다. 

동적 모듈이 반환하는 속성은 @Module() 데코레이터에 정의된 기본 모듈 메타데이터를 재정의하지 않고 확장한다는 점에 유의하세요. 이것이 정적으로 선언된 Connection provider와 동적으로 생성된 repository providers가 모듈에서 내보내지는(exported) 방식입니다.


전역 범위(global scope)에 동적 모듈을 등록하려면 전역 속성을 true로 설정합니다.
```
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

DatabaseModule은 다음과 같은 방식으로 가져오고 구성할 수 있습니다.
```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```
동적 모듈을 차례로 다시 내보내려면(re-export) 내보내기(exports) 배열에서 forRoot() 메서드 호출을 생략할 수 있습니다.
```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```


## 출처
---
https://hckcksrl.medium.com/solid-%EC%9B%90%EC%B9%99-182f04d0d2b

https://docs.nestjs.com/modules