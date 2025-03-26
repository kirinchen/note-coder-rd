

JavaFX does not have a direct equivalent to Android's `LinearLayout` with `layout_weight`. However, you can achieve similar behavior using **JavaFX layout managers**, such as `HBox` or `VBox`, along with the `HBox.setHgrow()` or `VBox.setVgrow()` methods and percentage-based calculations.

### Example: Using `HBox` and `VBox` for Weight-Like Behavior

Here's how you can achieve layout proportions similar to `layout_weight`:

```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.layout.HBox;
import javafx.scene.layout.Priority;
import javafx.stage.Stage;

public class WeightedLayoutExample extends Application {

    @Override
    public void start(Stage primaryStage) {
        // Create an HBox to hold the buttons
        HBox hbox = new HBox();
        hbox.setPrefWidth(500); // Set a preferred width for the HBox (like weightSum)

        // Button 1 (weight 1)
        Button btn1 = new Button("1");
        btn1.setMaxWidth(Double.MAX_VALUE); // Allow resizing
        HBox.setHgrow(btn1, Priority.ALWAYS); // Make the button grow
        btn1.prefWidthProperty().bind(hbox.widthProperty().multiply(1.0 / 5)); // Weight = 1

        // Button 2 (weight 3)
        Button btn2 = new Button("2");
        btn2.setMaxWidth(Double.MAX_VALUE); // Allow resizing
        HBox.setHgrow(btn2, Priority.ALWAYS); // Make the button grow
        btn2.prefWidthProperty().bind(hbox.widthProperty().multiply(3.0 / 5)); // Weight = 3

        // Button 3 (weight 1)
        Button btn3 = new Button("3");
        btn3.setMaxWidth(Double.MAX_VALUE); // Allow resizing
        HBox.setHgrow(btn3, Priority.ALWAYS); // Make the button grow
        btn3.prefWidthProperty().bind(hbox.widthProperty().multiply(1.0 / 5)); // Weight = 1

        // Add buttons to the HBox
        hbox.getChildren().addAll(btn1, btn2, btn3);

        // Create the scene and set the stage
        Scene scene = new Scene(hbox, 500, 100);
        primaryStage.setScene(scene);
        primaryStage.setTitle("Weighted Layout Example");
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}

```

---

### Key Points in the Example

1. **`HBox` as the Container**:
    - Used for horizontal layout, similar to Android's `LinearLayout`.
2. **Setting Weight**:
    - The `prefWidthProperty().bind()` method dynamically adjusts each button's width based on the total width of the `HBox` and the desired proportion.
    - The formula `hbox.widthProperty().multiply(weight / totalWeight)` emulates `layout_weight`.
3. **Resizable Behavior**:
    - `HBox.setHgrow(node, Priority.ALWAYS)` ensures that the nodes grow and shrink proportionally when the window is resized.
    - `setMaxWidth(Double.MAX_VALUE)` allows nodes to resize fully.

---

### Example Output

If the total width of the `HBox` is `500px` and the weights are `1`, `3`, and `1`:

- Button 1: `500 * (1/5) = 100px`
- Button 2: `500 * (3/5) = 300px`
- Button 3: `500 * (1/5) = 100px`

---

### For Vertical Layouts

You can use `VBox` instead of `HBox` for vertical layouts. The logic remains the same, but you'll use `VBox.setVgrow()` and `prefHeightProperty().bind()`.

---

### Alternative: `GridPane`

If you want more control over the layout, you can use `GridPane` with percentage-based column or row constraints.

```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.layout.ColumnConstraints;
import javafx.scene.layout.GridPane;
import javafx.stage.Stage;

public class GridPaneExample extends Application {

    @Override
    public void start(Stage primaryStage) {
        GridPane grid = new GridPane();

        // Column constraints (weights)
        ColumnConstraints col1 = new ColumnConstraints();
        col1.setPercentWidth(20); // Weight = 1
        ColumnConstraints col2 = new ColumnConstraints();
        col2.setPercentWidth(60); // Weight = 3
        ColumnConstraints col3 = new ColumnConstraints();
        col3.setPercentWidth(20); // Weight = 1

        grid.getColumnConstraints().addAll(col1, col2, col3);

        // Add buttons to the grid
        grid.add(new Button("1"), 0, 0);
        grid.add(new Button("2"), 1, 0);
        grid.add(new Button("3"), 2, 0);

        // Create the scene and set the stage
        Scene scene = new Scene(grid, 500, 100);
        primaryStage.setScene(scene);
        primaryStage.setTitle("GridPane Example");
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}

```

---

### Key Takeaways

- JavaFX does not have a `layout_weight` attribute like Android, but you can achieve similar behavior with `HBox`, `VBox`, or `GridPane`.
- Use `prefWidthProperty().bind()` for dynamic proportions or `GridPane` with `ColumnConstraints`/`RowConstraints` for more structured layouts.