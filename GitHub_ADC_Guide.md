# راهنمای گام‌به‌گام: GitHub + ADC/Potentiometer + UART
## برای پروژه UPS 2kW

> امیرحسین عزیز — این فایل رو ذخیره کن و مرحله به مرحله جلو برو.
> هر مرحله که تموم شد، یه تیک بزن (✅).
> اگه جایی گیر کردی، برگرد و بگو کدوم مرحله.

---

## بخش اول: GitHub — از صفر تا اولین Push

### هدف
یه ریپوی GitHub بسازی، کدهات رو بذاری توش، و هر روز آپدیت کنی.

### مرحله ۱: نصب Git (اگه نصب نیست)

1. برو به https://git-scm.com/download/win
2. دانلود و نصب کن (Next, Next, Next... OK)
3. توی CMD یا PowerShell بنویس:
   ```bash
   git --version
   ```
4. اگه نسخه رو نشون داد (مثلاً `git version 2.43.0`) → ✅ نصب شده

### مرحله ۲: تنظیم Git (فقط یک بار)

توی CMD یا Git Bash بنویس (ایمیل و نام خودت رو بذار):

```bash
git config --global user.name "AmirHossein"
git config --global user.email "your-email@example.com"
```

### مرحله ۳: ساخت ریپو توی GitHub

1. برو به https://github.com و لاگین کن
2. روی دکمه سبز **New** (بالا سمت راست) کلیک کن
3. **Repository name** بنویس: `UPS-2kW-LineInteractive`
4. **Description** بنویس: `Line-Interactive Pure Sinewave UPS - 2kW - STM32F303RE`
5. **Public** رو انتخاب کن (رایگانه و بقیه هم می‌تونن ببینن)
6. تیک **Add a README file** رو بزن
7. **Create repository** بزن

### مرحله ۴: Clone کردن ریپو روی کامپیوتر

1. توی صفحه ریپوی GitHubت، دکمه سبز **<> Code** رو بزن
2. لینک HTTPS رو کپی کن (مثلاً: `https://github.com/amirhossein/UPS-2kW-LineInteractive.git`)
3. توی CMD یا Git Bash، برو به پوشه‌ای که می‌خوای پروژه رو نگه داری:
   ```bash
   cd C:\Users\YourName\Documents
   ```
4. بنویس:
   ```bash
   git clone https://github.com/amirhossein/UPS-2kW-LineInteractive.git
   ```
5. یه پوشه جدید به اسم `UPS-2kW-LineInteractive` ساخته می‌شه → ✅

### مرحله ۵: اولین Commit و Push

1. برو توی پوشه پروژه:
   ```bash
   cd UPS-2kW-LineInteractive
   ```
2. یه فایل جدید بساز (مثلاً یه فایل متنی):
   ```bash
   echo "Project Started!" > notes.txt
   ```
3. فایل رو به Git اضافه کن:
   ```bash
   git add notes.txt
   ```
4. Commit کن (یه پیام معنی‌دار بذار):
   ```bash
   git commit -m "Initial commit - project start"
   ```
5. Push کن به GitHub:
   ```bash
   git push origin main
   ```
6. برو به صفحه GitHub ریپوت و refresh کن — فایل `notes.txt` باید اونجا باشه! → ✅

### مرحله ۶: روال روزانه (هر روز اینو تکرار کن)

```bash
# ۱. برو توی پوشه پروژه
cd C:\Users\YourName\Documents\UPS-2kW-LineInteractive

# ۲. فایل‌های جدید یا تغییر یافته رو اضافه کن
git add .

# ۳. Commit کن (پیام معنی‌دار!)
git commit -m "Day 1 - LED blink working on Nucleo-F303RE"

# ۴. Push کن به GitHub
git push origin main
```

### نکات طلایی Git

| دستور | کاربرد |
|-------|--------|
| `git status` | ببین چه فایل‌هایی تغییر کردن |
| `git log` | تاریخچه کامیت‌ها |
| `git diff` | ببین چی تغییر کرده |
| `git pull` | اگه روی یه کامپیوتر دیگه کار کردی، تغییرات رو بگیر |

> **قانون:** هر روز حداقل یه Commit. حتی اگه یه خط کد عوض کردی.

---

## بخش دوم: ADC + Potentiometer + UART — گام‌به‌گام

### هدف
یه پتانسیومتر به Nucleo وصل کنی، مقدارش رو با ADC بخونی، و روی کامپیوتر از طریق UART نشون بدی.

### مرحله ۱: مدار سخت‌افزاری

```
         +3.3V (Nucleo pin CN7-16)
           |
          [Potentiometer 10k]
           |
           +-----> PA0 (Nucleo pin CN7-28)
           |
          GND (Nucleo pin CN7-29)
```

**اتصال‌ها:**
- پایه وسطی پتانسیومتر → **PA0** (CN7-28 روی Nucleo)
- یک پایه کناری → **3.3V** (CN7-16)
- پایه دیگه → **GND** (CN7-29)

> ⚠️ **مهم:** ولتاژ بیشتر از 3.3V به PA0 نده! ADC می‌سوزه.

### مرحله ۲: کانفیگ STM32CubeMX

