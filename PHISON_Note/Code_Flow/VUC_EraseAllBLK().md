#### Code Flow
1. `ERASEALL_INIT`
2. `ERASEALL_SEARCH_DBT_CODE`
3. `ERASEALL_CHECK_EARLYBAD_BLK`
4. `ERASEALL_ERASE`
5. `ERASEALL_BUILD_NEW_BAD_TABLE`
6. `ERASEALL_FINISH`
#### Erase Type ( `gEraseAll.uwEraseMode` )
- <font color="#ff0000">VUC_ERASE_ALL_ERASE_ALL</font> ( BIT0 )
	- 在 `ERASEALL_ERASE` 中才會做 Erase Block，否則所有 Block 全部都不做 Erase
- <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 )
	- 在 `ERASEALL_SEARCH_DBT_CODE` 時，會去相信從 Flash 中讀上來的 DBT Block
	- 在 `ERASEALL_ERASE` 時，若該 Block 是 LaterBad 就不會去做 Erase
		  【<font color="#f79646">但 System Area 的 Block 都還是會做 Erase</font>】
- <font color="#ff0000">VUC_ERASE_ALL_SCANEARLY_BAD</font> ( BIT2 )
	- 目前 Code 上無作用，有沒有舉不影響 Code Flow
- <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 )
	- 在 `ERASEALL_SEARCH_DBT_CODE` 時，
		  把 DBT Block 讀上來之後，會直接去 Erase Flash 上的 DBT Block
	- 在 `ERASEALL_ERASE` 時，會去 Erase DBT Block
- <font color="#ff0000">VUC_ERASE_ALL_UPDATE_DBT</font> ( BIT4 )
- <font color="#ff0000">VUC_ERASE_ALL_CLEAR_LATER_BAD</font> ( BIT5 )
- <font color="#ff0000">VUC_ERASE_ALL_CLEAR_ERASE_COUNT</font> ( BIT6 )
- <font color="#ff0000">VUC_ERASE_ALL_SKIP_SCAN_SYSAREA</font> ( BIT7 )
	- 有舉，則不做 `ERASEALL_SEARCH_DBT_CODE`，直接重建 DBT Block
- -
- <font color="#ffff00">VUC_ERASE_ALL_FORCE_ERASE_ALL</font> ( 0xFE )
	- 有舉，`ERASEALL_INIT` -> `ERASEALL_ERASE`
- -
- 特殊的交互作用：
	- 在 <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 ) 沒舉，
		  但 <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 ) 有舉的情況下，
		  反而會在 `ERASEALL_ERASE` 時，去 Erase Flash 上的 DBT Block【？？
#### ERASEALL_INIT
* 主要在做 VUC_EraseAllBLK 的 Initial
* 判斷 `gEraseAll.uwEraseMode` 是否為 0xFE
	* 是，-> `ERASEALL_ERASE`
	* 否，-> `ERASEALL_SEARCH_DBT_CODE`
#### ERASEALL_SEARCH_DBT_CODE
* 主要在搜尋 Flash 內是否已經有 DBT Block
* -
* if【沒舉 <font color="#ff0000">VUC_ERASE_ALL_SKIP_SCAN_SYSAREA</font> ( BIT7 )】
	* 利用 [[SystemAreaScanBlock()_TBD]] 去掃 DBT Block，將結果存在`gSystemArea.ubDBTBlockNum`，
		  再存到`gEraseAll.ubDBTCount`中 ( 此時已做完 [[DBTCheckHeader()]] )
