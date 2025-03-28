---
title: Angular 常用筆記
tags: [javascript, Angular]

---

# Angular 常用筆記

## css class 

1. 
```htmlmixed=
[class.my_class] = "step === 'step1'"
```

2. 
```htmlmixed=
[ngClass]="{'my_class': step === 'step1'}"
```

3. and multiple option:
```htmlmixed=
[ngClass]="{'my_class': step === 'step1', 'my_class2' : step === 'step2' }"
```
4. 
```htmlmixed=
[ngClass]="{1 : 'my_class1', 2 : 'my_class2', 3 : 'my_class4'}[step]"
```


5. 
```htmlmixed=
[ngClass]="step == 'step1' ? 'my_class1' : 'my_class2'"
```


###### tags: `javascript` `Angular`