---
id: view-style-props
title: View Style Props
---

### 例

```SnackPlayer name=ViewStyleProps
import React from "react";
import { View, StyleSheet } from "react-native";

const ViewStyleProps = () => {
    return (
      <View style={styles.container}>
        <View style={styles.top} />
        <View style={styles.middle} />
        <View style={styles.bottom} />
      </View>
    );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "space-between",
    backgroundColor: "#fff",
    padding: 20,
    margin: 10,
  },
  top: {
    flex: 0.3,
    backgroundColor: "grey",
    borderWidth: 5,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
  },
  middle: {
    flex: 0.3,
    backgroundColor: "beige",
    borderWidth: 5,
  },
  bottom: {
    flex: 0.3,
    backgroundColor: "pink",
    borderWidth: 5,
    borderBottomLeftRadius: 20,
    borderBottomRightRadius: 20,
  },
});

export default ViewStyleProps;
```

# リファレンス

## Props

### `backfaceVisibility`

| 型                            |
| ----------------------------- |
| enum(`'visible'`, `'hidden'`) |

---

### `backgroundColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomEndRadius`

| 型     |
| ------ |
| number |

---

### `borderBottomLeftRadius`

| 型     |
| ------ |
| number |

---

### `borderBottomRightRadius`

| 型     |
| ------ |
| number |

---

### `borderBottomStartRadius`

| 型     |
| ------ |
| number |

---

### `borderBottomWidth`

| 型     |
| ------ |
| number |

---

### `borderColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderEndColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftWidth`

| 型     |
| ------ |
| number |

---

### `borderRadius`
Border の見た目が変わらない場合 `overflow: 'hidden'` を一緒に使ってみてください。

| 型     |
| ------ |
| number |

---

### `borderRightColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderRightWidth`

| 型     |
| ------ |
| number |

---

### `borderStartColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderStyle`

| 型                                      |
| --------------------------------------- |
| enum(`'solid'`, `'dotted'`, `'dashed'`) |

---

### `borderTopColor`

| 型                 |
| ------------------ |
| [color](colors.md) |

---

### `borderTopEndRadius`

| 型     |
| ------ |
| number |

---

### `borderTopLeftRadius`

| 型     |
| ------ |
| number |

---

### `borderTopRightRadius`

| 型     |
| ------ |
| number |

---

### `borderTopStartRadius`

| 型     |
| ------ |
| number |

---

### `borderTopWidth`

| 型     |
| ------ |
| number |

---

### `borderWidth`

| 型     |
| ------ |
| number |

---

### `elevation` <div class="label android">Android</div>
View の高さを Android の[elevation API](https://developer.android.com/training/material/shadows-clipping.html#Elevation)を通して指定できます。 Android 5.0+のみのサポートなので、それ以前のバージョンでは変化が出ません。

| 型     |
| ------ |
| number |

---

### `opacity`

| 型     |
| ------ |
| number |
