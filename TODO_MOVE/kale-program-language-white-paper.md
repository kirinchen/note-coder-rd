---
title: kale-program-language-white-paper

---

# kale-program-language-white-paper


## BASIC Syntactic 
this code syntax like css

```css=
ClazzName {
    @Inject
    width int: 100;
    @Required 
    @Inject(scaneRange=ScaneRange.Default);
    height: null;
    @Param
    @Required 
    background OtherClass: null;
    border: NewClass{
        style:solid;
        width: 5px;
        color: #ccc;
    }
    func1:<  // --A
        console.log('HI~');
    >
    $val: <
        return $clone($this);
    >
}

const = ClazzName(
    height = 50;
    background  = NewClass() ;
)
```

### Syntactic sugar

- A :
```css=
func1:{
    $val: <
        console.log('HI~');
        return undefined;
    >    
}
```


