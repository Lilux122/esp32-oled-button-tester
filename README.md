# esp32-oled-button-tester
🎮 ESP32 кнопочный тест с OLED дисплеем
Простой, но забавный проект для демонстрации работы кнопок с микроконтроллером ESP32 и вывода различного контента на OLED-дисплей 0.96" (SSD1306). Каждая кнопка отображает уникальное сообщение или простое изображение.

(Здесь вы можете вставить GIF-анимацию, показывающую работу устройства)

🚀 Возможности
Интерактивность: 6 кнопок для управления выводимым контентом.

Визуализация: Вывод текста и простой пиксельной графики на OLED-дисплей.

Анти-дребезг: Программная реализация подавления дребезга контактов для стабильной работы кнопок.

Простота: Легко читаемый и модифицируемый код, идеально подходящий для начинающих.

Забавный контент: От "Hello world!" до пиксельного динозаврика из Google Chrome!

🛠️ Компоненты
Для сборки проекта вам понадобятся:

Компонент

Количество

Примечание

ESP32 WROOM-32

1

Основной микроконтроллер

OLED дисплей 0.96" (SSD1306)

1

I2C версия, 128x64 пикселя

Тактовые кнопки

6

Любые подходящие кнопки

Макетная плата

1

Для сборки схемы без пайки

Соединительные провода

~15

"Папа-папа"

🔌 Схема подключения
Все кнопки подключаются одной ножкой к соответствующему пину GPIO на ESP32, а другой ножкой — к земле (GND). В коде используется внутренний подтягивающий резистор (INPUT_PULLUP), поэтому внешние резисторы не нужны.

OLED Дисплей (I2C):

VCC -> 3.3V на ESP32

GND -> GND на ESP32

SDA -> GPIO 23

SCL -> GPIO 22

Кнопки:
| Кнопка | Пин ESP32 | Действие |
| :---: | :---: | :--- |
| 1 | GPIO 26 | Отображает "Hello world!" |
| 2 | GPIO 25 | Рисует кораблик |
| 3 | GPIO 32 | Отображает "lol" |
| 4 | GPIO 12 | Рисует динозаврика Google |
| 5 | GPIO 14 | Отображает "Maksimka" |
| 6 | GPIO 27 | Отображает "Artemka" |

⚙️ Установка и прошивка
Проект разработан с использованием PlatformIO IDE в Visual Studio Code.

Откройте проект в VS Code с установленным PlatformIO.

Установите библиотеки: PlatformIO автоматически установит необходимые зависимости, указанные в platformio.ini:

adafruit/Adafruit GFX Library

adafruit/Adafruit SSD1306

Подключите ESP32 к вашему компьютеру.

Скомпилируйте и загрузите прошивку, нажав на кнопку "Upload" в PlatformIO.

💡 Как это работает
Основная логика находится в файле src/main.cpp.

setup(): Инициализирует Serial порт, I2C интерфейс, дисплей и настраивает пины кнопок в режим INPUT_PULLUP.

loop(): В цикле постоянно опрашиваются все кнопки с помощью функции handleButton(). При обнаружении нажатия (button.pressed == true), вызывается соответствующая функция для отображения контента на дисплее.

handleButton(): Реализует механизм подавления дребезга (debounce). Она отслеживает изменение состояния пина и, если оно стабильно в течение DEBOUNCE_DELAY (250 мс), регистрирует нажатие.

display...(): Набор функций, каждая из которых отвечает за очистку дисплея и отрисовку определенного текста или изображения.

<details>
<summary>💻 Посмотреть исходный код</summary>

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// Настройки дисплея
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
#define OLED_ADDRESS 0x3C

// Пины I2C
#define I2C_SDA_PIN 23
#define I2C_SCL_PIN 22

// Пины кнопок
#define BUTTON_1_PIN 26  // "Hello world!"
#define BUTTON_2_PIN 25  // Кораблик
#define BUTTON_3_PIN 32  // "lol"
#define BUTTON_4_PIN 12  // Динозаврик Google
#define BUTTON_5_PIN 14  // "Maksimka"
#define BUTTON_6_PIN 27  // "LOX"

// Константы для дебаунса
#define DEBOUNCE_DELAY 250

