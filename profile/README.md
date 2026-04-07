# 🚗 차량 위험 상황 경고 및 대응 시스템

> **프로젝트**: Team Starex  
> **개발 기간**: 2026년  
> **상태**: ✅ 개발 진행 중

차량 내 위험 상황을 실시간으로 감지하고 운전자에게 경고하며 자동으로 대응하는 **임베디드-IoT 통합 안전 시스템**입니다. 센서로부터 입력을 수집하고, 중앙 ECU에서 데이터를 처리하며, 웹 기반 대시보드와 음향/시각 장치를 통해 실시간으로 안전 상황을 관리합니다.

## 👥 팀 소개

---

### 🔗 [⭐ 팀장 황선안](https://github.com/sernan96) - **CONTROL_ECU + HMI_WEB**

**담당 역할**:

- **중앙 제어**: 상태별 실시간 로그 핸들러, 시스템 타이머 구현
- **중앙 제어 ↔ 라즈베리파이**: CAN 송수신부
- **웹 대시보드**: CAN 백엔드 미들웨어, Node.js 서버, React 프론트엔드

---

### 🔗 [강현수](https://github.com/rgtuu) - **ACT_ECU (액추에이터 제어)**

**담당 역할**:

- 출력 제어 (스피커, 도어락, 트렁크, 브레이크/테일램프, 경고음, 안전벨트)
- 중앙 제어 ↔ 출력부 CAN 수신부 

---

### 🔗 [김정연](https://github.com/kjy0916) - **INPUT_ECU (센서 입력)**

**담당 역할**:

- 입력 처리 (홀센서 기반 악셀/브레이크, 가변 저항 핸들, 사용자 버튼 입력, 센서 )
- 입력부 → 중앙 제어 CAN 송신부
- 센서 입력 신호 안정화 로직

---

### 🔗 [이무석](https://github.com/Leemuseok) - **CONTROL_ECU (중앙 제어)**

**담당 역할**:

- 중앙 ECU 핵심 로직 (속도, 가속도 계산, 시스템 상태 판단)
- 송수신 데이터 형식 정의 & CAN 인터페이스 정의
- 중앙 제어 ↔ 출력부 CAN 송신부
- 입력부 → 중앙 제어 CAN 수신부

---

## V모델 기반 프로세스

<img width="1064" height="499" alt="image" src="https://github.com/user-attachments/assets/8a954a9f-a251-44a1-8871-5586882593d4" />


## 📋 시스템 구조

<img width="876" height="569" alt="image" src="https://github.com/user-attachments/assets/c1a205c1-782a-4cf8-9122-c6cb4a7c3a9a" />

**데이터 흐름**:

1. INPUT_ECU → CAN → CONTROL_ECU: 센서 데이터 수신
2. CONTROL_ECU → 분석/판정
3. CONTROL_ECU → CAN → ACT_ECU: 제어 신호 송신
4. CONTROL_ECU → CAN → 라즈베리파이: 상태/정보 송신
5. 라즈베리파이 (can_receiver.py) → Node.js 서버 → React 프론트엔드: WebSocket 실시간 전송

---

## ✨ 핵심 기능

### 🎯 센싱 및 데이터 수집 (INPUT_ECU)

- **ADC 센서**: 각종 아날로그 센서 신호 수집 (가속도, 속도, 조향각 등)
- **디지털 입력**: 버튼, 스위치 등 디지털 신호 처리
- **실시간 데이터**: CAN 버스를 통해 중앙 ECU(TC375)로 전송

### 🧠 중앙 처리 (CONTROL_ECU - TC375)

- **데이터 분석**: INPUT_ECU로부터 수집한 센서 데이터 분석
- **위험도 판정**: 상황 심각도에 따른 우선순위 결정
- **제어 신호 생성**: ACT_ECU로 제어 명령 송신
- **상태 정보 송신**: HMI_WEB(라즈베리파이)으로 현재 차량 상태 송신
- **이벤트 감지**:
  - 출발 시 조향각 정렬 여부 확인
  - 급격한 조향 감지
  - 무응답 상태 모니터링

### 🚨 대응 및 경고 (ACT_ECU)

