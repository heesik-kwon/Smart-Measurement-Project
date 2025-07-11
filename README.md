# 💻 Smart Measurement System
> 멀티 센서 기반의 스마트 계측 시스템 프로젝트입니다.     
Verilog로 stopwatch & watch, HC-SR04 초음파 센서, DHT11 온습도 센서, FIFO & UART 기반 데이터 통신 기능을 직접 설계하고 검증하였습니다.

---

## 📌 프로젝트 개요

| 항목             | 내용                                                   |
|------------------|--------------------------------------------------------|
| **⏱️ 수행 기간** | 2024.05.30 ~ 2024.06.01                               |
| **🖥️ 개발 환경**  | Vivado, VSCode                                        |
| **📦 플랫폼**     | Basys3 FPGA 보드 (Xilinx Artix-7)                    |
| **💻 언어**       | Verilog HDL                                           |
| **📡 통신**       | UART 통신       |
| **🧪 센서**       | HC-SR04 (초음파 거리 센서), DHT11 (온습도 센서)       |

---

## 🎯 주요 기능

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

## 🧩 시스템 블록 다이어그램 (System Block Diagram)

> 시스템의 전체 동작 흐름을 나타낸 블록 다이어그램으로, 입력 신호부터 제어 및 출력까지의 구조를 모듈 단위로 표현하였습니다.

<img src="https://github.com/user-attachments/assets/7c95914c-8028-47a9-bafc-100f97f3f380" width="600"/>

---

## 🧩 Top-Level RTL Schematic (Vivado)

> Vivado를 통해 생성한 Top-Level RTL Schematic입니다.   
각 하드웨어 모듈 간의 실제 연결 관계를 기반으로 회로 구조를 시각화하여, 시스템의 데이터 흐름과 제어 경로를 명확하게 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/eb8632bb-1630-4219-835c-05fe5ffe4312" width="800"/>

---

## 🧩 HC-SR04 RTL Schematic (Vivado)

> Vivado에서 구성한 초음파 거리 측정 시스템의 RTL Schematic입니다.   
버튼 입력으로 HC-SR04 센서를 트리거하고, 측정된 거리를 FND에 표시하며 UART를 통해 decimal 형식으로 송신합니다.

<img src="https://github.com/user-attachments/assets/05de31f7-171f-4850-987a-452831baa44f" width="800"/>

---

## 🧩 Basys3 보드 LED 및 SW 기능 정리

> 프로젝트에서 사용된 **Basys3 FPGA 보드의 LED와 SWITCH 역할**을 정리한 표입니다.  
LED는 실시간 상태 출력 용도로 사용되며, SW는 모드 전환 및 기능 제어에 활용됩니다.

<img src="https://github.com/user-attachments/assets/805d0462-ad33-4cca-b066-46175dcb481d" width="750"/>

---

## 🔧  HC-SR04 + UART 검증 및 시뮬레이션 (Verification & Simulation)

> 각 모듈의 동작을 Testbench를 통해 검증하고, 파형 시뮬레이션을 통해 신호 변화와 타이밍을 확인하였습니다.

### 🔄 Timing Verification of Trigger Pulse

> HC-SR04 초음파 센서는 40kHz 주파수로 8사이클의 초음파를 송신하며, 이때 걸리는 시간은 약 **200μs**입니다.

<img src="https://github.com/user-attachments/assets/466f310e-62be-4595-9ef1-6715ebf31d5e" width="850"/>

- **센서 사양 계산**  
  40kHz → 25μs 주기  
  8사이클 → 25μs × 8 = **200μs**

- **시뮬레이션 결과**  
  `trig` 이후 `echo` 출력까지 약 **239μs** 소요 → 정상 범위 내

### ✅ 검증 결과 요약

- 시뮬레이션 결과에서는 `trig` 신호 발생 후, `echo` 신호가 약 **239μs** 후에 High가 되는 것을 확인할 수 있습니다.  
- 이는 센서 데이터시트상의 사양과 유사하며, **정상적으로 트리거 동작이 수행됨을 검증**할 수 있습니다.

