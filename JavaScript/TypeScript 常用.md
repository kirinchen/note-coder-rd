---
title: TypeScript 常用
tags: [Typescript]

---

# TypeScript 常用

## Enum

### Enum to Array

```typescript=
export enum Weeks {  
    MONDAY = 1,  
    TUESDAY= 2,  
    WEDNESDAY = 3,  
}

const arrayObjects = []  
      
for (const [propertyKey, propertyValue] of Object.entries(Weeks)) {  
      if (!Number.isNaN(Number(propertyKey))) {  
        continue;  
    }  
    arrayObjects.push({ id: propertyValue, name: propertyKey });  
}  
  
console.log(arrayObjects);  
```

```typescript=
const ans = Object.values(RunState);
```

## 引用

### public 多個物件
```typescript!
export default { Loading, Loading2 } 
```

## class

### gennely
```typescript=
class Car {
  descroption: string;

  constructor() {
    this.descroption = '我是車子';
  }

  public getDescription(): string {
    return this.descroption;
  }
}
```

###### tags: `Typescript`