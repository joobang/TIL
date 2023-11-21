# Authentication

인증은 대부분의 애플리케이션에서 필수적인 부분입니다. 인증을 처리하는 데는 다양한 접근 방식과 전략이 있습니다. 모든 프로젝트에 적용되는 접근 방식은 특정 애플리케이션 요구 사항에 따라 다릅니다. 이 장에서는 다양한 요구 사항에 맞게 조정할 수 있는 여러 가지 인증 접근 방식을 제시합니다.

요구 사항을 구체화해 보겠습니다. 이 사용 사례의 경우 클라이언트는 사용자 이름과 비밀번호로 인증을 시작합니다. 인증되면 서버는 인증을 증명하기 위해 후속 요청 시 인증 헤더에 전달자 토큰으로 전송될 수 있는 JWT를 발행합니다. 또한 유효한 JWT가 포함된 요청에만 액세스할 수 있는 보호된 경로를 생성합니다.

첫 번째 요구 사항인 사용자 인증부터 시작하겠습니다. 그런 다음 JWT를 발행하여 이를 확장하겠습니다. 마지막으로 요청에서 유효한 JWT를 확인하는 보호된 경로를 생성합니다.

## Creating an authentication module
AuthModule과 그 안에 AuthService 및 AuthController를 생성하는 것부터 시작하겠습니다. AuthService를 사용하여 인증 로직을 구현하고 AuthController를 사용하여 인증 엔드포인트를 노출할 것입니다.
```bash
$ nest g module auth
$ nest g controller auth
$ nest g service auth
```
AuthService를 구현하면서 사용자 작업을 UsersService에 캡슐화하는 것이 유용하다는 것을 알게 될 것이므로 이제 해당 모듈과 서비스를 생성해 보겠습니다.

```bash
$ nest g module users
$ nest g service users
```
생성된 파일의 기본 내용을 아래와 같이 바꿉니다. 
샘플 앱의 경우, UserServcie는 단순히 하드코딩된 인메모리 사용자 목록과 사용자 이름으로 사용자를 검색하는 찾기 메서드를 유지 관리합니다. 

실제 앱에서는 선택한 라이브러리(예: TypeORM, Sequelize, 몽구스 등)를 사용하여 사용자 모델과 지속성 레이어를 빌드할 수 있습니다.
```ts
// users/users.service.ts
import { Injectable } from '@nestjs/common';

// This should be a real class/interface representing a user entity
export type User = any;

@Injectable()
export class UsersService {
  private readonly users = [
    {
      userId: 1,
      username: 'john',
      password: 'changeme',
    },
    {
      userId: 2,
      username: 'maria',
      password: 'guess',
    },
  ];

  async findOne(username: string): Promise<User | undefined> {
    return this.users.find(user => user.username === username);
  }
}
```
UsersModule에서 필요한 유일한 변경 사항은 이 모듈 외부에서 볼 수 있도록 @Module 데코레이터의 내보내기(exports) 배열에 UsersService를 추가하는 것입니다.(곧 AuthService에서 사용하게 될 것입니다).

```ts
// users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

```

## Implementing the "Sign in" endpoint
AuthService는 사용자를 검색하고 비밀번호를 확인하는 작업을 수행합니다. 이를 위해 signIn() 메서드를 생성합니다. 아래 코드에서는 편리한 ES6 스프레드 연산자를 사용하여 사용자 객체에서 비밀번호 속성을 제거한 후 반환합니다. 이는 사용자 객체를 반환할 때 비밀번호나 기타 보안 키와 같은 민감한 필드를 노출하지 않으려는 일반적인 관행입니다.

```ts
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async signIn(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user?.password !== pass) {
      throw new UnauthorizedException();
    }
    const { password, ...result } = user;
    // TODO: Generate a JWT and return it here
    // instead of the user object
    return result;
  }
}
```

