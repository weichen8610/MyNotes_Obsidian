#### Input
* `UNIT`
* `PAGE`
#### Purpose
* 主要是利用 Input 算出該 `UNIT` 和 `PAGE` 的 Entry Index = PCA
* 算法：`UNIT` * ( Entry 數 per Unit ) + `PAGE` * ( Entry 數 per Page )
#### Code Flow
- `(((UNIT) << gub4kEntrysPerUnitAlignLog) + ((PAGE) << gub4kEntryPerPlaneLog))`