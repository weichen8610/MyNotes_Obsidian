#### Input
* `uwStartBlockIndex`
* `uwCount`
* `ubEraseRead`
* `ubSetDBT`
#### Purpose
* 主要用來檢查 Block Index：`uwStartBlockIndex` ~ ( `uwStartBlockIndex + uwCount` )
	  是否為 EarlyBad，並做 Mark Bad 的動作。
* `ubEraseRead`：
	*  == BIT0：做 <font color="#ffc000">Erase</font> 來檢查 EarlyBad
	*  == BIT1：做 <font color="#ffc000">Non-Data Program</font> 或 <font color="#ffc000">Read</font> 來檢查 EarlyBad
* `ubSetDBT`：有舉才會做 Mark Bad
#### Code Flow
* if【`ubEraseRead & BIT0`】：
	* 進 for Loop，對每一個 CH / CE / Die / Plane 
		  的 BlockIndex =  `uwStartBlockIndex` ~ ( `uwStartBlockIndex + uwCount`)
		  利用 [[COP0API_SendEraseSQ()_TBD]] 做 Erase Block：
		- 若 Erase FAIL 利用 [[DBTMarkBad()_TBD]] Mark <font color="#ff0000">DBT_RDT_EARLY_BAD</font> 
- if【`ubEraseRead & BIT1`】：
	- 進 for Loop，對每一個 CH / CE / Die / Plane 
		  的 BlockIndex =  `uwStartBlockIndex` ~ ( `uwStartBlockIndex + uwCount`)：
		- 先用 [[M_GET_DBT_BAD_TYPE()_TBD]] 確認該 Block 是否已經是 EarlyBad，
			  並且確認不是 PH Block 才會做以下：
			- 根據不同的 <font color="#f666d4">Flash Type</font> 來決定要做 <font color="#ffc000">Non-Data Program</font> 或是 <font color="#ffc000">Read</font>：
			- 若是 `TOSHIBA_SLC/3D_TLC/3D_QLC / SANDISK_3D_TLC`：
				- 利用 [[COP0APU_SendNonDataSLCProgram()_TBD]] 
					  來做該 Block 的 <font color="#ffc000">Non-Data Program</font> 
			- 若是 `HYNIX_3D_TLC`：
				- 利用 [[DBTPIOReadCheckEarlyBadBlock()_TBD]] 
					  來做該 Block 的 <font color="#ffc000">Read</font> ( 透過 PIO CMD)
				- 再利用 [[M_SET_DBT_BAD_TYPE()_TBD]] 來 Mark Bad
			- 若是其他 Flash Type：
				- 利用 [[COP0SystemReverseCop0CieInPCA()_TBD]] 
					  來做該 Block 中 "First Page" & "Last Page" 的 <font color="#ffc000">Read</font> 
				- 再利用 [[DBTCheckEarlyBadMark()_TBD]]：
					  查看第一個 Page 的 Bad Mark 決定是否要 MarkBad
	- 若是 `TOSHIBA_SLC/3D_TLC/3D_QLC / SANDISK_3D_TLC`：
		- 會再根據 `ErrorPCARecord->ulErrorPCA[]` 來進行 [[DBTMarkBad()_TBD]] 