---

### 🔄 High Level Detecting Verification

> 아래는 `high_level_detector` 모듈의 기능을 검증하기 위한 Testbench 시뮬레이션 결과입니다.     
> 이 모듈은 echo 신호의 high 레벨 구간을 감지하고, 그 구간의 시작과 종료 시점에 각각 high_level_flag와 done 신호를 갱신합니다.

<img src="https://github.com/user-attachments/assets/875b5226-c357-4751-a0d4-f21fae461656" width="850"/>

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

### 🔄 Calculator Verification

> HC-SR04 초음파 센서의 `echo` 신호가 high인 구간 동안 클럭을 카운트하여 거리를 계산하는 로직입니다.

<img src="https://github.com/user-attachments/assets/78eef7f2-433a-4606-bd5b-e2be364a5822" width="850"/>

1. **거리 측정 시작**
   - `high_level_flag = 1`인 동안 `count`가 클럭마다 증가합니다.

2. **측정 종료 감지**
   - `done` 신호가 1이 되면:
     - `distance <= count / 58` (단위: cm)
     - `dist_done <= 1`로 설정됨
     - `count`는 초기화됩니다

3. **거리 계산 예시**
   - 예: `count = 64`일 때 → `distance = 64 / 58 ≒ 1cm`

### ✅ 검증 결과 요약

- 이 시뮬레이션은 거리 측정 타이밍과 계산 결과가 정확하게 동작하고 있음을 검증합니다.

---

### 🔄 SR04 Controller Verification

> 초음파 거리 측정 모듈의 전체 동작을 검증한 시뮬레이션 결과입니다. `sr04_controller` 모듈은 `tick_gen`, `start_trigger`, `high_level_detector`, `calculator`로 구성되어 있으며, 트리거부터 거리 계산까지의 흐름이 정확히 수행되는지를 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/3b4714b9-84cc-4914-8007-a8afee53aab2" width="850"/>

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

### ✅ 검증 결과 요약

- `trig`, `echo`, `done`, `distance` 신호들이 순차적으로 정상 동작함을 파형을 통해 확인하였습니다.
- 계산된 거리 값 또한 정확히 **4cm**로 출력되는 것을 확인할 수 있었고, **SR04 초음파 모듈 제어 및 거리 측정 알고리즘이 정상 작동함을 시뮬레이션으로 입증**하였습니다.

---

### 🔄 UART Transmission Flow Verification

> 거리 측정이 완료되면 `"Distance = XXXXcm\r\n"` 포맷으로 UART를 통해 메시지가 전송됩니다. 아래는 전체 UART 송신 흐름 시뮬레이션입니다.

<img src="https://github.com/user-attachments/assets/74e1b9f9-f24b-4d6e-ba6f-53f60c58b412" width="850"/>

#### 동작 순서

1. **`dist_done ↑`**  
   거리 계산 완료 시 신호 발생 → ASCII 변환 트리거됨

2. **`push_tx ↑`**  
   UART로 전송을 시작함을 알림 → `tx_din`에 전송할 문자 로드됨

3. **`tx_din` 출력**  
   `"Distance = 006cm\r\n"` 문자열이 한 바이트씩 출력됨

4. **`tx_busy` / `tx_done`**  
   전송 중 `tx_busy = HIGH`, 완료 시 `tx_done = HIGH` 한 클럭

5. **🔁 반복**  
   거리 갱신 시마다 위 과정이 반복되어 UART로 전송됨

### ✅ 검증 결과 요약

#### 송신 문자열 예시

```
Distance = 006cm\r\n
```

- 앞자리 `0`은 공백(0x20)으로 대체됨
- 총 17바이트 전송
- `dist_uart_sender`가 ASCII 4자리와 고정 문자열을 병합하여 전송 처리

---

## 🌡️ DHT11 온습도 센서 통신 프로토콜

