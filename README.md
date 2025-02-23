# Home-IoT-System
STM32 보드를 이용한 Home IoT System


# 1. 프로젝트 소개

- **프로젝트 명**  
  - STM32 보드를 이용한 Home IoT System <br/><br/>

- **시연 url**
  - 시연 url 업로드 <br/><br/>

- **프로젝트 동기**
  - 스마트 홈 IoT 시스템의 설계와 구현 원리를 이해
  - STM32 보드와 센서를 활용한 자동화 시스템의 제어를 구현하기 위해 실시  <br/><br/>

- **프로젝트 목표**
  - STM32 환경에서 GPIO, ADC, PWM, DMA의 원리를 학습 
  - 공식 문서를 참조하여 핀(Pin) 설정
  - 초음파 센서, 조도 센서, 모터, LED를 통합 제어
  - 사용자 모드와 자동화 모드의 구분 및 전환 구현 <br/><br/>

- **프로젝트 설치 및 실행 방법**
  - 아래 링크에서 IAR workbench 설치
    - https://www.iar.com/kr/products/architectures/arm/iar-embedded-workbench-for-arm/iar-embedded-workbench-for-arm---free-trial-version/
  - 계정 등록 후 라이센스를 발급 받고 IAR License manager에서 라이센스 등록
  - 'Home-IoT-System' 폴더 업로드
  - STM32 보드에 main.c 코드를 실행해 포팅 <br/><br/>

# 2. 프로젝트 스펙
- **제작 기간** : 2024.11.15 - 2024.12.20<br/><br/>
- **팀원 및 역할**<br/><br/>
  - 편경찬
    - 초음파센서
    - 버튼 인터럽트
    - 조도센서
    - LED 센서
    - 블루투스 통신<br/><br/>
  - 조영진
    - 모터 센서<br/><br/>
  - 최민기
    - 블루투스 통신<br/><br/>
- **개발 환경**
  - 사용 보드 : STM32F107VC
  - 사용 언어 : C<br/><br/>
- **리포지토리 구조**
  ```
  ├── Home-IoT-System
  │   ├── CoreSupport
  │   ├── LCD_library
  │   ├── Libraries
  │   │   ├── CMSIS
  │   │   │   └── DeviceSupport
  │   │   │       └── Startup
  │   │   └── STM32F10x_StdPeriph_Driver_v3.5
  │   │       ├── inc
  │   │       └── src
  │   └── user
  |       ├── main.c
  │       └── inc
  └── 보고서&레퍼런스
  ```
  <br/>

# 3. 프로젝트 시나리오
  - 사용자가 집에 들어왔을 때 초음파 센서로 사람을 인식하고 블루투스로 알림을 띄워주는 기능
  - 집 외부에 하나의 조도센서를 배치해 일조량을 읽고 집 내부에 또 다른 조도센서를 배치해 집 내부의 밝기를 읽어 들인 후
  - 이 둘을 적절한 연산과 맵핑을 통해 최적의 밝기를 내는 센서등 및 무드등의 기능
  - 햇빛의 강도에 따라 모터 즉 선풍기를 가동하는 기능
  - 마지막으로 UART 블루투스 통신과 버튼 인터럽트를 통해 사용자가 직접 센서들을 제어할 수 있는 기능으로 구성<br/><br/>