물론 실제 애플리케이션에서는 비밀번호를 일반 텍스트로 저장하지 않습니다. 대신 솔트 처리된 단방향 해시 알고리즘이 포함된 bcrypt와 같은 라이브러리를 사용할 것입니다. 이 접근 방식을 사용하면 해시된 비밀번호만 저장한 다음 저장된 비밀번호를 들어오는 비밀번호의 해시된 버전과 비교하므로 사용자 비밀번호를 일반 텍스트로 저장하거나 노출하지 않습니다. 샘플 앱을 단순하게 유지하기 위해 이러한 절대적인 의무를 위반하고 일반 텍스트를 사용했습니다. 실제 앱에서는 이렇게 하지 마세요!

이제 AuthModule을 업데이트하여 UsersModule을 가져옵니다.
```ts
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [UsersModule],
  providers: [AuthService],
  controllers: [AuthController],
})
export class AuthModule {}
```

이제 AuthController를 열고 signIn() 메서드를 추가해 보겠습니다.

이 메서드는 클라이언트에서 사용자를 인증하기 위해 호출됩니다. 이 메서드는 요청 본문에서 사용자 이름과 비밀번호를 받고, 사용자가 인증되면 JWT 토큰을 반환합니다.

```ts
// auth/auth.controller.ts
import { Body, Controller, Post, HttpCode, HttpStatus } from '@nestjs/common';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @HttpCode(HttpStatus.OK)
  @Post('login')
  signIn(@Body() signInDto: Record<string, any>) {
    return this.authService.signIn(signInDto.username, signInDto.password);
  }
}
```