- **시각 경고**: 상황 심각도에 따른 LED 제어 (빨강/주황/노랑/초록)
- **음향 경고**: 버저를 통한 다양한 경고음 패턴
- **오디오 알림**: MP3 오디오 재생으로 음성 경고
- **자동 제어**: 위험 상황 시 서보 모터 등으로 긴급 대응

### 📊 웹 대시보드 (HMI_WEB)

- **실시간 모니터링**: 원형 속도계, 조향각 디스플레이
- **상태 표시**: 주행 상태(유휴/주행/정지), 문 잠금 상태
- **안전도 표지**: 4단계 심각도 시각화
- **이벤트 로그**: 발생한 모든 안전 이벤트 기록
- **반응형 UI**: 모든 디바이스에서 접속 가능

---

## 🏗️ 프로젝트 구조

### 📁 ACT_ECU - 액추에이터 제어 유닛 (담당: 강현수)

**목표**: 차량의 모든 액추에이터 제어 및 유저 인터페이스 디바이스 구동

**주요 구성요소**:

```
ACT_ECU/
├── App/              # 애플리케이션 로직
│   ├── led/         # LED 제어 로직
│   ├── buzzer/      # 버저 제어 로직
│   ├── servo/       # 서보 모터 제어 로직
│   ├── dfplayer/    # MP3 플레이어 제어 로직
│   └── scheduler/   # 작업 스케줄링
├── Driver/          # 하드웨어 드라이버
├── Libraries/       # Infineon iLLD 라이브러리
└── Common/          # 공통 설정 및 타입
```

**개발 보드**: Hitex Shield Buddy (TC275 기반)  
**마이크로컨트롤러**: Infineon TriCore TC275 (3개 CPU 코어)  
**IDE**: Aurix Development Studio  
**컴파일러**: TASKING  
**주요 기능**:

- LED 상태 표시등 제어
- 경고 음향 재생 (버저, MP3)
- 서보 모터를 통한 기계 제어
- CAN 통신으로 위험 신호 수신

---

### 📁 INPUT_ECU - 입력처리 유닛 (담당: 김정연)

**목표**: 차량 내 모든 센서 신호를 수집하고 디지털화

**주요 구성요소**:

```
INPUT_ECU/
├── Drv_AdcInput/   # ADC 센서 드라이버
├── Drv_Button/     # 버튼 입력 드라이버
├── Can_TxInput/    # CAN 메시지 전송
├── App_InputEcu/   # 애플리케이션 로직
├── Configurations/ # 설정 파일
└── Libraries/      # Infineon iLLD
```

**개발 보드**: Hitex Shield Buddy (TC275 기반)  
**마이크로컨트롤러**: Infineon TriCore TC275  
**IDE**: Aurix Development Studio  
**컴파일러**: TASKING  
**주요 기능**:

- 아날로그 센서 신호 수집 (ADC)
- 버튼 입력 감지
- 수집된 데이터를 CAN 메시지로 변환 및 전송
- 신호 필터링 및 전처리

---

### 📁 CONTROL_ECU - 중앙 제어 유닛 (담당: 이무석, 황선안)

**목표**: 전체 시스템의 두뇌 역할 - INPUT_ECU 데이터 수신, 분석, 제어 신호 생성

**개발 보드**: TC375 Lite Kit (TC375 기반)  
**마이크로컨트롤러**: Infineon TriCore TC375  
**IDE**: Aurix Development Studio  
**컴파일러**: TASKING

**주요 기능**:

- INPUT_ECU로부터 CAN 통신으로 센서 데이터 수신
- 수집한 데이터 분석 및 위험도 판정
- ACT_ECU(TC275)로 CAN 제어 신호 송신
- 라즈베리파이(HMI_WEB)로 CAN으로 상태 정보 송신
- 실시간 이벤트 생성 및 판정

---

### 📁 라즈베리파이 (HMI_WEB) - 웹 기반 대시보드 & 미들웨어 (담당: 황선안)

**목표**: 운전자를 위한 실시간 모니터링 인터페이스 + CONTROL_ECU와의 통신 중개

**런타임 환경**: 라즈베리파이 4 모델 B

**구성요소**:

