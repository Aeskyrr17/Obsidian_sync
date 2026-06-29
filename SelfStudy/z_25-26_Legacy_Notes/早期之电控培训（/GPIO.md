---

---
### GPIO输出模式HAL_GPIO_WritePin

- 推挽输出（用芯片内置3.3V）
    - P-MOS 连（VDD）& N-MOS（连VSS） 
    - 只能输出3.3V
- 开漏输出（用于外界5V高电平输出）
    - N-MOS
    - 无驱动模式
- 复用推挽输出
- 复用开漏输出

### GPIO输入模式

- 浮空
- 上拉（高）
- 下拉（低）


- HAL_GPIO_READPIN

读取当前电平。高GPIO_PIN_SET/低GPIO_PIN_REST

- HAL_GPIO_WritePin
- HAL_GPIO_TogglePin

翻转电平

