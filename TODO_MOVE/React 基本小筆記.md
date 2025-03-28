---
title: React 基本小筆記
tags: [React, TypeScript]

---

# React 基本小筆記 

## 指令

### 創建 react typescript project

```shell
 npx create-react-app my-app --template typescript
```

## vs code config

## Run Debug
* 先安裝 codemooseus.vscode-devtools-for-chrome
![](https://hackmd.io/_uploads/BkNb-t_V2.png)
* 點選 create a launch.json
* 編輯launch.json
```json=
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "chrome",
            "request": "launch",
            "name": "Launch Chrome against localhost",
            "url": "http://localhost:3000",
            "webRoot": "${workspaceFolder}"
        }
    ]
}
```
* 在 console ``` npm start ```
* F5 play!!!

## JSX

### jsx function example
```typescript!
const TestRow = (props: any): JSX.Element => (
  <div>
    xxxx {props.name}
    <button onClick={testEvent}> oooo </button>
    <img src={require('./download.png')}></img>
    <img src={logo} width={50} height = {50} ></img>
  </div>)
```

```typescript!
const FragmentTest = (props: any): JSX.Element => {
  console.log(props);
  return <Fragment>
    <li> dd </li>
    <li> cc </li>
    <li> vv </li>
  </Fragment>
}
```

### list 渲染 - Array
```typescript!
  const vals = [
    <li key={1}> s- </li>,
    <li key={2}> b* </li>,
    <li key={3}> x1 </li>,
  ];
  return <>
    {vals}
  </>
```
> !**important** 記得要加 **key**


### props 常用

#### props.children
> 取得標籤內部的 jsx

ex : 取得內部的 jsx 的 attribute 屬性
```typescript!
console.log(props.children[0].props.testTag);
```

## hook

### useEffect use await example

```typescript=
import React, { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // Define an async function inside the useEffect
    const fetchData = async () => {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
    };
    // Call the async function
    fetchData();
  }, []); // Dependencies array, ensure to add variables that might affect the fetch operation

  return <div>Check the console for the fetched data.</div>;
}

export default MyComponent;

```

###### tags: `React` `TypeScript`