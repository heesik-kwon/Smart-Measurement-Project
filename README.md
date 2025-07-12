# 💻 Smart Measurement System
> 멀티 센서 기반의 스마트 계측 시스템 프로젝트입니다.     
Verilog로 stopwatch & watch, HC-SR04 초음파 센서, DHT11 온습도 센서, FIFO & UART 기반 데이터 통신 기능을 직접 설계하고 검증하였습니다.

---

# 📌 프로젝트 개요

| 항목             | 내용                                                   |
|------------------|--------------------------------------------------------|
| **⏱️ 수행 기간** | 2024.05.30 ~ 2024.06.01                               |
| **🖥️ 개발 환경**  | Vivado, VSCode                                        |
| **📦 플랫폼**     | Basys3 FPGA 보드 (Xilinx Artix-7)                    |
| **💻 언어**       | Verilog HDL                                           |
| **📡 통신**       | UART 통신       |
| **🧪 센서**       | HC-SR04 (초음파 거리 센서), DHT11 (온습도 센서)       |

---

# 🎯 주요 기능

- ⏱️ **스톱워치 및 디지털 시계**  
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

# 📦 시스템 블록 다이어그램 (System Block Diagram)

> 시스템의 전체 동작 흐름을 나타낸 블록 다이어그램으로, 입력 신호부터 제어 및 출력까지의 구조를 모듈 단위로 표현하였습니다.

<img width="1356" height="678" alt="image" src="https://github.com/user-attachments/assets/145faffa-1bf9-4a39-aa4e-e806bd33d738" />


---

# 🧩 Top-Level RTL Schematic (Vivado)

> Vivado를 통해 생성한 Top-Level RTL Schematic입니다.   
각 하드웨어 모듈 간의 실제 연결 관계를 기반으로 회로 구조를 시각화하여, 시스템의 데이터 흐름과 제어 경로를 명확하게 확인할 수 있습니다.

<img width="1826" height="774" alt="image" src="https://github.com/user-attachments/assets/f6b89604-35e3-41d3-b827-afd219e8cefe" />


---

# 🧩 HC-SR04 RTL Schematic (Vivado)

> Vivado에서 구성한 초음파 거리 측정 시스템의 RTL Schematic입니다.   
버튼 입력으로 HC-SR04 센서를 트리거하고, 측정된 거리를 FND에 표시하며 UART를 통해 decimal 형식으로 송신합니다.

<img width="1818" height="538" alt="image" src="https://github.com/user-attachments/assets/ebc25f16-0cfb-4d7f-bbad-770c9a74cae6" />


---

# 🚥 Basys3 보드 LED 및 SW 기능 정리

> 프로젝트에서 사용된 **Basys3 FPGA 보드의 LED와 SWITCH 역할**을 정리한 표입니다.  
LED는 실시간 상태 출력 용도로 사용되며, SW는 모드 전환 및 기능 제어에 활용됩니다.

<img src="https://github.com/user-attachments/assets/805d0462-ad33-4cca-b066-46175dcb481d" width="750"/>

---

# 📈  HC-SR04 + UART 검증 및 시뮬레이션

> 각 모듈의 동작을 Testbench를 통해 검증하고, 파형 시뮬레이션을 통해 신호 변화와 타이밍을 확인하였습니다.

## 🔄 Timing Verification of Trigger Pulse

> HC-SR04 초음파 센서는 40kHz 주파수로 8사이클의 초음파를 송신하며, 이때 걸리는 시간은 약 **200μs**입니다.

<img src="https://github.com/user-attachments/assets/466f310e-62be-4595-9ef1-6715ebf31d5e" width="850"/>

- **센서 사양 계산**  
  40kHz → 25μs 주기  
  8사이클 → 25μs × 8 = **200μs**

- **시뮬레이션 결과**  
  `trig` 이후 `echo` 출력까지 약 **239μs** 소요 → 정상 범위 내

### ✅ 검증 결과

