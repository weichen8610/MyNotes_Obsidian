#### Input
* `ubDirectEraseDBTCase`
#### Purpose
* 主要是用來計算 D1 / D3 Unit 在 reduce Unit 以外的 Unit，平均的 EC per Unit
#### Code Flow
* 若 `gEraseAll.ubMode` 的 <font color="#ff0000">VUC_ERASE_ALL_CLEAR_ERASE_COUNT</font>  ( BIT6 )
	  或 <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 ) 有舉，
	  或 `ubDirectEraseDBTCase == TRUE` 的話：
	* 先計算之前 D1 / D3 Unit 平均的 EC：
		* `gpVT->ulPreviousEC`+= [[WLGetECAverageInline()]] ( D3 Unit's Average EC )
		* `gpVT->ulPreviousD1EC`+= [[WLGetD1ECAverageInline()]] ( D1 Unit's Average EC )
	* 清空 `gpulEC` 中所有 Unit 的 EC 為 0
		* 每個 Unit 的 EC 用 4 個 Byte 紀錄，最多有 4096 ( <font color="#ffc000">MAX_UNIT</font> ) 個 Unit
	* 將先前 D1 / D3 Unit 平均的 EC 紀錄於 `gPreformat` 中：
		* `gPreformat.ulPreviousEC = gpVT->ulPreviousEC`
		* `gPreformat.ulPreviousD1EC = gpVT->ulPreviousD1EC`
* 若 FALSE，則該 Function do nothing