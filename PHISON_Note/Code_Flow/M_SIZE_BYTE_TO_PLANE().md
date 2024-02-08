#### Input
* `BYTE_NUM`
#### Purpose
* 利用`BYTE_NUM` 對 16 KB 做 DIV_CEILING，來計算出需要用到多少個 Page
#### Code Flow
* (`BYTE_NUM` + 16KB  - 1) / ( 16KB )