- 시뮬레이션 결과에서는 `trig` 신호 발생 후, `echo` 신호가 약 **239μs** 후에 High가 되는 것을 확인할 수 있습니다.  
- 이는 센서 데이터시트상의 사양과 유사하며, **정상적으로 트리거 동작이 수행됨을 검증**할 수 있습니다.

---

## 🔄 High Level Detecting Verification

> 아래는 `high_level_detector` 모듈의 기능을 검증하기 위한 Testbench 시뮬레이션 결과입니다.     
> 이 모듈은 echo 신호의 high 레벨 구간을 감지하고, 그 구간의 시작과 종료 시점에 각각 high_level_flag와 done 신호를 갱신합니다.

<img src="https://github.com/user-attachments/assets/875b5226-c357-4751-a0d4-f21fae461656" width="850"/>

### 🧭 시뮬레이션 동작 흐름

| 시점 (ns)    | 조건                         | 설명                                |
|--------------|----------------------------------|-------------------------------------|
| 약 50,000 ns | `echo ↑`                        | echo 신호가 High로 상승             |
| 약 60,000 ns | `high_level_flag ↑`             | FSM이 상승 에지를 감지하고 flag 설정 |
| 약 200,000 ns| `echo ↓`, `high_level_flag ↓`, `done ↑` | echo 종료와 동시에 done 출력        |

### ✅ 검증 결과

- FSM은 `echo ↑`를 감지하여 `high_level_flag`를 High로 설정하고, 이후 `echo ↓`가 발생하면 `done`을 **동시에 출력**하며 측정 완료 신호를 생성함
- 이는 echo 신호의 유효 시간 동안 거리 측정 준비가 되었음을 의미하며, **FSM이 정상적으로 High pulse 감지 및 완료 조건을 처리하고 있음을 검증**할 수 있음

---

## 🔄 Calculator Verification

> HC-SR04 초음파 센서의 `echo` 신호가 high인 구간 동안 클럭을 카운트하여 거리를 계산하는 로직입니다.

<img src="https://github.com/user-attachments/assets/78eef7f2-433a-4606-bd5b-e2be364a5822" width="850"/>

### 🧭 시뮬레이션 동작 흐름

| 시점 (ns)        | 조건 또는 동작                     | 설명                                         |
|------------------|----------------------------------|----------------------------------------------|
| 약 1,020,000     | `high_level_flag ↑`             | echo 신호가 High로 전환되어 count 시작      |
| 1,078,000 ~ 1,080,000 | `count ≈ 64` 도달                | echo High 유지 시간 측정 완료                 |
| 약 1,080,000     | `done ↑`                        | echo가 Low로 전환되며 거리 측정 종료         |
| 약 1,090,000     | `distance ← 64 / 58 = 1`        | 계산 결과 distance 값 저장됨 (`001`)         |
| 약 1,100,000     | `count ← 0` 초기화              | 다음 측정을 위한 준비 완료                  |

### ✅ 검증 결과

- `high_level_flag`가 1인 동안 `count`가 증가하고,
- `done` 신호가 발생한 시점에 count 값을 이용해 거리 계산 수행
- 계산된 거리 값은 `distance = 64 / 58 ≈ 1cm`로 정상적으로 출력됨
- 이후 `count`는 0으로 초기화되어 다음 측정 준비가 완료됨
- 시뮬레이션을 통해 **초음파 거리 계산 로직의 정확한 동작이 입증됨**

---

## 🔄 SR04 Controller Verification

> 초음파 거리 측정 모듈의 전체 동작을 검증한 시뮬레이션 결과입니다. `sr04_controller` 모듈은 `tick_gen`, `start_trigger`, `high_level_detector`, `calculator`로 구성되어 있으며, 트리거부터 거리 계산까지의 흐름이 정확히 수행되는지를 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/3b4714b9-84cc-4914-8007-a8afee53aab2" width="850"/>


### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)        | 조건 또는 동작        | 설명                                                     |
|------------------|------------------------|----------------------------------------------------------|
| 약 498.240       | `start ↑`             | 버튼 입력(`btn_start`) 감지 → 거리 측정 시작               |
| 약 498.240       | `trig ↑`              | 초음파 송신 트리거 신호 발생                              |
| 약 534.240       | `echo ↑`              | 센서로부터 반사파 수신 시작                               |
| 약 766.240       | `echo ↓`, `dist_done ↑` | 반사파 수신 종료 → 거리 계산 완료 신호 출력               |
| 약 780.000       | `dist = 0x004`        | 거리 계산 결과 저장: `232us / 58 ≈ 4cm`                  |

### 📐 거리 계산 식

- **거리 계산 기준**: 거리(cm) = `echo high 유지 시간(μs) / 58`  
- 예시 결과:  
  - `echo` 유지 시간 = 766.240μs − 534.240μs = **232μs**  
  - 계산된 거리: 232 / 58 = **4cm** → `dist = 0x004`

### ✅ 검증 결과

- 버튼 입력 → 거리 출력까지 모든 플로우가 정상적으로 시뮬레이션됨
- `echo` 신호의 지속 시간에 따라 정확한 거리(`4cm`)가 산출됨
- `dist_done` 신호와 함께 결과가 `dist[9:0]`에 저장됨
- 전체 흐름이 **사양에 맞게 잘 구현되었음을 확인**

---

## 🔄 UART Transmission Flow Verification

> 거리 측정이 완료되면 `"Distance = XXXXcm\r\n"` 포맷으로 UART를 통해 메시지가 전송됩니다. 아래는 전체 UART 송신 흐름 시뮬레이션입니다.

<img src="https://github.com/user-attachments/assets/74e1b9f9-f24b-4d6e-ba6f-53f60c58b412" width="850"/>

### 🧭 시뮬레이션 동작 흐름

| 시점 (ns)        | 이벤트 및 조건               | 설명                                                        |
|------------------|------------------------------|-------------------------------------------------------------|
| 약 1,800,000     | `done ↑`                     | 거리 측정 완료 신호 발생 (`dist = 006`)                    |
| 약 1,810,000     | `push_tx ↑`                  | UART 전송 요청 신호 발생 → FIFO or FSM으로 전송 명령         |
| 약 1,820,000     | `tx_busy ↑`                  | UART 송신 시작 상태 진입                                     |
| 이후 구간       | `tx_din` 순차적으로 업데이트 | `D`, `i`, `s`, `t`, `a`, `n`, `c`, `e`, `:`, `0`, `0`, `6`, `\n` 전송 |
| 약 3,000,000+    | `tx_done ↑`                  | 1문자 전송 완료 → 다음 문자 전송 준비                         |
| 이후 반복        | `push_tx`, `tx_busy` 반복    | 다음 문자를 위한 UART 전송 루틴 반복 수행                    |

### ✅ 검증 결과

- 거리 측정 완료(`done`) 신호 발생 후, UART 전송 요청(`push_tx`)이 트리거됨
- `tx_busy`가 1이 되며 UART 모듈이 바이트 단위 송신을 수행
- `tx_din`에는 전송할 문자들이 순차적으로 로드되며, `tx_done` 발생 시 다음 문자로 전환됨
- 전체 문자열(`stance:006\n`)이 정상 전송되는 동안 위 흐름이 반복됨
- 시뮬레이션을 통해 **UART 송신 FSM이 완료 신호 이후 정상 동작함**을 검증할 수 있음
---

# 🌡️ DHT11 온습도 센서 통신 프로토콜

> DHT11 센서는 단일 신호선(Single-Bus)을 사용해 MCU와 데이터를 주고받습니다.     
DHT11의 통신 시퀀스를 시간 흐름에 따라 단계별로 나타낸 다이어그램과 각 구간의 신호 상태입니다.

<img src="https://github.com/user-attachments/assets/64bec993-3103-4d77-aaab-ed1c15fbbd64" width="800"/>

---


# 📦 DHT11 시스템 블록 다이어그램

