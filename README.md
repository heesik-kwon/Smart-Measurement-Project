# 💻 Smart Measurement Project  
> 멀티 센서 기반의 스마트 계측 시스템 프로젝트입니다. Verilog로 구현된 stopwatch & watch, HC-SR04 초음파 센서, DHT11 온습도 센서, FIFO & UART를 통한 데이터 통신 기능을 포함합니다.

---

## 📌 프로젝트 개요

| 항목             | 내용                                                   |
|------------------|--------------------------------------------------------|
| **⏱️ 수행 기간** | 2024.07.04 ~ 2024.07.10                               |
| **🖥️ 개발 환경**  | Vivado, VSCode                                        |
| **📦 플랫폼**     | Basys3 FPGA 보드 (Xilinx Artix-7)                    |
| **💻 언어**       | Verilog HDL                                           |
| **📡 통신**       | UART 통신       |
| **🧪 센서**       | HC-SR04 (초음파 거리 센서), DHT11 (온습도 센서)       |

---

## 🎯 주요 기능

- ⏱️ **스톱워치 및 디지털 시계 기능**  
  - 버튼으로 시간 측정/정지/초기화  
  - 분:초 / 시:분 모드 전환 가능  
  - 쿠쿠 알람 기능 (매 정시마다 `o_cuckoo` 출력)

- 📟 **FND 7-Segment 디스플레이 제어**  
  - 시계/스톱워치/센서 데이터에 따라 동적으로 출력  
  - 클럭 분주 및 다중 자리 선택 방식 구현  
  - FND 출력 선택: 시계, 거리, 온습도 (sw[5], sw[6] 조합으로 제어)

- 📡 **초음파 거리 측정 (HC-SR04)**  
  - 거리값을 FND 및 UART로 출력  
  - 거리값 ASCII 변환 및 `"Distance = xxxxcm"` 문자열 전송  
  - UART 전송 버퍼(FIFO 포함) 및 송신 FSM 구성

- 🌡️ **온습도 측정 (DHT11)**  
  - 온도/습도 값을 FND 및 UART로 출력  
  - `"Temp = XXC\r\nHumi = XX%"` 형식으로 UART 전송  
  - 센서 FSM으로 데이터 동기화 및 유효성 검증 처리

- 🛰️ **UART 통신 기반 명령 수신 및 처리**  
  - `"RESET"`, `"CLEAR"`, `"SR"`, `"DHT"`, `"UP"`, `"DOWN"` 등  
  - UART 수신 FIFO와 FSM 기반 커맨드 파서 설계

---

## 🧩 시스템 블록 다이어그램 (System Block Diagram)

> 시스템의 전체 동작 흐름을 나타낸 블록 다이어그램으로, 입력 신호부터 제어 및 출력까지의 구조를 모듈 단위로 표현하였습니다.

<img src="https://github.com/user-attachments/assets/7c95914c-8028-47a9-bafc-100f97f3f380" width="600"/>

---

## 🧩 Top-Level RTL Schematic (Vivado)

> Vivado를 통해 생성한 Top-Level RTL Schematic입니다. 각 하드웨어 모듈 간의 실제 연결 관계를 기반으로 회로 구조를 시각화하여, 시스템의 데이터 흐름과 제어 경로를 명확하게 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/eb8632bb-1630-4219-835c-05fe5ffe4312" width="750"/>

---

## 🧩 HC-SR04 RTL Schematic (Vivado)

> Vivado에서 구성한 초음파 거리 측정 시스템의 회로도입니다. 버튼 입력으로 HC-SR04 센서를 트리거하고, 측정된 거리를 FND에 표시하며 UART를 통해 decimal 형식으로 송신합니다.

<img src="https://github.com/user-attachments/assets/05de31f7-171f-4850-987a-452831baa44f" width="750"/>

---

## 🧩 하드웨어 회로 구성도 (HW Circuit Diagram)

> 실제 배선 및 회로 연결 상태를 전기적으로 표현한 도식입니다.

<img src="https://github.com/user-attachments/assets/d4058982-6451-42ec-b9ad-9de2e3149383" width="400"/>

---

## 🔧 하드웨어 주요 기능 (HW-Level Features)

