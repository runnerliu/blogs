## DDD系列 - 资料汇总

### 文章汇总

#### DDD 概念

- [领域驱动设计在互联网业务开发中的实践](https://link.juejin.cn?target=https%3A%2F%2Ftech.meituan.com%2F2017%2F12%2F22%2Fddd-in-practice.html)

#### 六边形架构

- [Hexagonal architecture](https://link.juejin.cn?target=https%3A%2F%2Falistair.cockburn.us%2Fhexagonal-architecture%2F)
- [从三明治到六边形](https://link.juejin.cn?target=https%3A%2F%2Finsights.thoughtworks.cn%2Farchitecture-from-sandwich-to-hexagon%2F)
- [Example Implementation of a Hexagonal Architecture](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fthombergs%2Fbuckpal)
- [Hexagonal Architecture with Java and Spring](https://link.juejin.cn?target=https%3A%2F%2Freflectoring.io%2Fspring-hexagonal%2F)

### 学习总结

#### 工程结构

```
.
├── app-name
│   ├── adapter
│   │   ├── in
│   │   │   └── web
│   │   └── out
│   │       └── persistence
│   ├── application
│   │   ├── port
│   │   │   ├── in
│   │   │   └── out
│   │   └── service
│   └── domain
├── common
└── main

app-name：业务名称
 - adapter：适配器
   - in：输入，与外界进行交互
     - web
   - out：输出，与数据库、缓存等进行交互，是 application.port.out 的持久化实现
     - persistence：持久化实现
 - application：应用
   - port：端口，一般定义为接口，面向接口编程，向适配器提供接口
     - in：向适配器提供处理接口
     - out：输出端口，向 service 模块中提供持久化等接口
   - service：输入输出端口的具体业务实现
 - domain：领域模型，从业务中抽象出的领域对象
common：基础设施
main：主文件
```


