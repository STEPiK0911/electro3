# Работу выполнил Погребняк Степан Сиваселвамович

# Цель работы

![image](https://github.com/user-attachments/assets/0be72ad4-5e08-4d48-934c-bc6e3907d4c9)




```
#include <stdint.h>

#define GPIOA_BASE  0x48000000
#define GPIOB_BASE  0x48000400
#define RCC_BASE    0x40021000

#define GPIOA_MODER     (*(volatile uint32_t*)(GPIOA_BASE + 0x00))
#define GPIOA_IDR       (*(volatile uint32_t*)(GPIOA_BASE + 0x10))
#define GPIOA_ODR       (*(volatile uint32_t*)(GPIOA_BASE + 0x14))

#define GPIOB_MODER     (*(volatile uint32_t*)(GPIOB_BASE + 0x00))
#define GPIOB_ODR       (*(volatile uint32_t*)(GPIOB_BASE + 0x14))

#define RCC_AHBENR      (*(volatile uint32_t*)(RCC_BASE + 0x14))

// Битовые маски сегментов для цифр 0-9
uint8_t segment_map[] = {
    0b00111111, // 0
    0b00000110, // 1
    0b01011011, // 2
    0b01001111, // 3
    0b01100110, // 4
    0b01101101, // 5
    0b01111101, // 6
    0b00000111, // 7
    0b01111111, // 8
    0b01101111  // 9
};

// Инициализация GPIO
void initGPIO() {
    // Включение тактирования GPIOA и GPIOB
    RCC_AHBENR |= (1 << 17) | (1 << 18);

    // Настройка GPIOA: PA0-PA7 как Output (для сегментов)
    GPIOA_MODER &= ~(0xFFFF);  
    GPIOA_MODER |= 0x5555;

    // Настройка GPIOB: PB0 как Output (для светодиода)
    GPIOB_MODER &= ~(0x3);
    GPIOB_MODER |= 0x1;

    // Настройка GPIOA: PA8 как Input (для генератора логических уровней)
    GPIOA_MODER &= ~(0x3 << 16);
}

// Отображение цифры на семисегментном индикаторе
void displayDigit(uint8_t digit) {
    GPIOA_ODR = segment_map[digit]; // Установка состояния выходов
}

// Задержка (простейший цикл ожидания)
void simpleDelay(volatile uint32_t count) {
    while (count--);
}

// Главная функция
int main() {
    initGPIO();

    while (1) {
        // Считывание логического уровня с PA8
        uint8_t level = (GPIOA_IDR & (1 << 8)) ? 1 : 0;

        // Управление светодиодом (PB0)
        if (level) {
            GPIOB_ODR |= (1 << 0);  // Включить светодиод
        } else {
            GPIOB_ODR &= ~(1 << 0); // Выключить светодиод
        }

        // Считывание двоичного числа с PA0-PA7
        uint8_t binary_value = GPIOA_IDR & 0xFF;

        // Преобразование двоичного числа в десятичное
        uint8_t decimal_value = 0;
        for (int i = 0; i < 8; i++) {
            if (binary_value & (1 << i)) {
                decimal_value += (1 << i);
            }
        }

        // Отображение десятичного числа на семисегментном индикаторе
        displayDigit(decimal_value % 10);

        // Задержка
        simpleDelay(1000000);
    }

    return 0;
}

```