Raspberry Pi를 중심으로 다양한 센서 및 디스플레이 장치와 연동하여 실시간 상태 감지 및 사용자 입력에 반응하는 기능들을 제공합니다.

### 📺 1. SPI 통신 기반 OLED 디스플레이 출력

> SPI를 통해 `128x64 OLED` 화면에 과일 분류 결과, 저장 상태, 온습도 등을 실시간 출력합니다.

**💡 사용 핀**:  
- SCK: GPIO11 (Pin 23)  
- MOSI: GPIO10 (Pin 19)  
- CS: GPIO05 (Pin 29)  
- DC: GPIO06 (Pin 31)  
- RESET: GPIO13 (Pin 33)

**📄 관련 코드 예시:**

```python
WIDTH, HEIGHT = 128, 64
spi = busio.SPI(clock=board.SCK, MOSI=board.MOSI)
cs = digitalio.DigitalInOut(board.D5)
dc = digitalio.DigitalInOut(board.D6)
reset = digitalio.DigitalInOut(board.D13)
oled = adafruit_ssd1306.SSD1306_SPI(WIDTH, HEIGHT, spi, dc, reset, cs)

# OLED 출력용 이미지 그리기
image = Image.new("1", (oled.width, oled.height))
draw = ImageDraw.Draw(image)
draw.text((0, 0), "apple R:1 U:2", font=font, fill=255)
oled.image(image)
oled.show()
```

## 🔧 검증 및 시뮬레이션 (Verification & Simulation)

각 모듈의 동작을 Testbench를 통해 검증하고, 파형 시뮬레이션을 통해 신호 변화와 타이밍을 확인하였습니다.

### ✅ Timing Verification of Trigger Pulse

> HC-SR04 초음파 센서는 40kHz 주파수로 8사이클의 초음파를 송신하며, 이때 걸리는 시간은 약 **200μs**입니다.

<img src="https://github.com/user-attachments/assets/466f310e-62be-4595-9ef1-6715ebf31d5e" width="700"/>

- **센서 사양 계산**  
  40kHz → 25μs 주기  
  8사이클 → 25μs × 8 = **200μs**

- **시뮬레이션 결과**  
  `trig` 이후 `echo` 출력까지 약 **239μs** 소요 → 정상 범위 내
  
시뮬레이션 결과에서는 `trig` 신호 발생 후, `echo` 신호가 약 **239μs** 후에 High가 되는 것을 확인할 수 있습니다.  
이는 센서 데이터시트상의 사양과 유사하며, **정상적으로 트리거 동작이 수행됨을 검증**할 수 있습니다.

---

### 🔄 high_level_detector 동작 과정

> 아래는 high_level_detector 모듈의 기능을 검증하기 위한 Testbench 시뮬레이션 결과입니다. 이 모듈은 echo 신호의 high 레벨 구간을 감지하고, 그 구간의 시작과 종료 시점에 각각 high_level_flag와 done 신호를 갱신합니다.

<img src="https://github.com/user-attachments/assets/875b5226-c357-4751-a0d4-f21fae461656" width="650"/>

1. **초기 상태**
   - `echo`, `echo_d`, `high_level_flag`, `done` 모두 0

2. **상승 에지 감지**
   - `echo`가 0 → 1로 바뀌고, 다음 클럭에서 `echo_d`가 업데이트됨
   - `echo && !echo_d` 조건 만족 → `high_level_flag <= 1`, `done <= 0`
   - → **high 구간 시작**

3. **high 유지 구간**
   - `echo`가 1을 유지하는 동안 상태 변화 없음

4. **하강 에지 감지**
   - `echo`가 1 → 0으로 바뀌고, 다음 클럭에서 `echo_d` 업데이트됨
   - `!echo && echo_d` 조건 만족 → `high_level_flag <= 0`, `done <= 1`
   - → **high 구간 종료**

---

### 📏 거리 계산기 `calculator` 동작 과정

> HC-SR04 초음파 센서의 `echo` 신호가 high인 구간 동안 클럭을 카운트하여 거리를 계산하는 로직입니다. 아래는 시뮬레이션을 기반으로 한 주요 동작 과정입니다:

<img src="https://github.com/user-attachments/assets/78eef7f2-433a-4606-bd5b-e2be364a5822" width="650"/>