> DHT11 센서는 단일 신호선(Single-Bus)을 사용해 MCU와 데이터를 주고받습니다.     
DHT11의 통신 시퀀스를 시간 흐름에 따라 단계별로 나타낸 다이어그램과 각 구간의 신호 상태입니다.

<img src="https://github.com/user-attachments/assets/64bec993-3103-4d77-aaab-ed1c15fbbd64" width="800"/>

---


## 📦 DHT11 시스템 블록 다이어그램

> Basys3 보드를 기반으로 구성된 **DHT11 온습도 센서 시스템의 블록 다이어그램**입니다.    
사용자의 버튼 입력을 받아 DHT11 센서와 통신하며, 측정된 온습도 데이터를 7세그먼트(FND)에 출력하는 구조입니다.

<img src="https://github.com/user-attachments/assets/65c80b03-df40-40d3-805e-bf9e9b3efe83" width="750"/>

---

## 🔄 DHT11 FSM 상태 전이도

> DHT11 센서 통신을 위한 Finite State Machine (FSM) 상태 전이도입니다.  
FSM은 총 8개의 상태(IDLE ~ STOP)를 순차적으로 거치며, Start Signal → 응답 수신 → 데이터 수신 과정을 수행합니다.

<img src="https://github.com/user-attachments/assets/03a07d73-6a46-4716-aeba-7ee6d14a6fb6" width="750"/>

## 🔧 DHT11 + UART 검증 및 시뮬레이션 (Verification & Simulation)

> 본 시뮬레이션은 `TOP_DHT11` 모듈을 기반으로 DHT11 온습도 센서 FSM이 정상적으로 시작되고 상태가 전이되는지를 검증한 것입니다.  
아래 파형을 통해 FSM의 시작 조건 및 `start` 입력 동작 여부를 확인할 수 있습니다.

### 🔁 FSM Start Trigger Verification

> 아래는 DHT11 센서 FSM의 `start` 신호가 입력되었을 때, FSM 상태(`c_state`)가 `IDLE(0)`에서 `START(1)`로 전이되는 과정을 확인한 시뮬레이션 결과입니다.

<img src="https://github.com/user-attachments/assets/2c555e4b-866d-44c5-ad75-c70a41173455" width="850"/>

- **FSM 초기 상태**
  - `c_state = 0` → IDLE 상태에서 대기 중
  - `dht11_io`는 기본적으로 High 상태를 유지
  - `start` 신호가 LOW였다가 45,000ns 부근에서 HIGH(1)로 전환

- **FSM 상태 전이**
  - `start`가 1로 올라간 이후, FSM은 `c_state = 1` (START 상태)로 전이됨
  - 이는 DHT11 FSM 정의 상 `start` 신호가 트리거 역할을 한다는 점을 시뮬레이션으로 검증한 결과

### ✅ 검증 결과 요약

- `start` 신호가 정상적으로 인식되며 FSM이 IDLE → START로 전이됨
- FSM이 시작되면 이후 `START → WAIT → SYNC_L → SYNC_H` 순서로 진행되며 DHT11 통신 프로토콜을 따라 동작함
- **정상적인 상태 전이 시퀀스가 이루어졌음을 시뮬레이션으로 입증**

---

### 🔁 FSM Transition & Tick Timing Verification

> 아래 시뮬레이션은 `DHT11 FSM`이 START → WAIT → SYNC_L 상태로 **정상적으로 전이**되는지,  
그리고 내부 `tick`이 적절히 발생하여 시간 흐름이 잘 반영되는지를 검증한 결과입니다.

<img width="2363" height="681" alt="image" src="https://github.com/user-attachments/assets/360a3f82-71fb-4400-8d52-26daadc0fc65" />

### 🔍 신호 정의

- `c_state[3:0]` : FSM의 현재 상태  
- `tick_cnt_reg[10:0]` : 시간 흐름을 나타내는 내부 카운터 (1tick = 10us)  
- `w_tick` : 일정 주기마다 1이 되는 Tick Pulse (FSM 전이 조건)

### 🧭 시뮬레이션 동작 흐름

