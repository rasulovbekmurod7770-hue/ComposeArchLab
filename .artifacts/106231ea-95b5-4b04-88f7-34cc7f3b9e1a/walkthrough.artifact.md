# Walkthrough - Jetpack Compose Phases Practice Class

Complete solutions and hands-on implementations for all questions in the **Jetpack Compose Phases — Practice Class** (`uz.ttpu.framephases`).

## Part 1 & Theoretical Solutions (Questions 1–3, 8, 12)

### Question 1 — The Three Phases of a Frame
- **(a) Order & Outputs**:
  1. **Composition**: Outputs the UI tree (a hierarchical description of UI elements).
  2. **Layout**: Outputs measured and positioned layout nodes (sizes and (x, y) coordinates).
  3. **Drawing**: Outputs drawing commands onto the canvas to render pixels on screen.
- **(b) Layout Steps**: 1. Measure children (`measure`), 2. Place children (`place`).
- **(c) `LazyColumn` Exception**: `LazyColumn` needs to know item sizes and viewport constraints during layout before deciding which items to compose. Thus, child composition is deferred/dependent on the parent's layout pass. Example: a chat message list where item visibility depends on scroll offset determined during layout.
- **(d) Skipping Mechanism**: Compose uses memoization and skeletal recomposition skipping (tracking state reads via restart scopes) to skip unchanged composables and layout passes.

### Question 2 — Walk the Layout Algorithm
- **Tree**: Row (Image, Column (Text A, Text B))
- **(a) Measure/Place Order**:
  - Measure Image (48x48)
  - Measure Column (calls measure on Text A (100x20), Text B (80x20), combines them into Column size 100x40)
  - Decide Row size (148x48)
  - Place Row
  - Place Image at (0, 0)
  - Place Column at (48, 4)
  - Place Text A at (48, 4)
  - Place Text B at (48, 24)
- **(b) Sizes & Positions**:
  - Column size: 100 × 40 px.
  - Row size: 148 × 48 px.
  - Image: (0, 0).
  - Column: (48, 4).
  - Text A: (48, 4).
  - Text B: (48, 24).
- **(c) Visit Counts**: Each node is visited once during measure and once during place. If measured twice at each level, traversal time would grow exponentially ($O(2^n)$), destroying 60fps performance.
- **(d) Tree Generation**: Produced by Composition phase; contains `LayoutNode` objects holding modifiers, measurement constraints, and placement data.

### Question 3 — State Reads & Restart Scopes
- **(a)** Snippet 1 reads `.value` inside composition; Snippet 2 uses property delegates which also read `.value` during composition when the modifier/property is accessed. Both are evaluated when building the modifier in composition.
- **(b)** For Text("A") (`.offset { ... }`), `offsetX` is read during the **layout phase** (placement lambda). When `offsetX` changes, only the layout phase (and drawing) re-runs, skipping composition. For Text("B"), `offsetPx` is read in **composition**, so when `offsetX` changes, composition re-runs.
- **(c)** A restart scope is a block of code tracked by Compose snapshot state observation that can be re-executed when state changes. For a read directly inside a composable function, the restart scope is the composable function itself.

### Question 8 — Which Phase Re-runs?
- **(a) `Modifier.padding(p)`**: Read in **Composition**. Changes run: Composition (runs), Layout (runs), Drawing (runs).
- **(b) `Modifier.offset { IntOffset(p, 0) }`**: Read in **Layout (Placement)**. Changes run: Composition (skipped), Layout (runs - placement only), Drawing (runs).
- **(c) `Modifier.offset(p)`**: Read in **Composition** (modifier creation). Changes run: Composition (runs), Layout (runs), Drawing (runs).
- **(d) `Canvas(...) { drawRect(c) }`**: Read in **Drawing**. Changes run: Composition (skipped), Layout (skipped), Drawing (runs).
- **(e) `Layout(...) { m, cs -> ... s ... }`**: Read in **Layout (Measure)**. Changes run: Composition (skipped), Layout (runs - measure/place), Drawing (runs).
- **(f) `Modifier.drawWithContent { ... c ... }`**: Read in **Drawing**. Changes run: Composition (skipped), Layout (skipped), Drawing (runs).
- *Choice for 60fps animation*: **(b)** or **(d)** (lambda offset or draw block) because they skip composition and heavy layout measurement entirely, avoiding jank.

### Question 12 — Two Frames for One Layout
- **Frame 1**: `imageHeightPx` is 0 initially. Composition reads 0, text top padding is 0. Layout phase runs `onSizeChanged`, sets `imageHeightPx = 360`, scheduling a recomposition. Text renders incorrectly at top padding 0 on Frame 1.
- **Frame 2**: Composition runs with `imageHeightPx = 360`, layout runs with correct 360px padding.
- **(a) Warning Sign**: Converting pixel measurements (`toDp()`) inside composition or state is a warning sign because pixels depend on layout device density and size, which aren't known until layout.
- **(b) Pattern**: Read-during-layout / cyclic phase dependency pattern.
- **(c) Broken Principle**: Unidirectional data flow (state should drive layout, not layout updating state that re-triggers composition in a loop).

---

## Part 2, 3, & 4: Hands-On Code Components

All demo files implemented under package `uz.ttpu.framephases`:
- [PhaseLog.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/PhaseLog.kt) — `Modifier.logPhases(name)` reporting measurement, placement, and draw (`tag:Phases`).
- [LectureTree.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/LectureTree.kt) — Question 4 UI tree with logs.
- [PaddingDemo.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/PaddingDemo.kt) — Question 5 Composition phase read demo (`tag:Phases`).
- [OffsetDemo.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/OffsetDemo.kt) — Question 6 Layout phase lambda read demo (`tag:Phases`).
- [ColorDemo.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/ColorDemo.kt) — Question 7 Drawing phase read demo (`tag:Phases`).
- [ParallaxSlow.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/ParallaxSlow.kt) & [ParallaxFast.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/ParallaxFast.kt) — Question 9 Parallax scroll optimization (`tag:Parallax`).
- [ScrollToTopNaive.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/ScrollToTopNaive.kt) — Question 10 `derivedStateOf` optimization (`tag:Derived`).
- [Parent.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/Parent.kt), [Child.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/Parent.kt), [Sibling.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/Parent.kt) — Question 11 Restart scope skipping via lambda reads (`tag:Skip`).
- [CyclicDemo.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/CyclicDemo.kt) & [CyclicFixed.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/CyclicFixed.kt) — Question 13 Cyclic loop vs single source of truth layout (`tag:Cycle`).
- [StackBelow.kt](file:///C:/Users/Lab/AndroidStudioProjects/ComposeArchLab/app/src/main/java/uz/ttpu/framephases/StackBelow.kt) — Question 14 Custom layout implementation using the 3-step measure/size/place algorithm (`tag:StackBelow`).

## Verification Results

### Build Status
- Ran `app:assembleDebug`:
  ```
  BUILD SUCCESSFUL
  ```