1. **거리 측정 시작**
   - `high_level_flag = 1`인 동안 `count`가 클럭마다 증가합니다.

2. **측정 종료 감지**
   - `done` 신호가 1이 되면:
     - `distance <= count / 58` (단위: cm)
     - `dist_done <= 1`로 설정됨
     - `count`는 초기화됩니다

3. **거리 계산 예시**
   - 예: `count = 64`일 때 → `distance = 64 / 58 ≒ 1cm`

이 시뮬레이션은 거리 측정 타이밍과 계산 결과가 정확하게 동작하고 있음을 검증합니다.

---

### ✅ SR04 Controller 전체 동작

> 초음파 거리 측정 모듈의 전체 동작을 검증한 시뮬레이션 결과입니다. `sr04_controller` 모듈은 `tick_gen`, `start_trigger`, `high_level_detector`, `calculator`로 구성되어 있으며, 트리거부터 거리 계산까지의 흐름이 정확히 수행되는지를 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/3b4714b9-84cc-4914-8007-a8afee53aab2" width="650"/>

1. **btn_start**  
    - 사용자가 버튼 입력으로 거리 측정 시작을 요청합니다.

2. **trig**  
    - `start_trigger` 모듈에 의해 10μs 펄스가 발생합니다.

3. **echo**
    - 센서로부터 echo 신호가 수신되며, 이 구간의 폭이 거리 계산의 기준이 됩니다.

4. **done**  
    - `high_level_detector` 모듈에서 echo의 하강 에지를 감지하여 완료 신호를 발생시킵니다.

5. **거리 출력**  
    - `calculator` 모듈은 count 값을 바탕으로 거리(cm)를 계산하며, `distance` 신호에 결과가 출력됩니다.  
       → 예: `232μs / 58 = 4cm`

`trig`, `echo`, `done`, `distance` 신호들이 순차적으로 정상 동작함을 파형을 통해 확인하였습니다. 계산된 거리 값 또한 정확히 **4cm**로 출력되는 것을 확인할 수 있었고, **SR04 초음파 모듈 제어 및 거리 측정 알고리즘이 정상 작동함을 시뮬레이션으로 입증**






**💡 사용 핀**:  
- Button: GPIO17 (Pin 11)

**📄 관련 코드 예시:**

```python
button = digitalio.DigitalInOut(board.D17)
button.direction = digitalio.Direction.INPUT
button.pull = digitalio.Pull.DOWN
prev_button_state = False

if current_button_state and not prev_button_state:
    now_str = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    log_row = [now_str, temperature_str, humidity_str, fruit_info]
    logs.append(log_row)
    if len(logs) > 5:
        logs = logs[-5:]
    with open(log_file, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(header)
        writer.writerows(logs)
    saved_flag = True
    saved_time = time.time()
```
![image](https://github.com/user-attachments/assets/7f62eb33-5fc6-47df-9d96-84ecbf3accc8)

---

### 🌡️ 3. DHT11 온습도 센서 측정 및 OLED 출력

> 내부 온도와 습도를 주기적으로 측정하여 OLED에 출력하고, 로그 저장 시 함께 기록합니다.

**💡 사용 핀**:  
- DHT11: GPIO04 (Pin 7)

**📄 관련 코드 예시:**

```python
import adafruit_dht
dht_device = adafruit_dht.DHT11(board.D4)

try:
    temperature = dht_device.temperature
    humidity = dht_device.humidity
    env_line = f"{temperature:.1f}C {humidity:.1f}%"
except:
    env_line = "Sensor Error"

draw.text((0, 48), env_line, font=font, fill=255)
oled.image(image)
oled.show()
```

---


## 💻 소프트웨어 처리 흐름도 (SW Flowchart)

> 과일 인식, 경고 처리, 센서 로깅 등 소프트웨어 내부 흐름을 시각화한 순서도입니다.

<img src="https://github.com/user-attachments/assets/5f16bceb-147d-4bfa-9e66-9075d5058d6c" width="300"/>

---

## 📂 프로젝트 폴더 구조

```

```

---

## 💡 기대 효과

- 가정 내 **식재료 관리 자동화**
- **식중독 예방**을 위한 상한 과일 감지
- **경량 AI 시스템 구현** 예제로 활용 가능
