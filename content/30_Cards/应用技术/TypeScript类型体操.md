---
type: procedure
tags:
  - TypeScript
  - 类型系统
created: 2026-04-21T20:52:00
---

# TypeScript 类型体操

类型体操是指利用 TypeScript 强大的类型系统，通过复杂的类型操作实现各种计算和逻辑推导的技术。

## 基础类型操作

### 1. 泛型约束

```typescript
// 基本泛型
function identity<T>(arg: T): T {
  return arg;
}

// 约束泛型
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### 2. 条件类型

```typescript
type Extract<T, U> = T extends U ? T : never;
type Exclude<T, U> = T extends U ? never : T;

// 条件类型嵌套
type Flatten<T> = T extends Array<infer U> ? U : T;
```

## 中级技巧

### 3. infer 推断

```typescript
// 从数组类型提取元素类型
type ArrayElement<T> = T extends (infer U)[] ? U : never;

// 从函数类型提取返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// 从 Promise 提取 resolve 类型
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;
```

### 4. 映射类型

```typescript
// 将所有属性变为只读
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 将所有属性变为可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 将属性变为必选
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// 选取部分属性
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// 排除部分属性
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

## 高级模式

### 5. 模板字面量类型

```typescript
type EventName<T extends string> = `on${Capitalize<T>}`;
type ButtonEvent = EventName<'click'>; // 'onClick'

type Conncat<T extends string, U extends string> = `${T}_${U}`;
```

### 6. 递归类型

```typescript
// DeepPartial - 深可选
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// DeepReadonly - 深只读
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// DeepRequired - 深必选
type DeepRequired<T> = {
  [P in keyof T]-?: T[P] extends object ? DeepRequired<T[P]> : T[P];
};
```

### 7. 分布式条件类型

```typescript
// Union 类型分发
type ToArray<T> = T extends any ? T[] : never;
type StrOrNumArray = ToArray<string | number>; // string[] | number[]

// 过滤 Union
type Filter<T, U> = T extends U ? never : T;
type Remaining = Filter<string | number | boolean, string | number>; // boolean
```

## 实战技巧

### 8. 类型安全的检查器

```typescript
type IsAny<T> = 0 extends (1 & T) ? true : false;
type IsNever<T> = [T] extends [never] ? true : false;
type IsUnion<T, U = T> = T extends U ? ([U] extends [T] ? false : true) : never;
```

### 9. Variadic Tuple Types

```typescript
type Concat<T extends any[], U extends any[]> = [...T, ...U];
type Push<T extends any[], V> = [...T, V];
type Unshift<T extends any[], V> = [V, ...T];

// 应用：函数参数组合
type Intersect<T extends (...args: any[]) => void, U extends (...args: any[]) => void> =
  (...args: Parameters<T>) => void;
```

### 10. 类型保护工具

```typescript
type IsString<T> = T extends string ? true : false;
type IsArray<T> = T extends any[] ? true : false;
type IsObject<T> = T extends object ? (T extends any[] ? false : true) : false;

// 空类型检查
type IsEmpty<T> = keyof T extends never ? true : false;
```

## 常用内置类型

| 类型 | 作用 |
|------|------|
| `ReturnType<T>` | 获取函数返回类型 |
| `Parameters<T>` | 获取函数参数类型元组 |
| `ConstructorParameters<T>` | 获取构造函数参数 |
| `InstanceType<T>` | 获取实例类型 |
| `Partial<T>` | 所有属性可选 |
| `Required<T>` | 所有属性必选 |
| `Readonly<T>` | 所有属性只读 |
| `Pick<T, K>` | 选取属性 |
| `Omit<T, K>` | 排除属性 |
| `Exclude<T, U>` | 从 T 排除可赋值给 U 的类型 |
| `Extract<T, U>` | 从 T 提取可赋值给 U 的类型 |
| `NonNullable<T>` | 去除 null 和 undefined |
| `Record<K, V>` | 创建键值对类型 |

## 练习建议

1. **从基础开始**：熟练掌握泛型和条件类型
2. **理解 infer**：infer 是高级类型操作的核心
3. **练习映射类型**：Keyof、in 操作符是类型变换的基础
4. **理解分发机制**：Union 类型在条件类型中的行为
5. **阅读源码**：学习 Lodash、utility-types 等库的源码

## 参考资源

- TypeScript 官方文档
- type-challenges - 类型练习题集
- utility-types - 常用类型工具库
