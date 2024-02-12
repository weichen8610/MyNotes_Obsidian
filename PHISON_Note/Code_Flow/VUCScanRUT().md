#### Input
* `void`
#### Purpose
* 主要是利用 `gSystemArea.DBTBlock[]` 紀錄的 DBT Block 位址，
	  讀 RUT L1 & L2 上來到 `gpRUTL1` 和 `gpulRUT_L2` 上
#### Code Flow
* `ubRUTLayer1TotalPlane`：
	  用 [[M_SIZE_BYTE_TO_PLANE()]] RUT_L1_LEN，確認要讀多少個 Page 上來
- `ubRUTLayer2TotalPlane`：
	  用 [[M_SIZE_BYTE_TO_PLANE()]] RUT_L2_LEN，確認要讀多少個 Page 上來
* 執行 [[COP0_RUTSetup()_TBD]]
* 利用 [[DBTReadPlane()]] 讀 RUT_L1 和 RUT_L2 到 `gpRUTL1` 和 `gpulRUT_L2` 上
* return <font color="#ffc000">PASS</font>