| 단계 | 시점 (us) | 상태 전이 | 설명 |
|------|----------|----------------|------|
| ①    | 18,980 ~ 19,000 | `1 → 2` | FSM은 START(1) 상태에서 1900 tick 도달 후 WAIT(2) 상태로 전이됨 |
| ②    | 19,010           | `2 → 3` | 2번 상태(WAIT)에서 **3 tick 후**, SYNC_L(3)로 전이 |


### ✅ 검증 결과 요약

- FSM이 **START → WAIT → SYNC_L → SYNC_H**로 정확한 타이밍에 따라 전이됨을 확인
- `tick_cnt_reg`와 `w_tick` 신호가 **정상적으로 작동**하며 시간 제어를 정확히 수행함
- DHT11 프로토콜 타이밍 요구 사항과 부합하는 결과

---

### 🔁 Start Response & State 전이 Verification (STATE 3 → 5)

> 아래 시뮬레이션은 **DHT11의 응답 신호 수신 과정과 FSM 상태 전이**를 나타냅니다.  
> 특히 `dht11_io`의 응답 변화(0→1, 1→0)에 따라 FSM이 **State 3 → 4 → 5**로 정상 전이됨을 보여줍니다.

<img width="2358" height="681" alt="image" src="https://github.com/user-attachments/assets/ea61d767-6eb5-4924-803e-65ff6fbb6d73" />

### 🔍 신호 정의

- `dht11_io_reg`: 센서와 통신 중인 I/O 레지스터
- `c_state[3:0]`: FSM의 현재 상태
  - 3: 센서 응답 대기
  - 4: 응답 신호 감지 (0→1)
  - 5: 응답 신호 종료 감지 (1→0)

### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)         | 신호 변화                  | 상태 전이              | 설명                                          |
|-------------------|----------------------------|-------------------------|-----------------------------------------------|
| 약 19,130.025     | dht11_io: 0 → 1          | STATE 3 → STATE 4     | 센서 응답 시작 (LOW → HIGH)명                           |
|-------------------|--------------|------------------------------------------|
| 약 19,260.025     | 5 → 6        | dht11_io == 0, w_tick == 1 만족       |
| 이후 반복 구간     | 6 → 5 → 6 ... | dht11_io 상승 에지 감지 반복            |


### ✅ 검증 결과 요약

- FSM이 `c_state = 5 → 6 → 5 → 6` 형태로 반복 전이되며  
  각 비트의 rising/falling edge에 따라 정상적으로 데이터를 받아옴
- `dht11_io`가 0 → 1로 상승하면 State 5 (DATA_SYNC),  
  이후 일정 시간 동안 유지되면 State 6 (DATA_DETECT)로 전이됨
- **40비트 데이터 수신 과정이 정상적으로 수행되고 있음을 시뮬레이션을 통해 입증**

---

### ✅ FSM 종료 및 Checksum 판별 Verification

> 아래 시뮬레이션은 DHT11 FSM의 마지막 단계인 **STOP(7)** 상태로의 전이와  
> 그 직전에 수행되는 **checksum 판별 동작**을 검증한 결과입니다.

<img width="2267" height="680" alt="image" src="https://github.com/user-attachments/assets/3e00fab4-17e5-44ca-8c77-f7aafc990c01" />


### 🔍 신호 정의

- `c_state[3:0]`: FSM 현재 상태
- `data_cnt_reg[5:0]`: 40비트 수신 카운터
- `dht11_valid`: checksum이 유효할 경우 High
- `dht11_done`: 전체 수신 완료 플래그

### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)         | 이벤트                 | 설명                                                |
|-------------------|------------------------|-----------------------------------------------------|
| 약 23,000.000     | `data_cnt_reg == 0`    | 40비트 수신 완료 후 0으로 초기화됨                    |
| 약 23,010.000     | `c_state: 6 → 7`       | FSM이 STOP 상태로 진입                               |
| 직후              | `dht11_valid == 1`     | 내부 checksum 검증 결과가 유효함                     |
| 직후              | `dht11_done == 1`      | 수신 완료 신호가 High로 설정됨                      |