// Создание объекта дисплея
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Структура для хранения состояния кнопки
struct Button {
  int pin;
  bool lastState;
  bool currentState;
  unsigned long lastDebounceTime;
  bool pressed;
};

// Массив кнопок
Button buttons[6] = {
  {BUTTON_1_PIN, HIGH, HIGH, 0, false},
  {BUTTON_2_PIN, HIGH, HIGH, 0, false},
  {BUTTON_3_PIN, HIGH, HIGH, 0, false},
  {BUTTON_4_PIN, HIGH, HIGH, 0, false},
  {BUTTON_5_PIN, HIGH, HIGH, 0, false},
  {BUTTON_6_PIN, HIGH, HIGH, 0, false}
};

// Объявления функций
void handleButton(int buttonIndex);
void displayHelloWorld();
void displayShip();
void displayLol();
void displayDinosaur();
void displayMaksimka();
void displayLox();

void setup() {
  Serial.begin(115200);
  
  // Инициализация I2C
  Wire.begin(I2C_SDA_PIN, I2C_SCL_PIN);
  
  // Инициализация дисплея
  if(!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDRESS)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;); // Зависание в случае ошибки
  }
  
  Serial.println(F("OLED initialized successfully"));
  
  // Инициализация кнопок
  for(int i = 0; i < 6; i++) {
    pinMode(buttons[i].pin, INPUT_PULLUP);
  }
  
  // Отображение начального текста
  displayHelloWorld();
}

void loop() {
  // Обработка всех кнопок
  for(int i = 0; i < 6; i++) {
    handleButton(i);
  }
  
  // Проверка нажатий и выполнение соответствующих действий
  if(buttons[0].pressed) {
    displayHelloWorld();
    buttons[0].pressed = false;
  }
  if(buttons[1].pressed) {
    displayShip();
    buttons[1].pressed = false;
  }
  if(buttons[2].pressed) {
    displayLol();
    buttons[2].pressed = false;
  }
  if(buttons[3].pressed) {
    displayDinosaur();
    buttons[3].pressed = false;
  }
  if(buttons[4].pressed) {
    displayMaksimka();
    buttons[4].pressed = false;
  }
  if(buttons[5].pressed) {
    displayLox();
    buttons[5].pressed = false;
  }
}

// Функция обработки дебаунса кнопок
void handleButton(int buttonIndex) {
  int reading = digitalRead(buttons[buttonIndex].pin);
  
  // Если состояние изменилось, сброс таймера дебаунса
  if (reading != buttons[buttonIndex].lastState) {
    buttons[buttonIndex].lastDebounceTime = millis();
  }
  
  // Если прошло достаточно времени с последнего изменения
  if ((millis() - buttons[buttonIndex].lastDebounceTime) > DEBOUNCE_DELAY) {
    // Если состояние стабильно изменилось
    if (reading != buttons[buttonIndex].currentState) {
      buttons[buttonIndex].currentState = reading;
      
      // Кнопка нажата (переход от HIGH к LOW)
      if (buttons[buttonIndex].currentState == LOW) {
        buttons[buttonIndex].pressed = true;
        Serial.print("Button ");
        Serial.print(buttonIndex + 1);
        Serial.println(" pressed");
      }
    }
  }
  
  buttons[buttonIndex].lastState = reading;
}

// Функция для отображения "Hello world!"
void displayHelloWorld() {
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  
  // Центрирование текста
  int16_t x1, y1;
  uint16_t w, h;
  display.getTextBounds("Hello world!", 0, 0, &x1, &y1, &w, &h);
  display.setCursor((SCREEN_WIDTH - w) / 2, (SCREEN_HEIGHT - h) / 2);
  display.println("Hello world!");
  
  display.display();
}