> Basys3 보드를 기반으로 구성된 **DHT11 온습도 센서 시스템의 블록 다이어그램**입니다.    
사용자의 버튼 입력을 받아 DHT11 센서와 통신하며, 측정된 온습도 데이터를 7세그먼트(FND)에 출력하는 구조입니다.

<img src="https://github.com/user-attachments/assets/65c80b03-df40-40d3-805e-bf9e9b3efe83" width="750"/>

---

# 🧩 DHT11 RTL Schematic (Vivado)

> DHT11 센서로부터 수신한 온습도 데이터에 대해 UART 전송(UART_CTRL_sensor) 및 FND 표시(fnd_controller_dht11)를 동시에 수행하는 센서-송신 통합 RTL Schematic입니다.

<img width="1785" height="709" alt="image" src="https://github.com/user-attachments/assets/e09087e0-8239-412f-b75c-aeb1d5874dae" />

---

# ⏬ DHT11 FSM 상태 전이도

> DHT11 센서 통신을 위한 Finite State Machine (FSM) 상태 전이도입니다.  
FSM은 총 8개의 상태(IDLE ~ STOP)를 순차적으로 거치며, Start Signal → 응답 수신 → 데이터 수신 과정을 수행합니다.

<img src="https://github.com/user-attachments/assets/03a07d73-6a46-4716-aeba-7ee6d14a6fb6" width="750"/>

---

# 📈 DHT11 + UART 검증 및 시뮬레이션

> `TOP_DHT11` 모듈을 기반으로 DHT11 온습도 센서 FSM이 정상적으로 시작되고 상태가 전이되는지 검증하였습니다.  

## 🔁 FSM Start Trigger Verification

> DHT11 센서 FSM의 `start` 신호가 입력되었을 때, FSM 상태(`c_state`)가 `IDLE(0)`에서 `START(1)`로 전이되는 과정을 확인한 시뮬레이션 결과입니다.

<img src="https://github.com/user-attachments/assets/2c555e4b-866d-44c5-ad75-c70a41173455" width="850"/>

- **FSM 초기 상태**
  - `c_state = 0` → IDLE 상태에서 대기 중
  - `dht11_io`는 기본적으로 High 상태를 유지
  - `start` 신호가 LOW였다가 45,000ns 부근에서 HIGH(1)로 전환

- **FSM 상태 전이**
  - `start`가 1로 올라간 이후, FSM은 `c_state = 1` (START 상태)로 전이됨
  - 이는 DHT11 FSM 정의 상 `start` 신호가 트리거 역할을 한다는 점을 시뮬레이션으로 검증한 결과

### ✅ 검증 결과

- `start` 신호가 정상적으로 인식되며 FSM이 IDLE → START로 전이됨
- FSM이 시작되면 이후 `START → WAIT → SYNC_L → SYNC_H` 순서로 진행되며 DHT11 통신 프로토콜을 따라 동작함
- **정상적인 상태 전이 시퀀스가 이루어졌음을 시뮬레이션으로 입증**

---

## 🔁 FSM Transition & Tick Timing Verification

> `DHT11 FSM`이 START → WAIT → SYNC_L 상태로 **정상적으로 전이**되는지,  
그리고 내부 `tick`이 적절히 발생하여 시간 흐름이 잘 반영되는지를 검증한 결과입니다.

<img width="2363" height="681" alt="image" src="https://github.com/user-attachments/assets/360a3f82-71fb-4400-8d52-26daadc0fc65" />

### 🔍 신호 정의

- `c_state[3:0]` : FSM의 현재 상태  
- `tick_cnt_reg[10:0]` : 시간 흐름을 나타내는 내부 카운터 (1tick = 10us)  
- `w_tick` : 일정 주기마다 1이 되는 Tick Pulse (FSM 전이 조건)

### 🧭 시뮬레이션 동작 흐름

| 시점 (us) | 상태 전이 | 설명 |
|----------|----------------|------|
| 18,980 ~ 19,000 | `1 → 2` | FSM은 START(1) 상태에서 1900 tick 도달 후 WAIT(2) 상태로 전이됨 |
| 19,010           | `2 → 3` | 2번 상태(WAIT)에서 **3 tick 후**, SYNC_L(3)로 전이 |


