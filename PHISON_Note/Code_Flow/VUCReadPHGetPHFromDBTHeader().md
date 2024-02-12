#### Input
* `ulBufAdr`
#### Purpose
* 主要是利用來 DBT Header 中記的 PH Block 位置，判斷 2 個 PH Blk 的狀態
#### Code Flow
* `ubResult = `<font color="#ffc000">PH_COOSE_THE_SAME_ONE</font>
	* 2 個 PH Block 都不是 <font color="#ff0000">INVALID_BLOCK</font>，但是位址相同：
		  `pDBTHeader->PH[0].uwAll == pDBTHeader->PH[1].uwAll`
- `ubResult = `<font color="#ffc000">PASS</font>
	- `gPH.Info.B.btPHExist = TRUE`
		  要 2 個 PH Block 都存在 且 位址不同 才會舉 ( 不是 <font color="#ff0000">INVALID_BLOCK</font> )
	- `gPH.PHBlk[] = pDBTHeader->PH[]`，將 PH Block 的位址記錄下來
* `ubResult = `<font color="#ffc000">PH_NOT_EXIST</font>
	* 2 個 PH Block 都是 <font color="#ff0000">INVALID_BLOCK</font>
* `ubResult = `<font color="#ffc000">PH_LOST</font>
	* 當 `FALSE == pDBTHeader->ControlFlag.B.btOnlyScannedOnePH`
		  代表只有 1 個 PH Block 存在，另一個已經丟失了
- return `ubResult`