---
title: sccs & module
tags: [HTML/CSS]

---

# sccs & module

## sccs 

### sccs 起手式
```css=
$defaultColor : #f66;

.wrap {
    width: 100%;
    height: 200px;
    background: $defaultColor;
    h1, h2,h3 {
        font-size : 36px;
        color: #66f;
    }
    border: {
        style:solid;
        width: 5px;
        color: #ccc;
    }
}
```

### Hello Module

## Module

### css module 起手式
 
```css=
.useColor{
    color: #ae3c3c;
}

.moduleClazzInit{
    background: #4cce2b;
    composes : wrap from './style.scss'
}

.moduleClazz{
    font-size: 50px;
    color-scheme : useColor;
    composes: moduleClazzInit;
}

:global(.globalDiv){
    font-size: 50px;
    
}
```

use it
```jsx!
import TestModule from './test.module.scss'
<div className={ TestModule.moduleClazz }>vvv</div>
<div className="globalDiv"> ff </div>
```

###### tags: `HTML/CSS`