### ✅ 검증 결과

- FSM이 **START → WAIT → SYNC_L → SYNC_H**로 정확한 타이밍에 따라 전이됨을 확인
- `tick_cnt_reg`와 `w_tick` 신호가 **정상적으로 작동**하며 시간 제어를 정확히 수행함
- DHT11 프로토콜 타이밍 요구 사항과 부합하는 결과

---
## 🔁 Start Response & State 전이 Verification (STATE 3 → 5)

> **DHT11의 응답 신호 수신 과정과 FSM 상태 전이**를 나타냅니다.  
> 특히 `dht11_io`의 응답 변화와 `tick` 발생 조건을 기반으로 FSM이 **State 3 → 4 → 5**로 **정확히 전이됨을 확인**할 수 있습니다.

<img width="2358" height="681" alt="image" src="https://github.com/user-attachments/assets/ea61d767-6eb5-4924-803e-65ff6fbb6d73" />

### 🔍 신호 정의

- `dht11_io_reg` : DHT11 센서와 통신 중인 IO 입력 값 (레지스터로 동기화된 값)
- `c_state[3:0]` : FSM의 현재 상태
- `w_tick` : 일정 시간 간격으로 생성되는 타이밍 신호 (샘플링 타이밍 동기화용)    

### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)         | 조건                          | 상태 전이               | 설명                                  |
|-------------------|-------------------------------|--------------------------|---------------------------------------|
| 약 19,130.025     | `dht11_io == 1` & `tick` 발생 | `3 → 4`       | 센서 응답 시작 (LOW → HIGH 감지)      |
| 약 19,210.025     | `dht11_io == 0` & `tick` 발생 | `4 → 5`       | 센서 응답 종료 (HIGH → LOW 감지)      |


### ✅ 검증 결과

- FSM은 아래 조건을 모두 만족할 때 상태 전이를 수행:
  - **입력 변화 (`dht11_io`)**
  - **샘플링 타이밍 (`w_tick == 1`)**
- 해당 조건 만족 시 FSM은 `3 → 4 → 5`로 전이
- 이후 `c_state`는 5 (DATA_SYNC) 상태에서 `dht11_io`의 상승/하강 에지에 반응하며 `5 ↔ 6` 반복 (데이터 수신)
- **40비트 데이터 수신 준비가 정상적으로 수행되고 있음이 검증됨**

---

## 🔁 DATA 전송 과정 및 FSM 반복 전이 검증 (STATE 5 ↔ 6)

> 아래 시뮬레이션은 **DHT11 센서의 40비트 데이터 수신 과정**에서,  
> FSM이 `DATA_SYNC (5)`와 `DATA_DETECT (6)` 사이를 정상적으로 반복 전이함을 보여줍니다.  
> 이는 `dht11_io`의 에지 변화와 `tick` 신호에 따라 정확히 반응하고 있음을 나타냅니다.

<img width="2267" height="679" alt="image" src="https://github.com/user-attachments/assets/818eecc8-f01b-48ce-9dd8-31dddc9a9791" />

### 🔍 신호 정의

- `dht11_io_reg` : 센서로부터 수신되는 디지털 신호 (입력 캡처용 레지스터)
- `c_state[3:0]` : FSM의 현재 상태
- `w_tick` : 일정 시간 간격으로 발생하는 타이밍 신호 (샘플링 기준)
- `data_cnt_reg` : 수신할 데이터 비트 수 카운터 (0이 될 때까지 FSM 전이 반복) 

### 🧭 시뮬레이션 상태 전이 흐름

| 시점 (μs)         | 조건                          | 상태 전이                   | 설명                                         |
|-------------------|-------------------------------|------------------------------|----------------------------------------------|
| 약 19,260.025     | `dht11_io == 1` & `tick` 발생 | `5 → 6`           | 센서 데이터 비트의 시작 감지 (LOW → HIGH)    |
| 이후 반복 구간     | `tick` 주기 + 에지 변화 감지  | `6 ↔ 5` 반복 전이 | 40비트 수신을 위한 고/저 신호 측정 반복     |

