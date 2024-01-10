---
title: Nestjs
date: '2024-01-10 09:19:27'
head: []
outline: deep
sidebar: false
prev: false
next: false
---

# Nestjs

## Ioc 的一些概念

1. imports 导入其他的模块
2. exports  导出当前的 providers 供其他模块使用
3. provides  可以被注入, 也可以注入别的对象
4. controllers 只能被注入

## Providers 的几种类型

1. 类

```typescript
@Module({
  imports: [PersonModule, OtherModule, CccModule],
  controllers: [AppController],
  providers: [AppService]
})

// 等同于

@Module({
  imports: [PersonModule, OtherModule, CccModule],
  controllers: [AppController],
  providers: [{
   provide: AppService,
   useClass: AppService
 }]
})
```

2. 字符串

```typescript
{
    provide: 'app_service',
    useClass: AppService
}

// 此时需要手动@Inject注入
@Inject('app_service') private readonly appService: AppService
```

3. 具体的值

```typescript
{
  provide: 'person',
  useValue: {
    name: 'jack',
    age: 20
  }
}

@Controller()
export class AppController{
  constructor(
    private readonly appService: AppService,
    @Inject('person')
    private readonly person: { name: string; age: number }
  ) {}
}
```

4. 动态的值

<span style="font-weight: bold;" data-type="strong">useFactory 支持异步</span>

```typescript
// 1. 第一种场景
{
    provide: 'person2',
    useFactory() {
        return {
            name: 'bbb',
            desc: 'cccc'
        }
    }
}

@Inject('person2') private readonly person2: {name: string, desc: string}


// 2. 第二种场景, 可以通过参数注入别的provider
{
  provide: 'person3',

  useFactory(person: { name: string }, appService: AppService) {
    return {
      name: person.name,
      desc: appService.getHello()
    }
  },
  inject: ['person', AppService] // 注意这2个必须在providers中被引入声明过
}
```

5. 指定别名

```
{
  provide: 'person4',
  useExisting: 'person2'
}
```

‍

## 生命周期

​![](http://www.kdocs.cn/api/v3/office/copy/SzdjMVBabGhFb29QWTlqWE5TdmlVSmVtRFdsc0sxNXRmZXBXT2RyZ3didG1xcUlnc0Y3bDlhajUyWkFGVEJ6aGhXcllLOE1uMGw1cTBqM0hBeW44YnRBQ2FpUnQzYWVxNGF5UkxFbnhzK2cxS0VyaFpmc3N5WTBUZ3RtYitiSVBscW9kSkErQW5HYTZiREJmdVhabUZIMnZpYjJaU2h6VERaWm5QM041S3VPME5pQmxTOVRTaklPNVgzdURXcXl3cGxzZDFQRlhiRm5qa0lDQTBUMWhBSit5bndJdHk3VWRQZkJCamRVQnNTcFZweWFEQ3FjOW81VVpsL2tVakNVYldETmtpL3hUSUJBPQ==/attach/object/PJTK4BIAGE?)​

首先，递归初始化模块，会依次调用模块内的 controller、provider 的 onModuleInit 方法，然后再调用 module 的 onModuleInit 方法。 全部初始化完之后，再依次调用模块内的 controller、provider 的 onApplicationBootstrap 方法，然后调用 module 的 onApplicationBootstrap 方法, 卸载类似, PS: Module 都是在最后

‍

```typescript
import {
  Controller,
  Get,
  Inject,
  OnModuleInit,
  OnApplicationBootstrap
} from '@nestjs/common'
import { AppService } from './app.service'

@Controller()
export class AppController implements OnModuleInit, OnApplicationBootstrap {
  constructor(
    private readonly appService: AppService,
    @Inject('person')
    private readonly person: { name: string; age: number }
  ) {}

  onModuleInit() {
    console.log('onModuleInit in app controller')
  }

  onApplicationBootstrap() {
    console.log('onApplicationBootstrap in app controller')
  }
}
```

咳咳咳
