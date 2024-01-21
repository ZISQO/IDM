# IDM Scanner 설치및 기본 사용 방법 가이드    
<br/>
<br/>
#### 주의 : 이 스크립트의 사용은 전적으로 사용자의 책임이 있음을 동의함을 전제로 파손 시 책임은 사용자 본인에게 있음을 안내합니다 가이드에 기재된 작업 과정은 반드시 실행하고 언급되지 않은 것은 패스하셔도 됩니다
<br/>
<br/>

## 1. IDM 장착
IDM 스캐너를 3D 프린터 툴헤드에 장착합니다(일반적으로 Z의 노즐에서 2.6mm 오목하게 들어간 위치)
정확성을 보장하기 위해 센서 코일 플레이트의 상단 표면이 핫엔드 히팅 블럭의 하단 보다 낮게 노즐 끝보다 2mm 위로 위치하도록 설치해야 합니다
보론2 / 트라이던트의 경우 CNC X 캐리지를 구매했을 경우 보다 쉬운 설치가 가능합니다
<br/>

[보론2/트라이던트용 마운트](/STL)(IDM 센서를 굳이 뒤로 더 후퇴시키지 않아도 문제가 되지 않습니다)
  
## 2. IDM 클리퍼 모듈 설치 
ssh 상에서 clone 명령을 사용해 설치 스크립트를 실행합니다
```
cd ~
git clone https://github.com/ZISQO/IDM_LSP.git
chmod +x ./IDM_LSP/install.sh
./IDM/install.sh
chmod -x ./IDM_LSP/install.sh
```

## 2-1. 문레이커 업데이트 수정
NBTP의 자동 업데이트를 사용하려면 아래의 내용을 문레이커 환경 설정 파일에 추가해 주세요
```
[update_manager idm]
type: git_repo
channel: dev
path: ~/IDM
origin: https://gitee.com/NBTP/IDM.git
env: ~/klippy-env/bin/python
requirements: requirements.txt
install_script: install.sh
is_system_service: False
managed_services: klipper
info_tags:
  desc=IDM Surface Scanner
```

뉴타입의 IDM 업데이트를 사용하려면 아래의 내용을 문레이커 환경 설정 파일에 추가해 주세요
```
[update_manager idm]
type: git_repo
channel: dev
path: ~/IDM
origin: https://github.com/ZISQO/IDM.git
env: ~/klippy-env/bin/python
requirements: requirements.txt
install_script: install.sh
is_system_service: False
managed_services: klipper
info_tags:
  desc=IDM Surface Scanner
```

