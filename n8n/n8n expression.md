# n8n expression

## 抓取整個node result to json

```jsx
{{ JSON.stringify($node["YourNodeName"].json) }}

```

## write a function in expression

```jsx
{{ 
  (() => {
    const nodeData = $node["YourNodeName"].json;
    return JSON.stringify(nodeData);
  })() 
}}

```

## Replace Var

```java
const index = $('Code').first().json.idx;
$('Code').first().json.idx = index+1;
```