### ✅ 검증 결과

- FSM은 `DATA_SYNC (5)` 상태에서 `dht11_io`가 `1`로 상승하고 `tick`이 발생할 때 `DATA_DETECT (6)`으로 전이
- 이후 HIGH 지속 시간에 따라 데이터를 해석한 뒤 다시 `STATE 5`로 돌아감
- 이 과정은 `data_cnt_reg == 0`이 될 때까지 **총 40비트 동안 반복 수행**
- 시뮬레이션을 통해 FSM이 정확히 `5 → 6 → 5 → 6 ...` 형태로 정상 동작함이 검증됨

---

## 🔁 데이터 수신 종료 및 Checksum 검사 (STATE 6 → 7)

> 아래 시뮬레이션은 DHT11 센서로부터 **40비트 데이터를 모두 수신한 이후**,  
> FSM이 체크섬 검사를 수행하고, 이를 통과하면 최종적으로 `STOP (STATE 7)` 상태로 전이되는 과정을 보여줍니다.

<img width="2267" height="680" alt="image" src="https://github.com/user-attachments/assets/3e00fab4-17e5-44ca-8c77-f7aafc990c01" />

### 🔍 신호 정의

- `c_state[3:0]`: FSM 현재 상태
- `data_cnt_reg[5:0]`: 40비트 수신 카운터
- `dht11_valid`: checksum이 유효할 경우 High
- `dht11_done`: 전체 수신 완료 플래그


### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)         | 조건                        | 상태 전이         | 설명                                        |
|-------------------|-----------------------------|--------------------|---------------------------------------------|
| 약 23,000         | `data_cnt_reg == 0` 도달    | `6` 유지 | 마지막 비트 수신 후 체크섬 판단 대기         |
| 이후 Tick 시점     | Checksum 결과 유효           | `6 → 7` | 수신된 40비트 중 마지막 8비트와의 일치 확인  |


### ✅ 검증 결과

- `data_cnt_reg`가 0이 되면 40비트 수신이 완료됨
- 이후 FSM은 내부적으로 **Checksum 검사**를 수행
- 검사 통과 시 `dht11_valid = 1`이 출력되며 FSM은 STOP 상태(7)로 전이
- 해당 시뮬레이션을 통해 전체 수신 과정이 정상적으로 종료되는 것을 확인할 수 있음

---

## 🔁 통신 종료 대기 및 초기화 전이 (STATE 7 → 0)

> 아래 시뮬레이션은 **DHT11 센서 통신이 완료된 이후**,  
> FSM이 일정 시간(4 tick = 40μs) 대기한 후 **IDLE 상태(0)** 로 전이되는 과정을 보여줍니다.  
> 이 구간은 센서 응답 안정화 및 FSM 초기화를 위한 딜레이 타이머 역할을 수행합니다.

<img width="2271" height="681" alt="image" src="https://github.com/user-attachments/assets/da08b490-35c6-4e70-8538-4d6a0ec22bb3" />

### 🔍 신호 정의

- `w_tick`: 데이터 수신 시각 제어용 틱 신호 (1bit)
- `c_state[3:0]`: FSM의 현재 상태
  - 7: Checksum 비교 상태
  - 0: Idle 상태 (FSM 종료 및 대기)
  - `tick_cnt_reg` : FSM 내 내부 tick 카운터 (STOP 상태에서 4까지 count 후 초기화 전이)

### 🧭 시뮬레이션 상태 전이 흐름

| 시점 (μs)       | 조건                        | 상태 전이         | 설명                                      |
|------------------|-----------------------------|--------------------|-------------------------------------------|
| 약 23,150.000    | `w_tick` 4회 발생 시점 도달 | STATE 7 → STATE 0 | STOP 상태에서 40μs 대기 후 FSM 리셋 (IDLE) |