```
HMI_WEB/
├── backend/
│   ├── server.js           # Express 서버 메인
│   ├── can_receiver.py     # 미들웨어: TC375의 상태정보 수신 및 처리
│   ├── src/
│   │   ├── handlers/       # 상태정보 핸들러
│   │   ├── routes/         # API 라우트
│   │   ├── services/       # 비즈니스 로직
│   │   └── utils/          # 유틸리티
│   └── __tests__/          # 테스트
└── frontend/
    ├── src/
    │   ├── components/     # UI 컴포넌트
    │   ├── contexts/       # React Context
    │   ├── hooks/          # Custom Hooks
    │   ├── services/       # API/WebSocket 서비스
    │   └── utils/          # 유틸리티
    ├── __tests__/          # 테스트
    └── vite.config.js      # Vite 설정
```

**아키텍처**:

- **can_receiver.py**: CONTROL_ECU(TC375)로부터 CAN으로 상태 정보를 수신하여 Node.js 백엔드로 전달
- **Node.js 백엔드**: Express 서버로 WebSocket을 통해 실시간 데이터 스트리밍
- **React 프론트엔드**: 웹 기반 대시보드로 차량 상태를 시각화

**주요 기능**:

- 실시간 속도계 (0-120 km/h)
- 조향각 디스플레이 (-90°~+90°)
- 주행 상태 표시 (IDLE/DRIVING/STOPPED)
- 4단계 안전도 상태 (NORMAL/WARNING/CRITICAL/FATAL)
- 이벤트 로그 및 알림
- 문 잠금 상태 표시
- CONTROL_ECU(TC375)로부터 CAN으로 수신한 상태 정보 디스플레이

---

## 🔧 기술 스택

### 임베디드 시스템

| 구성                      | 기술                                        |
| ------------------------- | ------------------------------------------- |
| **액추에이터(ACT_ECU)**   | Infineon TriCore TC275 + Hitex Shield Buddy |
| **입력처리(INPUT_ECU)**   | Infineon TriCore TC275 + Hitex Shield Buddy |
| **중앙제어(CONTROL_ECU)** | Infineon TriCore TC375 + TC375 Lite Kit     |
| **IDE**                   | Aurix Development Studio                    |
| **컴파일러**              | TASKING                                     |
| **프레임워크**            | Infineon iLLD (Low-Level Drivers)           |
| **통신**                  | CAN 버스                                    |
| **언어**                  | C                                           |

### 임베디드 시스템

| 모듈                        | 구성                       | 기술                                |
| --------------------------- | -------------------------- | ----------------------------------- |
| **ACT_ECU** (출력 제어)     | TC275 + Hitex Shield Buddy | Aurix Studio, TASKING, iLLD, C, CAN |
| **INPUT_ECU** (입력 처리)   | TC275 + Hitex Shield Buddy | Aurix Studio, TASKING, iLLD, C, CAN |
| **CONTROL_ECU** (중앙 제어) | TC375 + TC375 Lite Kit     | Aurix Studio, TASKING, iLLD, C, CAN |

### 웹/IoT 시스템 (라즈베리파이 4 + Node.js)

| 계층             | 기술                           | 역할                              |
| ---------------- | ------------------------------ | --------------------------------- |
| **CAN 미들웨어** | Python 3 (can_receiver.py)     | TC375로부터 상태정보 수신 및 중개 |
| **백엔드**       | Node.js, Express.js, Socket.IO | 데이터 처리 및 WebSocket 스트리밍 |
| **프론트엔드**   | React 18, Vite, CSS3           | 차량 상태 시각화 및 UI            |
| **통신**         | WebSocket (Socket.IO)          | 실시간 양방향 통신                |

---

## 🚀 빠른 시작

### 📋 필수 요구사항

**임베디드 개발** (ACT_ECU, INPUT_ECU, CONTROL_ECU):

- Aurix Development Studio IDE
- TASKING 컴파일러
- Infineon iLLD 라이브러리 (TC275용, TC375용)
- Hitex Shield Buddy 개발 보드 (TC275용)
- TC375 Lite Kit 개발 보드 (TC375용)

**HMI_WEB 및 웹 시스템** (라즈베리파이):

- 라즈베리파이 4 모델 B (또는 동급 이상)
- Python 3+ (can_receiver.py 미들웨어용)
- Node.js 14+ 이상
- npm 또는 yarn
- CAN 인터페이스 (라즈베리파이용 GPIO/SPI CAN 보드)
- 최신 웹 브라우저