// Функция для отображения кораблика
void displayShip() {
  display.clearDisplay();
  
  // Центр экрана
  int centerX = SCREEN_WIDTH / 2;
  int centerY = SCREEN_HEIGHT / 2;
  
  // Корпус корабля (трапеция)
  display.drawLine(centerX - 25, centerY + 5, centerX + 25, centerY + 5, SSD1306_WHITE);
  display.drawLine(centerX - 25, centerY + 5, centerX - 15, centerY + 15, SSD1306_WHITE);
  display.drawLine(centerX + 25, centerY + 5, centerX + 15, centerY + 15, SSD1306_WHITE);
  display.drawLine(centerX - 15, centerY + 15, centerX + 15, centerY + 15, SSD1306_WHITE);
  
  // Заполнение корпуса
  for(int y = centerY + 6; y < centerY + 15; y++) {
    int width = 20 - (y - centerY - 6) * 2;
    display.drawLine(centerX - width/2, y, centerX + width/2, y, SSD1306_WHITE);
  }
  
  // Мачта
  display.drawLine(centerX - 5, centerY - 15, centerX - 5, centerY + 5, SSD1306_WHITE);
  display.drawLine(centerX + 5, centerY - 10, centerX + 5, centerY + 5, SSD1306_WHITE);
  
  // Паруса
  display.fillTriangle(centerX - 5, centerY - 15, centerX - 20, centerY - 10, centerX - 5, centerY - 5, SSD1306_WHITE);
  display.fillTriangle(centerX + 5, centerY - 10, centerX + 18, centerY - 8, centerX + 5, centerY - 2, SSD1306_WHITE);
  
  // Флаг
  display.fillRect(centerX - 5, centerY - 20, 8, 5, SSD1306_WHITE);
  
  display.display();
}

// Функция для отображения "lol"
void displayLol() {
  display.clearDisplay();
  display.setTextSize(4);
  display.setTextColor(SSD1306_WHITE);
  
  // Центрирование текста
  int16_t x1, y1;
  uint16_t w, h;
  display.getTextBounds("lol", 0, 0, &x1, &y1, &w, &h);
  display.setCursor((SCREEN_WIDTH - w) / 2, (SCREEN_HEIGHT - h) / 2);
  display.println("lol");
  
  display.display();
}

// Функция для отображения динозаврика Google
void displayDinosaur() {
  display.clearDisplay();
  
  // Центр экрана
  int centerX = SCREEN_WIDTH / 2;
  int centerY = SCREEN_HEIGHT / 2;
  
  // Тело динозаврика
  display.fillRect(centerX - 15, centerY - 5, 25, 15, SSD1306_WHITE);
  
  // Голова
  display.fillRect(centerX - 20, centerY - 10, 15, 10, SSD1306_WHITE);
  
  // Хвост
  display.fillTriangle(centerX + 10, centerY - 5, centerX + 20, centerY - 8, centerX + 10, centerY + 5, SSD1306_WHITE);
  
  // Ноги
  display.fillRect(centerX - 10, centerY + 10, 4, 8, SSD1306_WHITE);
  display.fillRect(centerX + 2, centerY + 10, 4, 8, SSD1306_WHITE);
  
  // Глаз
  display.fillCircle(centerX - 15, centerY - 7, 2, SSD1306_BLACK);
  display.drawPixel(centerX - 15, centerY - 7, SSD1306_WHITE);
  
  // Рот
  display.drawLine(centerX - 20, centerY - 2, centerX - 17, centerY - 2, SSD1306_BLACK);
  
  // Шипы на спине
  display.drawTriangle(centerX - 8, centerY - 5, centerX - 5, centerY - 10, centerX - 2, centerY - 5, SSD1306_WHITE);
  display.drawTriangle(centerX + 2, centerY - 5, centerX + 5, centerY - 8, centerX + 8, centerY - 5, SSD1306_WHITE);
  
  // Передние лапы
  display.fillRect(centerX - 8, centerY + 5, 3, 6, SSD1306_WHITE);
  display.fillRect(centerX + 2, centerY + 5, 3, 6, SSD1306_WHITE);
  
  display.display();
}

// Функция для отображения "Maksimka"
void displayMaksimka() {
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  
  // Центрирование текста
  int16_t x1, y1;
  uint16_t w, h;
  display.getTextBounds("Maksimka", 0, 0, &x1, &y1, &w, &h);
  display.setCursor((SCREEN_WIDTH - w) / 2, (SCREEN_HEIGHT - h) / 2);
  display.println("Maksimka");
  
  display.display();
}

// Функция для отображения "LOX"
void displayLox() {
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  
  // Центрирование текста
  int16_t x1, y1;
  uint16_t w, h;
  display.getTextBounds("Artemka", 0, 0, &x1, &y1, &w, &h);
  display.setCursor((SCREEN_WIDTH - w) / 2, (SCREEN_HEIGHT - h) / 2);
  display.println("Artemka");
  
  display.display();
}

</details>
