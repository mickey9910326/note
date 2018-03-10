
今天在寫avr的bootloader時，遇到了神奇的BUG  
專案：https://github.com/nuclear-refugee/bootloader

一開始.text section開始位置設定在 0xF000 (word memory)，
.text section + .data section共使用了約3800bytes的大小。
而使用的mcu ATMEGA128的BOOTSZ設定有4KB的bootloader大小。

在進行測試的時候發現，只要呼叫到函式A，就會出現問題。

猜測是return回0x0000的記憶體位置了。所以寫了一小段程式從uart傳送一筆資料出來，
並放到0x0000的記憶體位置，果不其然，只要呼叫到函式A，就會自動跳回0x0000，進一步測試，
發現有一大部分的函式都有相同的問題。

猜測是.data section在連結的時候出現了問題，造成呼叫函式時無法到達正確的記憶體位置。
(這部分純屬猜測，因技術能力不足QQ)

可是只要將程式的.text section搬移到0x0000程式就可以如期地進行，但因為是要做bootloader
所以搬移到0x0000並無法解決問題。

之後嘗試了將程式縮到3000bytes，或把變數全部給初始值，讓data中的.bss為0，都無法使程式正確
的運行。

於是，開始覺得是編譯器的問題，把萬惡的 **WinAVR** 替換成之前自己用mingw7.2編譯的ar-gcc
程式就可以如期的運行了。到此，也花了5個小時的時間，所以說，真的要少用停止維護，或社群小的
軟體。
