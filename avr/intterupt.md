參考資料：
 - avr-libc Interrupts api 說明文件  
   https://www.nongnu.org/avr-libc/user-manual/group__avr__interrupts.html  
 - What registers are used by the C compiler?  
   https://www.nongnu.org/avr-libc/user-manual/FAQ.html#faq_reg_usage  
   
 結論：
  1. avr進入ISR時，會由硬體關閉全域中斷(Global interrupts)。
  2. 因1.，故一般無法產生巢狀中斷的程式，若要使用巢狀中斷，需在ISR中使用ISR_NOBLOCK的attributes，以關閉1.之功能。
  3. 未使用，但已致能之中斷，請使用EMPTY_INTERRUPT來登記，ex:`EMPTY_INTERRUPT(ADC_vect);`

---
#### ISR(vector, attributes)  

avr-libc裡的介紹：  

    Introduces an interrupt handler function (interrupt service routine) that runs with global interrupts initially disabled by default with no attributes specified.  
    The attributes are optional and alter the behaviour and resultant generated code of the interrupt routine. Multiple attributes may be used for a single function, with a space seperating each attribute.  
    Valid attributes are ISR_BLOCK, ISR_NOBLOCK, ISR_NAKED and ISR_ALIASOF(vect).  
    vector must be one of the interrupt vector names that are valid for the particular MCU type.  
    
功能是登記中斷要做的事項。  
可用的attributes有 ISR_BLOCK, ISR_NOBLOCK, ISR_NAKED and ISR_ALIASOF(vect).  

attributes介紹：
1. ISR_BLOCK  
  在進入ISR時，由硬體關閉全域中斷(Global interrupts)。故在執行ISR中內容時，不會進入其他ISR。  
  這是預設值，`ISR(vector);`與`ISR(vector, ISR_BLOCK);`是一樣的。  

2. ISR_NOBLOCK  
  在進入ISR時，由編譯器加入指令sei，以打開全域中斷(Global interrupts)，並會打開中斷致能旗標(SREG)，使其他中斷可以發生。  
  故可以做出巢狀迴圈，但要注意stack overflows。  

3. ISR_NAKED
  一般ISR中，會有些起始動作(prologue)及結束動作(epilogue)，(ex:剛進入ISR時，會對SREG進行改寫的動作)。  
  若使用ISR_NAKED，就不會產生這一些動作，但使用者必須自己處理會影響到暫存器，以及跳出ISR的動作。  
  一般使用ISR_NAKED，會在ISR結尾加上reti()。  

4. ISR_ALIASOF(vect)
  將一個向量指向另一個向量。  
  Ex:   
  ``` c
  ISR(INT0_vect) {
      PORTB = 42;
  }
  
  ISR_ALIAS(INT1_vect, INT0_vect);
  ```
  
  ---
  衍生：為什麼中斷中不能使用printf
  參考資料：
 - C printf() in interrupt handler?  
   https://stackoverflow.com/questions/12704196/c-printf-in-interrupt-handler  
 - Why are malloc() and printf() said as non-reentrant?  
   https://stackoverflow.com/questions/3941271/why-are-malloc-and-printf-said-as-non-reentrant.  
 - isr不能用printf  
   https://blog.csdn.net/hdu1114/article/details/6704295  
