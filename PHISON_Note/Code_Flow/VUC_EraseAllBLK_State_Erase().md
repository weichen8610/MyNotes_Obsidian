#### Input
* `ubSLCMode`
* `ubSelectMode`
* `uwStartBlockIndex`
* `uwCount`
#### Purpose
* 用來 Erase Block，Block Index：`uwStartBlockIndex` ~ ( `uwStartBlockIndex + uwCount`)
* 並根據 Erase Mode 決定：
	* 是否要 Erase DBT Block
	* 是否要更新 DBT ( 要的話，是否要 Clear Later Bad )
	* 是否要略過，不 Erase Bad Block
* 最後若有 Error 的 PCA，就進行 Mark Bad
#### Code Flow
* `ubBadType` 隨時都代表該 Block 的 BadType
* `ubErase` 表示該 Block 是否要做 Erase
* -
* 進 for Loop，對每一個 CH / CE / Die / Plane
	的 BlockIndex =  `uwStartBlockIndex` ~ ( `uwStartBlockIndex + uwCount`)：
	- Set `ubErase = 1`
	- 先用 [[M_GET_DBT_BAD_TYPE()_TBD]] 確認該 Block 的 BadType ，
		  並存於 `ubBadType`中， 若該 Block 已經是 EarlyBad，`ubErase = 0`
	- if【`FALSE == gEraseAll.ubForceErase`】：
		  意即 Erase Mode 不是 <font color="#ffff00">VUC_ERASE_ALL_FORCE_ERASE_ALL</font> ( 0xFE )，
		  就確認該 Block 若是 PH Block，`ubErase = 0`
	- if【`gEraseAll.Others.B.btDBTExist`】，意即若 DBT Block 存在：
		- if【沒舉 <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 )】：
			- 就確認該 Block 若是 DBT Block，`ubErase = 0`
	- if【有舉 <font color="#ff0000">VUC_ERASE_ALL_UPDATE_DBT</font> ( BIT4 )】：
		- if【`DBT_FW_EARLY_BAD < ubBadType`】：
			- if【有舉 <font color="#ff0000">VUC_ERASE_ALL_CLEAR_LATER_BAD</font> ( BIT5 )】：
				- Set `ubBadType = DBT_GOOD_BLOCK`
				- 並利用 [[M_DBT_CLEAR_ENTRY()_TBD]] 清除 DBT 上的 Bad Type
			- else：
				- 根據原先的 `ubBadType`，
					  Set `ubBadType = DBT_GOOD_BLOCK or DBT_FW_EARLY_BAD`
				- 並利用 [[M_DBT_CLEAR_ENTRY()_TBD]] 清除 DBT 上的 Bad Type
			- 利用 [[M_SET_DBT_BAD_TYPE()_TBD]] ：
				  設定 DBT 上的 Bad Type =`ubBadType`
	- if【有舉 <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 )】：
		- 若 `ubBadType` 是 LaterBad 的話，`ubErase = 0`
		- <font color="#f79646">但是 System Area 若有 BadBlock 還是要做 Erase</font>：
			  若 Block 是 Code / System / DBT Block 的話，`ubErase = 1`
	- if【`ubErase`】：
		- 利用 [[COP0API_SendEraseSQ()_TBD]] 來 Erase 該 Block
- 最後利用 [[DBTMarkBad()_TBD]] ：
	  透過 `gpErrorPCARecord->ulErrorPCA[]`來 Mark <font color="#ff0000">DBT_RDT_EARLY_BAD</font>
- Set `gubLeavePreformatFlag = 1`
- return `void`