### 🏗️ 빌드 및 설치

#### 1️⃣ 임베디드 펌웨어 빌드 (ACT_ECU, INPUT_ECU, CONTROL_ECU)

**ACT_ECU (Hitex Shield Buddy)**:

```bash
cd ACT_ECU
# Aurix Development Studio에서 열기
# TC275_ACT_tb TriCore Debug 프로젝트 선택
# Project > Clean & Build
```

**INPUT_ECU (Hitex Shield Buddy)**:

```bash
cd INPUT_ECU
# Aurix Development Studio에서 열기
# tc275_input_ecu TriCore Debug 프로젝트 선택
# Project > Clean & Build
```

**CONTROL_ECU (TC375 Lite Kit)**:

```bash
cd CONTROL_ECU
# Aurix Development Studio에서 열기
# TC375 프로젝트 선택
# Project > Clean & Build
```

#### 2️⃣ 웹 시스템 설치 (라즈베리파이 HMI_WEB)

**Windows/Mac/Linux 일괄 실행**:

```bash
cd HMI_WEB
./start-all.bat          # Windows
# 또는
./start-all.sh           # Mac/Linux
```

**수동 설치** (각 터미널에서 별도 실행):

1️⃣ **미들웨어 및 백엔드** (라즈베리파이):

```bash
cd HMI_WEB/backend
pip install -r requirements.txt  # can_receiver.py 의존성
npm install
npm run dev
# 서버 실행: http://라즈베리파이IP:3001
```

2️⃣ **프론트엔드** (새 터미널):

```bash
cd HMI_WEB/frontend
npm install
npm run dev
# 접속: http://라즈베리파이IP:5173
```

---

## 📊 데이터 흐름

```
센서 신호 (속도, 조향각, 버튼 등)
        ↓
INPUT_ECU (TC275)  ─→ [센서 데이터 수집 및 변환]
        ↓ CAN
CONTROL_ECU (TC375) ─→ [데이터 분석 및 위험도 판정]
        ├─ CAN ─→ ACT_ECU (TC275)    ─→ [LED, 버저, 서보 제어]
        │
        └─ CAN ─→ 라즈베리파이 (HMI_WEB)
                   ├─ can_receiver.py (수신)
                   ├─ Node.js 백엔드 (처리)
                   └─ React 프론트엔드 (시각화) → [웹 대시보드]
```

---

## 🔗 문서

각 모듈별 상세 문서:

- [ACT_ECU 상세 문서](./ACT_ECU/ACT_ECU/README.md)
- [INPUT_ECU 상세 문서](./INPUT_ECU/INPUT_ECU/README.md)
- [HMI_WEB 상세 문서](./HMI_WEB/README.md)

---

## 🔗 문서

각 모듈별 상세 문서:

- [ACT_ECU 상세 문서](./ACT_ECU/ACT_ECU/README.md)
- [INPUT_ECU 상세 문서](./INPUT_ECU/INPUT_ECU/README.md)
- [HMI_WEB 상세 문서](./HMI_WEB/README.md)

---

## 📝 라이선스

- **Infineon iLLD**: Boost Software License v1.0 (Infineon 제공)
- **프로젝트 코드**: Team Starex

---

## 💡 주요 특징

✅ **중앙 제어 아키텍처**: TC375 기반 강력한 데이터 처리  
✅ **실시간 모니터링**: WebSocket 기반 무지연 데이터 전송  
✅ **분산 제어 시스템**: INPUT → 중앙분석 → OUTPUT의 명확한 구조  
✅ **멀티코어 활용**: TC275, TC375의 다중 CPU 코어 활용  
✅ **모듈식 설계**: 새로운 센서/액추에이터 추가 용이  
✅ **웹 기반 대시보드**: 라즈베리파이에서 접근 가능한 실시간 UI

---

## 📞 연락처

프로젝트 관련 질문이나 제안사항이 있으신가요?  
각 팀 멤버의 GitHub 프로필을 방문해 주세요.

**프로젝트 관리자**: [황선안 (@sernan96)](https://github.com/sernan96)

---

**최종 업데이트**: 2026년 4월 7일
