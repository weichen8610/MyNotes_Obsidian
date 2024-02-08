## System Category
### Mainframe Systems
* 意思是指專門在處理某一件事情的機器
#### Batch
* 指一次只能處理一條指定的電腦， CPU 一次做一個 Job ( One Job at a time )
* 特性：
	* 可靠性 和 安全性 很高
		* Reliability：不會因為某些 Component 壞掉，也不會直接整台 Crash
		* Security：入侵的機會比較小
	* No Interaction betwwen user & Jobs：
		* 在執行時，無法跟使用者做任何互動
	* I/O 的速度 << CPU 的速度 -> CPU Idle
* OS 的任務 ( OS Task )：
	* OS 不必做任何 Decision ( 因為一個 Job 進來，做完再等下一個 Job 從 I/O 進來 )
#### Multi-programming System ( Spooling )
* 為了要解決 CPU Idle 的問題，所以 Overlap I/O 和 CPU，
	  也就是在上一份 Job 被做完再做 I/O 時，馬上讓下一 個 Job 進 CPU
* 由於先前在做 I/O 時，CPU 必須去下指令或是監控才有辦法做完 I/O，
	  因此要導入 Spooling 的概念：
	* Spooling ( Simultaneous Periphrial Operation On-Line )：
		* I/O 在執行時不需要 CPU 的介入，
			  當 I/O 完成時再去通知 CPU 就好
			  【也就是 CPU 不需要去 Polling，只要接收 Interrupt 就好】
* 特性：
	* 只是解決了 CPU Idle 的問題
	* 還是 No Interaction ( between User & CPU )
	* 無法支援多個 User 同時使用 ( 必須要等上一位 User 使用完 CPU 才能用 )
* OS 的任務 ( OS Task )：
	* Memory managment：CPU 要去從 Memory 中 Load Job 進來
	* CPU Scheduling：CPU 要去選擇 Memory 中的哪些 Job，先後順序來執行
	* I/O System：CPU 要負責跟 I/O 端的小 Processor ( 就是 Controller ) 做溝通![[Multi-programing_JobScheduling.png]]
#### Time-sharing System ( Multi-tasking System )
* 為了要做到 Interaction between User & System
* 個人理解：
	* 加入了用 時間(Time) 來將單一 Job 做切割的概念，所以可以做到：
		1. Interaction：當 User 有指令進來時 ( 就是 I/O Interrupt )，
			   CPU 可以在極短時間內做完當下 切割完的 Job 片段，
			   並執行 Interupt，所以可以做到 User 與 CPU 的即時互動
		2. Mulitple User：由於每個 Job 可根據時間進行切割，
			   所以可以在做完該 User 的 一點點 Job 後，切換去執行另一個 User 的 Job，
			   以此類推就可以讓每一個 User 有 "同時" 在使用 CPU 的感覺
* CPU 可能在執行速度會相較前兩者曼一點點，但是其實程式是有很多時間在等 I/O 的，
	  所以 "利" ( User 看起來同時在使用 ) > "弊" ( 要一直在做 Switch Job )
* 當以下情形發生時，可做 Switch Job：
	1. Job Finish
	2. Waiting I/O
	3. Short period of time
* OS 的任務 ( OS Task )：
	* Virtual Memory：將 Job 在 Memory 和 Storage 之間做 Swap，來擴大 Memory 的空間
	* File System：為了更好的管理 Storage 裡面擺放資料的方式，必須根據檔案系統進行操作
	* Process Synchronaization & Deadlock：因為會同時執行多個 Job，
		  當不同的 Job 要用到相同 memory 的位置時，可能會因執行的先後順序產生非預期的結果，
		  所以必須維護好 Access memory 方式，以及避免程式之間互相等待，導致 DeadLock 的產生