### ✅ 검증 결과 요약

- FSM은 총 **40비트 수신이 완료되면 `data_cnt_reg == 0`** 조건에 따라  
  **STOP(7)** 상태로 정상 전이됨
- STOP 상태 진입 시점에서 **checksum이 유효한 경우 `dht11_valid == 1`**이 되고  
  동시에 전체 수신 완료를 의미하는 `dht11_done == 1`이 출력됨
- 이는 전체 FSM의 마지막 종료 및 검증 절차가 **정상 동작**함을 시뮬레이션을 통해 입증한 것

---

### 🔄 Final State Transition & Tick Verification (STATE 7 → 0)

> 아래 시뮬레이션은 DHT11 통신 FSM이 데이터를 모두 수신한 후,  
> 최종적으로 **STATE 7 → STATE 0 (IDLE)** 상태로 정상 복귀하는 과정을 보여줍니다.

<img width="2271" height="681" alt="image" src="https://github.com/user-attachments/assets/da08b490-35c6-4e70-8538-4d6a0ec22bb3" />


### 🧭 시뮬레이션 동작 흐름

| 시점 (μs)         | 신호 변화                  | 상태 전이            | 설명                                            |
|-------------------|----------------------------|-----------------------|-------------------------------------------------|
| 약 23,100 ~ 이후  | `w_tick` 4번 발생          | STATE 7 유지          | checksum 검증 상태 (tick 4회 발생)              |
| 약 23,150         | `c_state`: 7 → 0           | STATE 7 → STATE 0     | FSM 초기화 완료, 정상 종료                      |


### 🔍 신호 정의

- `w_tick`: 데이터 수신 시각 제어용 틱 신호 (1bit)
- `c_state[3:0]`: FSM의 현재 상태
  - 7: Checksum 비교 상태
  - 0: Idle 상태 (FSM 종료 및 대기)


### ✅ 검증 결과 요약

- `w_tick` 신호가 4회 발생한 뒤, `c_state`가 0으로 복귀하여 FSM이 정상 종료됨을 확인
- 이는 **DHT11의 40bit 데이터 수신 및 체크섬 판별을 완료한 후의 정상 동작 흐름**을 의미함

---

### 🔄 DHT11 40bit 데이터 수신 흐름 Verification

> 아래 시뮬레이션 파형은 DHT11 센서로부터 순차적으로 40bit 데이터를 수신하여 `data_reg`에 저장하는 과정을 보여줍니다.
이 데이터는 습도(8bit), 온도(8bit), 체크섬(8bit)을 포함한 총 5바이트(40bit)입니다.

<img width="2269" height="681" alt="image" src="https://github.com/user-attachments/assets/a8aeae06-91fe-4c93-bec2-a445d3330174" />

### 🧾 주요 관찰 포인트

- `c_state = 5` 상태에서 데이터 수신 시작됨
- `w_tick` 발생 주기에 따라 **1bit씩 `data_reg[39:0]`에 Shift-in**
- 파형 상의 `data_reg` 변화 과정을 통해 정상적으로 bit 수신이 이루어지는 것을 확인


### 🧪 데이터 저장 예시

- 예: `data_reg = 1010000000000000000000000000000000000000`  
  → 상위 비트부터 1bit씩 shift 저장됨을 의미


### ✅ 검증 결과 요약

- `data_reg[39:0]`가 `w_tick`의 상승 에지를 기준으로 1bit씩 채워지는 것을 통해,
  - FSM의 상태 `5번(State 5)`에서 **정상적인 데이터 수신 흐름이 작동**함을 검증
- 이후 `data_cnt_reg == 39`일 때 `c_state → 6`으로 전이됨

---

## 📟 CMD_RX UART 명령 수신 시스템 동작 결과

> UART를 통해 명령어를 전송하고, 이에 따른 초음파 거리 센서 및 온습도 센서의 결과를 `ComPortMaster`에서 확인한 예시입니다.

### ✅ 초음파 거리 센서 (`SR\n` 명령)

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