## 3. IDM 모듈의 클리퍼 설정
printer.cfg에 아래 섹션을 추가합니다
```
[idm]
#serial:
#   USB 모듈의 경우에만 활성화 시키며 ls /dev/serial/* 명령으로 검색되는 IDM 장치 경로를 추가해 주세요 /dev/serial/by-id/usb-idm_idm_...
#   by-id 로 장치 인식이 모호할 경우 by-path에서 검색된 SBC 장치의 USB 포트를 기재하면 됩니다
canbus_uuid:
speed: 5.
#   Z축 프로빙 하강 속도
lift_speed: 5.
#   Z축 프로빙 상승 속도
backlash_comp: 0.5
#   센서 반응 측정전 Z축 이동 시 발생하는 백래시 제거를 위한 보정 거리
x_offset: 0.
#   노즐 X축 기준 IDM 센서의 중앙 위치 옵셋
y_offset: 21.1
#   노즐 Y축 기준 IDM 센서의 중앙 위치 옵셋
trigger_distance: 2.
#   호밍 시 IDM 센서의 트리거 거리
trigger_dive_threshold: 1.5
#   범위 대 다이빙 모드 프로빙의 임계값이며. trigger_distance + trigger_dive_threshold`값을 을 초과하면 다이브가 사용됨
trigger_hysteresis: 0.006
#   트리거 해제에 대한 트리거 임계값의 히스테리시스를 트리거 임계값의 백분율로 표시
cal_nozzle_z: 0.1
#   수동 Z 오프셋 보정을 완료한 후 예상되는 노즐 오프셋 
cal_floor: 0.1
#   센서 반응 측정에 대한 최소 z 바운드
cal_ceil:5.
#   센서 반응 측정의 최대 z 바운드  
cal_speed: 1.0
#   응답 곡선을 측정하는 속도
cal_move_speed: 10.
#   응답 곡선 측정을 위해 위치로 이동하는 동안의 속도
default_model_name: default
#   기본으로 로드할 IDM 모델의 이름
mesh_main_direction: x
#   메시 측정의 이동 기준 방향
#mesh_overscan: -1
#   메쉬 선 끝에서 방향 변경에 사용할 거리이며 이 설정을 생략하면 선 간격과 사용 가능한 이동 거리에서 기본 값이 계산됨
mesh_cluster_size: 1
#   메시 그리드 포인트 클러스터의 반경
mesh_runs: 2
#   메시 스캔 중 수행할 패스 횟수
```

 보론2/트라이던트가 아닌 경우 x,y 오프셋 조정을 유의해야 하며 센서 코일의 위치는 곧 측정하고자 하는 노즐의 위치와 같아야 함을 유의하시면 됩니다 이를테면 300 빌드 사이즈의 노즐 중앙은 X150, Y150이지만 위 옵셋 기준으론 X150 Y128.9 지점이 센서가 베드 중앙을 측정하게 됩니다

printer.cfg 파일 속성 중 ```[bed_mesh]``` 섹션과 옵션을 필요한 정보로 추가/수정 하지 않으면 클리퍼 시작시 오류 표시로 인해 정상 작동하지 않습니다
printer.cfg 파일 속성 중 ```[probe]``` 섹션은 삭제/주석 처리 해야 하며 아래와 같이 Z 엔드스탑 설정을 변경하도록 합니다
```
[stepper_z]
endstop_pin: probe:z_virtual_endstop # use idm as virtual endstop
homing_retract_dist: 0 # idm은 0으로 설정하고 beacon은 3으로 설정합니다
```

이미 ```[safe_z_home]``` 또는 ```[homing_override]``` 섹션을 사용중이라면 다음 단계를 건너 뛸 수 있습니다
(homing_override 기능은 g28 명령 시, X/Y/Z 모두 재-호밍 하므로 가급적 safe_z_home 섹션을 활성화 시켜 사용하실 것을 권장합니다


```
[safe_z_home]
home_xy_position: <x축 중앙>, <Y축 중앙-21.1>
z_hop: 10
```

### 3.1 시리얼 장치 찾기
시리얼 장치를 찾으려면 다음과 같이 실행해 주세요
```
ls /dev/serial/by-id/*
```

### 3.2 캔버스 장치 찾기
```Katapult``` 깃허브 자료를 다음과 같이 복제합니다
```
cd ~
git clone https://github.com/Arksine/katapult.git
```

캔버스 네트워크를 추가하지 않았다면 can0 인터페이스를 추가합니다다
```
sudo nano /etc/network/interfaces.d/can0
```

아래 내용을 나노 편집기에 붙여 넣은 다음 Ctrl + X를 누른 뒤, 엔터를 눌러 저장합니다
```
allow-hotplug can0
iface can0 can static
 bitrate 1000000
 up ifconfig $IFACE txqueuelen 4096
```

```can0``` 인터페이스를 활성화 시키려면 다음과 같이 실행해 주세요
```
sudo ifup can0
```

캔버스 장치를 찾으려면 다음과 같이 실행하세요
```
python3 ~/klipper/lib/canboot/flash_can.py -q
```

## 4. IDM 교정
X, Y축을 호밍합니다
```
G28 X Y
```

노즐을 베드 시트 중앙으로 이동 시킨 다음, 아래 명령을 실행합니다
```
G90
G0 X<X축 중앙>, Y<Y축 중앙-21.1>
```

아래 명령을 실행해 교정을 시작합니다
```
IDM_CALIBRATE
```

노즐과 베드 시트 간격 조절이 끝났다면 Probe_calibration 창 우측 하단의 accept를 눌러 줍니다
```
ACCEPT
```

콘솔창에 save_config 명령을 실행해 calibration offset 값을 저장한 뒤 클리퍼를 재시작 하도록 합니다
```
SAVE_CONFIG
```

## 5. 테스트
IDM 장치 상태 확인
```
IDM_QUERY
```

Z축만 호밍
```
G28 Z
```

IDM 스캐너 정확도 측정
```
PROBE_ACCURACY samples=100
```

Z축 백래시를 측정
```
IDM_ESTIMATE_BACKLASH
```

## 6. 베드 메시 교정
베드 메시 명령을 실행합니다
```
BED_MESH_CALIBRATE
```

## 7. 출력
첫 번째 프린트에서는 툴헤드 탭의 z-offset 값을 수정해 첫 번째 레이어 옵셋 값을 조정하시길 권장하며

인쇄가 완료된 후, 이 값을 저장하기 위해 ```Z_OFFSET_APPLY_PROBE`` 명령을 사용하면 됩니다

## 8. 고속 모드 진입
```z_tilt``` 또는 ```quad_gantry_level``` 명령 실행 중 현재 ```horizontal_z_move``` 값이 만약 2mm로 측정되고 printer.cfg의 idm 섹션에 설정한 ```trigger_distance``` + ```trigger_dive_threshold``` 총 합이 4였다면 레벨링 교정 과정을 고속으로 진입하게 됩니다(다이브 모드) 물론 고속 모드를 사용하지 않으려면 ```trigger_distance```를 1로 설정하고 ```trigger_dive_threshold```를 0.5로 설정하면 1.5mm를 초과하는 Z 높이에선 고속모드로 ```Z-TILT```및 ```quad_gangry_level``` 교정을 고속 모드로 진행하지 않게 됩니다

## 9. 가속도계 활성화 Enable The Accelerometer
lisdw 가속도계가 추가된 모델의 경우 아래 설정을 printer.cfg에 추가하도록 합니다
```
[lis2dw]
cs_pin: idm:PA3
spi_bus: spi1

[resonance_tester]
accel_chip: lis2dw
probe_points:
    <X축 중앙> , <Y축 중앙>, 10
```

## 10. 명령 모음 세트 확인
콘솔 창에서 ```IDM``` 명령을 입력한 뒤 Tab 키를 누르면 확인할 수 있습니다

## 11. IDM 업데이트
IDM의 PH2.0 단자를 전원 연결한 후 빠르게 전원 코드를 뽑았다가 다시 연결하면 LED가 천천히 깜박이기 시작하여 캔부팅에 진입했음을 나타냅니다. 그렇지 않다면 이 단계를 다시 반복하세요.

### 11.1 Canboot 업데이트
IDM 캔버스 UUID 확인:
```
python3 ~/klipper/lib/canboot/flash_can.py -q
```

CanBoot 펌웨어 업로드
```
python3 ~/klipper/lib/canboot/flash_can -f ~/IDM/Canboot/Canboot_1M.bin -u <found uuid>
```
### 11.2 클리퍼 애플리케이션 업로드
IDM 캔버스 UUID:
```
python3 ~/klipper/lib/canboot/flash_can.py -q
```

Klipper 애플리케이션 펌웨어 업로드
```
python3 ~/klipper/lib/canboot/flash_can -f ~/IDM/Firmware/IDM_CAN_8kib_offset_1M.bin -u <found uuid>
```

## 12. USB 설치
USB로 변경하려면 USB 펌웨어로 설정하되 IDM PCB 뒷면의 USB 패딩 2개를 서로 맞 붙도록 쇼트 처리하면 됩니다.
```
python3 ~/klipper/lib/canboot/flash_can.py -f ~/IDM/Firmware/IDM_USB_8kib_offset.bin -u <found uuid>
```
