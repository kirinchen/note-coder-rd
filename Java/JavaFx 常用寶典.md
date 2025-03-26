

# 如何撐滿父層

```java
        VBox root = new VBox(canvasView);
        VBox.setVgrow(canvasView, javafx.scene.layout.Priority.ALWAYS);

```

## 在畫面開始後的Event(lifeCicry)

```java
 Platform.runLater(()->{
 });

```

# `Observable` 相關

## ObservableValue

```java
    public static class Data{
        public boolean zomLocVisible = true;
    }

    public final ObservableValue<Data> observer = new SimpleObjectProperty<Data>();
```

## Bindings

```java
SimpleIntegerProperty count = new SimpleIntegerProperty(5);

// Computed state (like useMemo)
var doubleCount = Bindings.createIntegerBinding(() -> count.get() * 2, count);
```