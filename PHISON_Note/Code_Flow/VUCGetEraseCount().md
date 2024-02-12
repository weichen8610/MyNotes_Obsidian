#### Input
* `void`
#### Purpose
* 主要是用來將 DBT Block 中的特定 Plane ( 取決於 `ulLCA` ) 讀取到`ulBufferAdr`中
* `ubReadMode`用來決定是 `USE_COP0` or `USE_MT`
* `ubWaitCOP0`決定跟 COP0 的交互方式
#### Code Flow
* 利用 [[VUCScanRUT()]]：
	* 讀取 DBT RUT_L1 和 RIT_L2 到 `gpRUTL1` 和 `gpulRUT_L2` 上
* 利用 [[SysAreaRead4KEntry()_TBD]]：
	* 讀 `gSystemArea.SystemBlock[0]` 到 <font color="#ff0000">SYSAREA_SCAN_RECORD_BUFFER</font> 上
	* 並利用 InfoBlock 的 Sector 6 (`pFWSetting`) 設定 <font color="#ffc000">VTmother</font> Blk 的 <font color="#92d050">VBRMP</font>：
		- 設定 `gpVT->VT.uwUnit[].B.uwUnit = pFWSetting->VBRMPforVT[].uwVBIndex`
		- 並設定與之對應的 `gpuwVBRMP[].B.ToRUT_Unit` 和 `gpuwVBRMP[].B.btIsSLCMode`
* 利用 [[FTLScanVT()_TBD]] 來讀 <font color="#f666d4">gpVT</font> 和 <font color="#f666d4">gpVTDBUF</font> 起來，
	  Load <font color="#ffc000">VTChild</font> Blk 和 <font color="#ffc000">InitInfo</font> Blk，並設定其 <font color="#92d050">VBRMP</font>：
	- `gpuwVBRMP[gpVT->VTChild.uwUnit.B.uwUnit].uwAll = gpVT->VTChildVB`
		- 還會做 [[FTLScanVTChild()_TBD]]，Check VTChild (？
	* `gpuwVBRMP[gpVT->InfoUnit.uwUnit.B.uwUnit].uwAll = gpVT->InitInfoVB`
* Load <font color="#ffc000">InitInfo</font> Blk 中的 EC (16KB) 出來：
	* 利用 [[M_GET_VCA_PLANE()]] 