1. پروژه جدید بساز (مثل دیروز) یا همون پروژه LED Blink رو باز کن
2. توی Pinout View، پین **PA0** پیدا کن
3. روش کلیک کن → **ADC1_IN1** انتخاب کن
4. تب **Analog** → **ADC1**:
   - **IN1** تیکش رو بزن
   - **Continuous Conversion Mode**: **Enabled**
   - **DMA Continuous Requests**: **Enabled**
5. تب **DMA Settings**:
   - **Add** → **ADC1** → **Circular** mode
6. تب **Connectivity** → **USART2**:
   - **Mode**: **Asynchronous**
   - **Baud Rate**: 115200
   - **Word Length**: 8 Bits
   - **Stop Bits**: 1
   - **Parity**: None
7. تب **NVIC**:
   - **USART2 global interrupt**: تیکش رو بزن
8. **Generate Code**

### مرحله ۳: کدنویسی توی STM32CubeIDE

توی `Core/Src/main.c`، این تغییرات رو بده:

#### ۱. متغیرهای سراسری (بالای `main()`)

```c
/* USER CODE BEGIN PV */
uint16_t adc_buffer[1];  // DMA buffer for ADC
char uart_buffer[64];    // Buffer for UART message
/* USER CODE END PV */
```

#### ۲. راه‌اندازی DMA و ADC (توی `main()`، بعد `MX_ADC1_Init()`)

```c
/* USER CODE BEGIN 2 */
// Start ADC with DMA
HAL_ADC_Start_DMA(&hadc1, (uint32_t*)adc_buffer, 1);

// Print welcome message
sprintf(uart_buffer, "UPS ADC Test Started\r\n");
HAL_UART_Transmit(&huart2, (uint8_t*)uart_buffer, strlen(uart_buffer), 100);
/* USER CODE END 2 */
```

#### ۳. حلقه اصلی (`while(1)`)

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // adc_buffer[0] has the latest ADC value (0-4095)
    uint16_t adc_val = adc_buffer[0];

    // Convert to voltage: 3.3V * adc_val / 4095
    float voltage = (3.3f * adc_val) / 4095.0f;

    // Create message
    sprintf(uart_buffer, "ADC: %4d | Voltage: %.2f V\r\n", adc_val, voltage);

    // Send over UART
    HAL_UART_Transmit(&huart2, (uint8_t*)uart_buffer, strlen(uart_buffer), 100);

    // Wait 500ms
    HAL_Delay(500);

    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
/* USER CODE END 3 */
```

#### ۴. Include های لازم (بالای فایل)

مطمئن شو این خط‌ها هستن (معمولاً CubeMX خودش اضافه می‌کنه):

```c
#include <stdio.h>
#include <string.h>
```

### مرحله ۴: تنظیمات UART توی STM32CubeIDE

1. پنل **Window → Show View → Terminal** رو باز کن
2. روی آیکون Terminal کلیک کن (یا Ctrl+Alt+Shift+T)
3. **Open a Terminal** بزن
4. تنظیمات:
   - **Choose terminal**: Serial Terminal
   - **Serial port**: COMx (پورتی که Nucleo بهش وصله — توی Device Manager ببین)
   - **Baud rate**: 115200
   - **Data size**: 8
   - **Parity**: None
   - **Stop bits**: 1
5. **OK** بزن

> 💡 **پیدا کردن COM Port:**
> توی Windows، Device Manager → Ports (COM & LPT) → ببین "STMicroelectronics STLink Virtual COM Port" روی کدوم COM هست.

### مرحله ۵: تست

1. کد رو **Build** کن (Project → Build All)
2. **Run** کن (مثلث سبز)
3. توی Terminal باید ببینی:
   ```
   UPS ADC Test Started
   ADC: 2048 | Voltage: 1.65 V
   ADC: 2048 | Voltage: 1.65 V
   ADC: 2048 | Voltage: 1.65 V
   ```
4. پتانسیومتر رو بچرخون:
   - یه طرف → `ADC: 0 | Voltage: 0.00 V`
   - وسط → `ADC: 2048 | Voltage: 1.65 V`
   - طرف دیگه → `ADC: 4095 | Voltage: 3.30 V`

→ ✅ **تست موفق!**

### مرحله ۶: Commit و Push

```bash
cd C:\Users\YourName\Documents\UPS-2kW-LineInteractive
git add .
git commit -m "Day 2 - ADC + Potentiometer + UART working on Nucleo-F303RE"
git push origin main
```

---

## چک‌لیست این هفته

| روز | کار | وضعیت |
|:---:|-----|:-----:|
| ۱ | LED Blink روی Nucleo | ☐ |
| ۲ | ADC + Potentiometer + UART | ☐ |
| ۳ | OrCAD: شماتیک باک کانورتر | ☐ |
| ۴ | OrCAD: PSpice شبیه‌سازی باک | ☐ |
| ۵ | Altium: شماتیک باک | ☐ |
| ۶ | Altium: PCB Layout باک | ☐ |
| ۷ | Python: شبیه‌سازی SPWM | ☐ |

---

*امیرحسین — هر سوالی داری، هر جایی گیر کردی، برگرد و بپرس. من اینجام.*
*موفق باشی!*
