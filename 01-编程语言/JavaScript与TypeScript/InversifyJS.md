# 介绍

InversifyJS 是一个用于 TypeScript 和 JavaScript 的轻量级 IoC（控制反转）容器，常用于依赖注入（Dependency Injection，简称 DI）场景，能够提升应用结构的模块化、可测试性和可维护性。
- # 示例
- 在tsconfig.json中启用了如下设置
  
  ```
  {
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
  }
  ```
- 文件结构
  
  ```
  src/
  ├── interfaces/
  │   └── Warrior.ts
  ├── entities/
  │   └── Ninja.ts
  ├── types.ts
  ├── inversify.config.ts
  └── app.ts
  ```
- 定义接口
  
  ```
  export interface Warrior {
  fight(): string;
  sneak(): string;
  }
  ```
- 实现类
  
  ```
  import { injectable } from "inversify";
  import "reflect-metadata";
  import { Warrior } from "../interfaces/Warrior";
  
  @injectable()
  export class Ninja implements Warrior {
  public fight(): string {
    return "ninja fight!";
  }
  
  public sneak(): string {
    return "ninja sneak!";
  }
  }
  ```
- 定义类型标识符（types.ts）
  
  ```
  const TYPES = {
  Warrior: Symbol.for("Warrior"),
  };
  
  export default TYPES;
  ```
- 配置IoC容器
  
  ```
  import { Container } from "inversify";
  import TYPES from "./types";
  import { Warrior } from "./interfaces/Warrior";
  import { Ninja } from "./entities/Ninja";
  
  const container = new Container();
  container.bind<Warrior>(TYPES.Warrior).to(Ninja);
  
  export default container;
  ```
- 使用容器解析依赖
  
  ```
  import "reflect-metadata";
  import container from "./inversify.config";
  import TYPES from "./types";
  import { Warrior } from "./interfaces/Warrior";
  
  const ninja = container.get<Warrior>(TYPES.Warrior);
  console.log(ninja.fight()); // 输出: ninja fight!
  console.log(ninja.sneak()); // 输出: ninja sneak!
  ```
- # 参考文档
  
  [https://inversify.io/](https://inversify.io/)
-