이상적으로는 Record<string, any> 유형을 사용하는 대신 DTO 클래스를 사용하여 요청 본문의 모양을 정의해야 합니다. 자세한 내용은 [유효성 검사(validation)](https://docs.nestjs.com/techniques/validation) 챕터를 참조하세요.

## JWT token
이제 인증 시스템의 JWT 부분으로 넘어갈 준비가 되었습니다. 요구 사항을 검토하고 구체화해 보겠습니다:

1. 사용자가 사용자 이름/비밀번호로 인증할 수 있도록 허용하여 보호된 API 엔드포인트에 대한 후속 호출에서 사용할 수 있도록 JWT를 반환해야 합니다. 이 요구사항을 충족하기 위한 작업은 순조롭게 진행 중입니다. 이를 완료하려면 JWT를 발급하는 코드를 작성해야 합니다.
2. 유효한 JWT가 Bearer 토큰으로 존재하는 경우에만 접근할 수 있도록 보호되는 API 경로를 생성하기

JWT requirements을 지원하기 위해 하나의 추가 패키지를 설치해야 합니다.
```bash
$ npm install --save @nestjs/jwt
```

nestjs/jwt 패키지는 JWT 조작에 도움이 되는 유틸리티 패키지입니다. 여기에는 JWT 토큰 생성 및 확인이 포함됩니다.

서비스를 깔끔하게 모듈화하기 위해 authService에서 JWT 생성을 처리합니다.

 auth 폴더에서 auth.service.ts 파일을 열고 JwtService를 삽입한 다음 signIn 메서드를 업데이트하여 아래와 같이 JWT 토큰을 생성합니다.

 ```ts
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService
  ) {}

  async signIn(username, pass) {
    const user = await this.usersService.findOne(username);
    if (user?.password !== pass) {
      throw new UnauthorizedException();
    }
    const payload = { sub: user.userId, username: user.username };
    return {
      access_token: await this.jwtService.signAsync(payload),
    };
  }
}
 ```

 사용자 객체 프로퍼티의 하위 집합에서 JWT를 생성하기 위해 signAsync() 함수를 제공하는 @nestjs/jwt 라이브러리를 사용하고 있으며, 이 함수는 단일 access_token 프로퍼티가 있는 간단한 객체로 반환합니다. 
 
 참고: JWT 표준과 일관성을 유지하기 위해 sub라는 속성 이름을 선택하여 userId 값을 보유합니다. 인증 서비스에 JwtService 공급자(provider)를 삽입(inject)하는 것을 잊지 마세요.

 이제 새 종속성을 가져오고 JwtModule을 구성하기 위해 AuthModule을 업데이트해야 합니다.
먼저 auth 폴더에 constants.ts를 생성하고 다음 코드를 추가합니다.
```ts
// auth/constants.ts
export const jwtConstants = {
  secret: 'DO NOT USE THIS VALUE. INSTEAD, CREATE A COMPLEX SECRET AND KEEP IT SAFE OUTSIDE OF THE SOURCE CODE.',
};
```

이 키를 사용하여 JWT 서명 및 확인 단계 간에 키를 공유할 것입니다.


>이 키를 공개적으로 노출하지 마세요. 여기서는 코드가 수행하는 작업을 명확히 하기 위해 공개했지만, 프로덕션 시스템에서는 secrets vault, 환경 변수 또는 구성 서비스(Configuration Service) 등의 적절한 방법을 사용하여 이 키를 보호해야 합니다.

이제 auth 폴더에서 auth.module.ts를 열고 다음과 같이 업데이트합니다.
```ts
//auth/auth.module.ts
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';
import { JwtModule } from '@nestjs/jwt';
import { AuthController } from './auth.controller';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    JwtModule.register({
      global: true,
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}
```

>작업을 더 쉽게 하기 위해 JwtModule을 글로벌로 등록하고 있습니다. 즉, 애플리케이션의 다른 곳에서 JwtModule을 임포트할 필요가 없습니다.

구성 객체를 전달하여 register()를 사용하여 JwtModule을 구성합니다. 

계속해서 cURL을 사용하여 경로를 다시 테스트해 보겠습니다. UsersService에 하드코딩된 모든 사용자 객체를 사용하여 테스트할 수 있습니다.

```bash
$ # POST to /auth/login
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme"}' -H "Content-Type: application/json"
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}
$ # Note: above JWT truncated
```

## Implementing the authentication guard
유효한 JWT(Json Web Token)가 요청에 포함되어 있도록 요구하여 엔드포인트를 보호할 수 있다.
이를 위해 경로를 보호하는 데 사용할 수 있는 AuthGuard를 생성합니다.
```ts
// auth/auth.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { jwtConstants } from './constants';
import { Request } from 'express';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    if (!token) {
      throw new UnauthorizedException();
    }
    try {
      const payload = await this.jwtService.verifyAsync(
        token,
        {
          secret: jwtConstants.secret
        }
      );
      // 💡 We're assigning the payload to the request object here
      // so that we can access it in our route handlers
      request['user'] = payload;
    } catch {
      throw new UnauthorizedException();
    }
    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

이제 보호 경로를 구현하고 AuthGuard를 등록하여 보호할 수 있습니다.

auth.controller.ts 파일을 열고 아래와 같이 업데이트합니다.

```ts
// auth.controller.ts
import {
  Body,
  Controller,
  Get,
  HttpCode,
  HttpStatus,
  Post,
  Request,
  UseGuards
} from '@nestjs/common';
import { AuthGuard } from './auth.guard';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @HttpCode(HttpStatus.OK)
  @Post('login')
  signIn(@Body() signInDto: Record<string, any>) {
    return this.authService.signIn(signInDto.username, signInDto.password);
  }

  @UseGuards(AuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}
```

방금 생성한 AuthGuard를 GET /profile 경로에 적용하여 보호할 수 있도록 합니다.
앱이 실행 중인지 확인하고 cURL을 사용하여 경로를 테스트합니다.

```bash
$ # GET /profile
$ curl http://localhost:3000/auth/profile
{"statusCode":401,"message":"Unauthorized"}

$ # POST /auth/login
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme"}' -H "Content-Type: application/json"
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2Vybm..."}

$ # GET /profile using access_token returned from previous step as bearer code
$ curl http://localhost:3000/auth/profile -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2Vybm..."
{"sub":1,"username":"john","iat":...,"exp":...}
```

AuthModule에서 JWT의 만료 시간을 60초로 구성했습니다. 
이는 너무 짧은 만료 시간이며 토큰 만료 및 새로 고침에 대한 세부 사항을 다루는 것은 이 문서의 범위를 벗어납니다. 하지만 JWT의 중요한 특성을 보여주기 위해 이 방법을 선택했습니다. 

인증 후 60초를 기다렸다가 GET /auth/프로필 요청을 시도하면 401 권한 없음 응답을 받게 됩니다. 이는 @nestjs/jwt가 자동으로 JWT의 만료 시간을 확인하므로 애플리케이션에서 직접 확인해야 하는 수고를 덜어주기 때문입니다.

이제 JWT 인증 구현이 완료되었습니다. 이제 자바스크립트 클라이언트(예: Angular/React/Vue) 및 기타 자바스크립트 앱이 트위터 API 서버와 안전하게 인증하고 통신할 수 있습니다.

## Enable authentication globally
대부분의 엔드포인트를 기본적으로 보호해야 하는 경우, 인증 가드를 전역 가드로 등록하고 각 컨트롤러 위에 @UseGuards() 데코레이터를 사용하는 대신 어떤 경로를 공개해야 하는지 플래그를 지정하면 됩니다.

먼저, 다음 construction을 사용하여 AuthGuard를 전역 가드로 등록합니다(예: AuthModule 등 모든 모듈에서)

```
providers: [
  {
    provide: APP_GUARD,
    useClass: AuthGuard,
  },
],
```

이 설정이 완료되면 Nest는 모든 엔드포인트에 AuthGuard를 자동으로 바인딩합니다.

이제 경로를 공개로 선언하는 메커니즘을 제공해야 합니다. 이를 위해 SetMetadata 데코레이터 팩토리 함수를 사용하여 사용자 정의 데코레이터를 만들 수 있습니다.

```ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

위 파일에서 두 개의 constants를 export 했습니다.
하나는 IS_PUBLIC_KEY라는 메타데이터 키이고, 다른 하나는 Public이라고 부를 새 데코레이터 자체입니다(프로젝트에 맞는 다른 이름을 지정할 수 있습니다).

이제 사용자 정의 @Public() 데코레이터가 생겼으므로 다음과 같이 모든 메서드를 데코레이션하는 데 사용할 수 있습니다.
```ts
@Public()
@Get()
findAll() {
  return [];
}
```

마지막으로, "isPublic" 메타데이터가 발견되면 AuthGuard가 참을 반환하도록 해야 합니다. 이를 위해 Reflector 클래스를 사용하겠습니다(자세한 내용은 [여기](https://docs.nestjs.com/guards#putting-it-all-together)를 참조하세요).

```ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService, private reflector: Reflector) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (isPublic) {
      // 💡 See this condition
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    if (!token) {
      throw new UnauthorizedException();
    }
    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: jwtConstants.secret,
      });
      // 💡 We're assigning the payload to the request object here
      // so that we can access it in our route handlers
      request['user'] = payload;
    } catch {
      throw new UnauthorizedException();
    }
    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

```

## Passport integration
[Passport](https://docs.nestjs.com/guards#putting-it-all-together)는 커뮤니티에서 잘 알려져 있고 많은 프로덕션 애플리케이션에서 성공적으로 사용되는 가장 인기 있는 node.js 인증 라이브러리입니다. 

이 라이브러리를 Nest 애플리케이션과 통합하는 방법은 @nestjs/passport 모듈을 사용하여 간단합니다.

Passport와 NestJS를 통합하는 방법을 알아보려면 이 [챕터](https://docs.nestjs.com/recipes/passport)를 확인하세요.