### ✅ 검증 결과

- FSM이 `STOP(7)` 상태에서 내부 `tick_cnt`를 증가시키며 4 tick(40μs) 동안 대기
- 4번째 `w_tick` 발생 시점에서 `c_state`가 `0`으로 전이됨 (IDLE 상태 진입)
- 이는 FSM이 **통신 종료 이후 안정적으로 초기 상태로 복귀**함을 의미하며,  
  다음 DHT11 통신 주기를 기다릴 준비가 되었음을 시뮬레이션으로 확인할 수 있음

---

## 🔄 DHT11 40bit 데이터 수신 흐름 Verification

> 아래 시뮬레이션 파형은 DHT11 센서로부터 순차적으로 40bit 데이터를 수신하여  
> `data_reg[39:0]`에 저장하는 과정을 보여줍니다.  
> 이 데이터는 습도(16bit), 온도(16bit), 체크섬(8bit)으로 구성된 총 5바이트(40bit)입니다.

<img width="2269" height="681" alt="image" src="https://github.com/user-attachments/assets/a8aeae06-91fe-4c93-bec2-a445d3330174" />

### 🧾 주요 포인트

- FSM 상태가 `c_state = 5 (DATA_SYNC)`일 때 센서 응답 신호를 감지하고, `c_state = 6 (DATA_DETECT)`에서 HIGH 유지 시간 측정을 통해 bit값 결정
- `w_tick`이 발생할 때마다 1bit씩 `data_reg[39:0]`에 **좌측으로 shift-in**되어 저장됨
- 파형 상 `data_reg` 값이 점차 `1010...`으로 채워지는 과정을 통해 센서로부터의 **정상적인 비트 수신이 진행 중임을 확인 가능**

### 🧪 데이터 저장 예시

- 예시: `data_reg = 1010000000000000000000000000000000000000`  
  → 상위 비트부터 하나씩 shift-in 되며 저장 중임을 의미

### ✅ 검증 결과

- `data_reg[39:0]`는 FSM 상태 `STATE 5 ↔ 6` 반복 중 `w_tick` 상승 에지를 기준으로 정확히 1bit씩 채워짐
- 이를 통해 FSM이 **정상적으로 40bit 수신 루틴을 수행**하고 있음을 입증
- 최종적으로 `data_cnt_reg == 0` 도달 시 → FSM은 **Checksum 검사 단계 (STATE 6 → 7)** 로 전이

---

# 📟 CMD_RX UART 명령 수신 시스템 동작 결과

> UART를 통해 명령어를 전송하고, 이에 따른 초음파 거리 센서 및 온습도 센서의 결과를 `ComPortMaster`에서 확인한 예시입니다.

### 📏 초음파 거리 센서 (`SR\n` 명령)

- **명령어 전송**: `SR\n`
- **기능 설명**: 초음파 센서(SR04)를 트리거하여 거리(cm) 측정
- **실행 화면**:

<img width="600" alt="image" src="https://github.com/user-attachments/assets/84aabfe6-155f-445e-979d-ca29c0890819" />

### 🌡️ 온습도 센서 (`DHT\n` 명령)

- **명령어 전송**: `DHT\n`
- **기능 설명**: DHT11 센서를 통해 온도(C) 및 습도(%) 측정
- **실행 화면**:

<img width="600" alt="image" src="https://github.com/user-attachments/assets/e17dabb3-5f59-4ee3-9b41-093ac4853076" />

---

# ⏯️ 시연 영상

> 시계, 초음파 센서, 온습도 센서를 통합한 시스템의 시연 영상입니다.          
출력 결과는 FND와 ComPort Master를 통해 확인할 수 있습니다.

### 📏 초음파 거리 센서

https://github.com/user-attachments/assets/dcd22885-6388-446f-8a79-c8168cddd4c7

### 🌡️ 온습도 센서 

https://github.com/user-attachments/assets/d4aef482-1400-46c6-b2e4-4b402ccb5744