* if【`gEraseAll.ubDBTCount != 0`】：
	* 利用 [[COP0SystemReverseCop0CieInPCA()_TBD]] 和 [[FlaGetPCA()_TBD]] 將`gSystemArea.DBTBlock[]` 存的 <font color="#92d050">DBT PCA </font>位置存入`gEraseAll.ulDBTPCA[]`中
	* 利用 [[DBTReadPlane()]] 將 <font color="#f666d4">DBT Header</font> 讀到 <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 上
	* `gSystemArea.uoDBTRevision = pDBTHeader->uoDBTRevision`
	* 利用 [[VUCReadPHGetPHFromDBTHeader()]] 來判斷 2 個 PH Blk 的存在與否，
		  並記錄到 `gPH.PHBlk[]` 中
	- 利用 [[VucEraseAllHandleECTable()]] 來決定是否繼承 EC
	- `gEraseAll.Others.B.btDBTExist = TRUE`
	- 進到 for Loop 將 DBT 所有 Page 讀上來：
		- 利用 [[DBTReadPlane()]] 將 DBT 讀到 <font color="#ff0000">DBT_BASE_ADDR</font> 上
		- 若有 Read FAIL 發生：
			- [[VucEraseAllCleanECTable()]] (TRUE)：清除 `gpulEC[]` 為 0
			- [[VucEraseAllProgramECIntoPH()_TBD]]：將 EC Program 進 PH
			- [[SysAreaEraseBlock()_TBD]] (`DBTBlk`, NORMAL_SLC_MODE)：
				- Erase 整個 DBT Block，並 <font color="#f79646">return</font>
	- if【有舉 <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 ) 】：
		- 利用 [[VUC_EraseAllBLK_State_EraseOldBadTable()_TBD]]：
			- 將 Flash 上的 DBT Block ( 2 個 ) 都刪除掉
				  【此時 <font color="#ff0000">DBT_BASE_ADDR</font> 上的 DBT Block 還留著】
		- if【有舉 <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 ) 】：
			- 相信 <font color="#ff0000">DBT_BASE_ADDR</font> 上的 DBT Block
				  【也就是相信舊的 DBT Block！】
		- else：
			- 將 RAM 上的 DBT Blk 和 DBT Header 都清掉：
				- Erase <font color="#ff0000">DBT_BASE_ADDR</font> 
				- Erase <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 
		- `gEraseAll.Others.B.btDBTExist = FALSE`
- if【`gEraseAll.ubDBTCount == 0`】：
	- 代表沒有 DBT，則需要重建 DBT
	- 將 <font color="#ff0000">DBT_BASE_ADDR</font> 和 <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 清 0 
- -> `ERASEALL_CHECK_EARLYBAD_BLK`
#### ERASEALL_CHECK_EARLYBAD_BLK
- 利用 Erase 一整個 Block 的方式，來檢查該 Block 是否為 BadBlock
- `gEraseAll.uwEraseBlockIndex`：此 State 要檢查的第一個 Block Index
- `gEraseAll.uwBlockPerCycle`：每次進到此 State 會檢查幾個 Block
- -
- if【`FALSE == gEraseAAll.Others.B.btDBTExist`】：
	- Set `gubLeavePreformatFlag = 0`
	- 計算 `uwCount`：這次該 State 要檢查幾個 Block
	- Set `gubDoingBuildDBTFlag = 1`
	- 利用 [[DBTCheckEraseBad()]] (`gEraseAll.uwEraseBlockIndex`, `uwCount`, BIT1, 1)：
		  透過 <font color="#ffc000">Erase</font> / <font color="#ffc000">Non-Data Program</font> / <font color="#ffc000">Read</font> 的方式檢查是否為 Bad Block
	- Set `gubLeavePreformatFlag = 1`；Set `gubDoingBuildDBTFlag = 0`
- -> `ERASEALL_ERASE`
#### ERASEALL_ERASE
- 不是只為了檢察 EarlyBad，而是真的要做 Erase Block 了
- -
- 計算 `uwCount`：這次該 State 要 Erase 幾個 Block
- if 【<font color="#ff0000">VUC_ERASE_ALL_ERASE_ALL</font> ( BIT0 ) 有舉】：
	- Set `gubLeavePreformatFlag = 0`；Set `gubDoingBuildDBTFlag = 1`
	- 利用 [[VUC_EraseAllBLK_State_Erase()]] 
		  (`ubSLCMode`, `ubSelectMode`, `gEraseAll.uwEraseBlockIndex`, `uwCount`)：
		- 來對 Block 進行 Erase
	- Set `gubLeavePreformatFlag = 1`；Set `gubDoingBuildDBTFlag = 0`
- `gEraseAll.uwEraseBlockIndex += uwCount`
- if【`gEraseAll.uwEraseBlockIndex < BLOKCK_NUM`】：
	- if【`TRUE == gEraseAll.ubForceErase`】：
		  意即 Erase Mode 是 <font color="#ffff00">VUC_ERASE_ALL_FORCE_ERASE_ALL</font> ( 0xFE )，
		- -> `ERASEALL_ERASE` ( 一直做 Erase Block )
	- else：
		- -> `ERASEALL_CHECK_EARLYBAD_BLK` ( 回去檢查下幾個 Block 的 BadType )
- else：
	- -> `ERASEALL_BUILD_NEW_BAD_TABLE` ( 最後要重新 Program DBT Block 了 )
#### ERASEALL_BUILD_NEW_BAD_TABLE
* Program 新的 DBT Block 下去
* 此 State 要有舉 <font color="#ff0000">VUC_ERASE_ALL_UPDATE_DBT</font> ( BIT4 ) 才會有作用
* -
* 