# 4. 프로젝트 배경지식 및 주요 설계
  - **핀 맵핑**<br/><br/>
  ![Image](https://github.com/user-attachments/assets/b149e924-8b01-4118-8739-fcd6f31650eb)

  - **NVIC 셋팅**<br/><br/>
  ![Image](https://github.com/user-attachments/assets/22bd66bb-cd1f-45e0-b947-00b7b3600af0)

  - **초음파 센서**
    - 초음파 센서는 사람이 집에 들어왔는지 감지하여 시스템을 활성화하는 역할을 한다.
    - 초음파 신호를 사용하여 거리 데이터를 계산하고 이를 기반으로 시스템의 활성화 유무를 결정한다<br/><br/>
  - **LCD 센서**
    - 사용자가 입장해 초음파 센서로 인식되면 LCD가 켜진다.
    - LCD에는 두 개의 조도 센서로 읽은 조도값과 그 둘을 배합한 mood_value 값이 실시간으로 표시된다.<br/><br/>
  - **조도 센서**
    - 2개의 CDS 조도 센서를 활용하여, 하나는 외부의 광량을. 다른 하나는 실내의 밝기를 측정한다.
    - DMA를 ADC의 두 채널(채널 8과 채널 9)에 각각 조도 센서 1과 2를 매핑해 아날로그 값을 읽도록 구성했다<br/><br/>
  - **LED 센서**
    - 스마트 홈 IoT 시스템의 자동 밝기 무드등 기능을 담당한다.
    - 두 조도값을 적절하게 계산하여 스마트 홈 내부에 최적화된 밝기를 제공한다.<br/><br/>
  - **모터 센서**
    - 모터는 선풍기의 역할로써 스마트 홈 IoT 시스템 내 풍량을 조정하여 사용자에게 쾌적한 환경을 제공해주는 역할을 한다.
    - cds 조도센서의 센서값을 이용하여 구한 mood_value값을 바탕으로 바람의 세기를 조절한다. <br/><br/>
  - **버튼 인터럽트**
    - 버튼 1, 3, 4는 각각 인터럽트 핸들러 4, 15_10, 0에 등록된다.
    - 버튼 1은 모터를 on/off, 버튼 3은 LED 무드등을 on/off 해 사용자 모드로 진입하게 한다.
    - 버튼 4를 누르면 다시 시스템 제어 모드로 전환된다.<br/><br/>
  - **블루투스 통신**
    - USART1은 Putty와 연결되며 USART2는 Bluetooth와 연결하여 두 USART 간의 양방향 데이터 통신을 가능하게 된다
    - 사용자가 블루투스 터미널로 'a'를 입력하면 두 개의 모터가 동작하고, 'b'를 입력하면 한 개의 모터가 동작하며, 'c'를 입력하면 모든 모터가 꺼지도록 구성했다.
    - 이 동작들은 모두 사용자 모드로 진입하는 인터럽트에 해당하고, 반대로 'r'을 입력하면 사용자 모드를 해제하고 시스템 모드로 돌아가도록 구현했다.
    - 마지막으로 'q'를 입력하면 사용자가 퇴실했다는 신호로 간주하며, 모든 센서를 초기화하고 초음파 센서만 동작하도록 설정했다. <br/><br/>
# 5. 프로젝트 결과

![Image](https://github.com/user-attachments/assets/14de36ef-bd48-4d0f-9074-8cfe059a8cf2)

![Image](https://github.com/user-attachments/assets/429e41cf-5fc0-4059-9639-54218775a547)

![Image](https://github.com/user-attachments/assets/ae0d77df-0d07-4fcf-babd-697173c84c47)

# 6. 프로젝트 결론 및 리뷰
1. 센서와 장치 간의 통합 제어
    - 초음파 센서를 이용해 사용자 출입을 정확히 감지하고, 이를 기반으로 시스템 초기화를 수행하였다.
    - 조도 센서를 통해 실내외 밝기를 측정하고, LED 밝기와 모터 동작을 환경에 따라 동적으로 제어하였다.<br/><br/>
2. 효율적인 데이터 처리
    - DMA(Direct Memory Access)를 사용해 ADC 데이터를 CPU 개입 없이 처리함으로써 성능을 최적화하였다.
    - TIM 및 PWM(Pulse Width Modulation) 신호를 활용해 LED 밝기를 효율적으로 조절하였다.<br/><br/>
3. 사용자 친화적인 인터페이스
    - UART 블루투스 통신 및 버튼 인터럽트를 사용해 사용자 모드에서 장치를 직접 제어할 수 있도록 구현하였다.
    - ‘a’, ‘b’, ‘c’ 등의 간단한 명령어를 통해 사용자 모드에서 모터를 제어하고, ‘r’ 키를 통해 시스템 제어 모드로 전환 가능하게 하여 사용 편의성을 높였음.
    - ‘q’ 키를 통해 스마트 홈 IoT 시스템을 중단할 수 있고, 이후 초음파 센서가 동작해 사용자가 다시 집에 들어오는지를 실시간으로 확인할 수 있었다.<br/><br/>
4. 확장 가능성 확보
    - 설계 단계에서 Wi-Fi 및 음성 인식 기능을 추가할 수 있도록 시스템의 확장 가능성을 열어두었다.
    - 추가적인 센서와 장치를 통합할 수 있는 구조로 설계하였으며, 향후 다양한 인터페이스를 연결할 기반을 마련하였다.
    - 예를 들면 DMA의 채널을 늘려서 추가적인 센서들을 결합할 수 있다.<br/><br/>

# 7. 직면했던 문제점

1. 초음파 센서 간섭 문제
    - 초음파 센서와 모터의 동작이 간섭하여 간헐적으로 거리 측정이 부정확한 결과를 보였었다.
    - 이를 해결하기 위해 타이머 설정과 간격 조정을 통해 간섭을 최소화하는 방법을 적용하였다.<br/><br/>
2. UART 데이터 처리 지연 
    - 블루투스를 통한 명령 처리 중 일부 데이터 전송이 지연되는 문제가 발생하였다.
    - USART 인터럽트 우선순위를 조정하고 데이터를 실시간으로 처리하도록 개선하였다.<br/><br/>
3. 조도 센서 데이터 정확도 
    - 조도 센서의 값이 특정 환경에서 불규칙적으로 변화하는 현상이 발생하였다.
    - ADC의 샘플링 속도와 해상도를 조정하고 필터링 알고리즘을 추가하여 문제를 완화하였다.<br/><br/>
4. 인터럽트 방식의 비효율성
    - 조도 센서를 읽는 인터럽트 방식은 센서 핀마다 별도의 설정이 필요하고, 센서 간 번갈아 발생하는 인터럽트로 인해 지연시간 증가와 코드 복잡성을 초래했다.
    - DMA의 다중 채널 방식을 채택해 CPU 개입 없이 주변 장치와 메모리 간 데이터를 전송하도록 구성해 센서 안정성과 신뢰성을 보장했다. <br/><br/>

# 7. 참고자료
